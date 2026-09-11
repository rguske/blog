---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "KubeVirt meets Eventing: Automating VM Lifecycle Data with Knative and FaaS"
description: "A hands-on, end-to-end walkthrough of tracking KubeVirt virtual machine lifecycle events (create/delete) with Knative Eventing's ApiServerSource, Broker and EventTransform, persisting the trimmed CloudEvents into PostgreSQL via a Python Knative Function, and reading them back out through a small web frontend."
date: 2026-09-11T09:00:00+02:00
draft: true
featuredImage: /img/kubevirt_meets_eventing_cover.png
categories: ["Platforms", "Serverless", "Virtualization"]
tags:
- RedHat
- OpenShift
- Knative
- KubeVirt
- Serverless
- FaaS
- CloudEvents
- PostgreSQL
- EventDriven
- Automation
---

## From Concept to Reality

{{< admonition note "Where this started" true >}}
This post builds directly on [Monitoring Virtual Machines with Knative Eventing](https://knative.dev/blog/articles/kubevirt_meets_eventing/), an article I co-authored with my colleague Matthias Weßendorf on the official Knative blog. That post lays out the "what" and "why" nicely and concisely. Consider this one the deeper, hands-on, OpenShift-flavored companion: full, reproducible manifests you can copy-paste straight into your own cluster, plus a new addition that wasn't part of the original article at all - a small web frontend to actually look at the data we're collecting.
{{< /admonition >}}

The underlying problem is one I keep running into with customers, and it's a boring but very real one: teams run virtual machines on KubeVirt/OpenShift Virtualization right next to their containerized workloads, and at some point somebody asks the question "which VMs do we actually have running, and with what specs?" Too often the honest answer is a spreadsheet that's been out of sync since the last person who maintained it left the team.

What you actually want is a small, always-up-to-date record, think a CMDB-like PostgreSQL table, that reflects reality without anyone having to remember to update it, and without a script polling the API every few minutes just to notice a VM appeared or disappeared. VM lifecycle operations (create, delete) already produce events on the Kubernetes API server. The trick is doing something useful with them the moment they happen, instead of ignoring them.

That's exactly the gap Functions-as-a-Service (FaaS) and Knative Eventing fill. Knative Eventing is the backbone that gets Kubernetes-native events flowing reliably from A to B, and FaaS gives us a small, single-purpose piece of business logic that only runs when there's actually an event worth acting on. No long-running polling service, no cron job, just a function that wakes up, does its one job, and goes back to sleep.

## Architecture at a Glance

{{< image src="/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png" caption="Figure I: End-to-end event flow from VM lifecycle event to database" src-s="/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png" >}}

Let's walk through the diagram hop by hop, since every box in there is a piece we'll actually deploy later in this post:

1. **Kubernetes API Server** - the event producer. The moment a `VirtualMachine` gets created or deleted, the API server is where that fact first exists as an event.
2. **`ApiServerSource`** - a Knative Eventing source watching the API server for exactly that kind of resource. It picks up the create/delete operation and forwards it as a CloudEvent.
3. **`Broker`** - the central routing point. The `ApiServerSource` sends its event here first.
4. **`EventTransform`** - the raw event coming out of the API server is huge and mostly irrelevant to us (full VM spec, status, metadata, you name it). `EventTransform` trims it down to just the fields we care about (name, namespace, CPU, memory, storage class, etc.) and hands the slimmed-down event back to the `Broker`.
5. **Two `Trigger`s** - one filtered on `dev.knative.apiserver.resource.add`, the other on `dev.knative.apiserver.resource.delete`. Each `Trigger` watches the `Broker` for its specific event type and, when it matches, invokes the same downstream subscriber.
6. **The `kn-py-vmdata-psql-fn` Function** - a Python Knative Function that receives the transformed event and writes (or removes) the corresponding row.
7. **PostgreSQL** - the actual "CMDB" table, always reflecting the current state of VMs in the cluster.
8. **The web frontend** (not shown in the diagram above) - a small, read-only UI on top of that PostgreSQL table, so you don't have to reach for `psql` every time you want to know what's running. This piece is my own addition on top of the original article's scope, and we'll build it later in this post.

## What Is the Event Transformer?

Step 4 in the list above deserves its own explanation before we start deploying anything. The `ApiServerSource` doesn't just tell you "a VM named `rhel-vm-2` was created", it forwards the *entire* Kubernetes API object for that `VirtualMachine`, wrapped in a CloudEvent. That's the full spec, the full status, all the metadata Kubernetes tracks internally, easily a few hundred lines of JSON for something as simple as a VM create event. Great for completeness, not so great when all a small Python function actually needs is a name, a namespace, and a handful of spec fields.

Here's a trimmed, illustrative excerpt of what that raw `dev.knative.apiserver.resource.add` event looks like (real payloads are considerably longer, this is not the full object):

```json
{
  "specversion": "1.0",
  "type": "dev.knative.apiserver.resource.add",
  "source": "https://172.30.0.1:443",
  "subject": "/apis/kubevirt.io/v1/namespaces/kubevirt-eventing/virtualmachines/rhel-vm-2",
  "id": "5508cafb-3332-4709-a1b1-a8657111d82c",
  "time": "2025-07-07T13:02:18.124Z",
  "data": {
    "spec": {
      "template": {
        "spec": {
          "domain": {
            "cpu": { "cores": 4, "sockets": 2 },
            "memory": { "guest": "8Gi" }
          },
          "networks": [{ "name": "default" }]
        }
      },
      "dataVolumeTemplates": [
        {
          "spec": {
            "storage": {
              "resources": { "requests": { "storage": "30Gi" } },
              "storageClassName": "coe-netapp-san"
            }
          }
        }
      ]
    }
  }
}
```

That's exactly the "trims the fat" problem the `EventTransform` API solves. Introduced in Knative Eventing v1.18, `EventTransform` is a CRD that uses [JSONata](https://jsonata.org/) expressions to reshape a CloudEvent's payload in-flight, picking out only the attributes you care about and dropping everything else. It's a standalone building block too, not tied to any single source or sink, so it can sit anywhere in your event flow: right after the `Broker`, in front of a `Trigger`, wherever trimming makes sense for that hop.

We won't write the JSONata expression itself just yet, that's coming up next, where we configure `EventTransform` to emit exactly the columns our `vmdb` PostgreSQL table expects: name, namespace, CPU cores/sockets, memory, storage size, storage class, and network.

## Deploying the Event Pipeline

Theory's out of the way, time to actually roll this out on an OpenShift cluster. Everything below is applied in order, since later objects reference the names created earlier.

### Setting the Stage: Brokers & RBAC

We deploy two `Broker`s rather than one: `broker-apiserversource` receives the raw, untrimmed events straight from the `ApiServerSource`, while `broker-eventtransform` only ever sees the already-trimmed events coming out of `EventTransform`. Keeping them separate means a `Trigger` subscribing to either one always gets events of a predictable shape, instead of having to filter raw and transformed events apart downstream.

```yaml
oc create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: broker-apiserversource
spec: {}
---
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: broker-eventtransform
spec: {}
EOF
```

Neither `Broker` has a `spec` beyond its name, that's the memory-backed, no-frills default, perfectly fine to get started with.

The `ApiServerSource` we're about to create doesn't get to watch cluster resources for free. By default there's no ServiceAccount with permission to `get`/`list`/`watch` `VirtualMachine`/`VirtualMachineInstance` objects, so we need a dedicated `ServiceAccount` plus a `Role`/`RoleBinding` granting exactly that:

```yaml
oc create -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: events-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: event-watcher
rules:
  - apiGroups:
      - "kubevirt.io"
    resources:
      - virtualmachines
      - virtualmachineinstances
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: role-event-watcher
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: event-watcher
subjects:
  - kind: ServiceAccount
    name: events-sa
EOF
```

### Wiring Up the ApiServerSource

With the plumbing in place, we can finally create the `ApiServerSource` itself. The important bit here is `mode: Resource`: instead of forwarding the generic, small Kubernetes `Event` objects (the kind you see with `oc get events`), `mode: Resource` makes the source watch the *actual resource* listed under `resources` and emit a CloudEvent carrying that resource's full current state every time it changes. We scope it to exactly one `apiVersion`/`kind` pair, `kubevirt.io/v1` `VirtualMachine`, so we only hear about VM lifecycle changes and nothing else running in the cluster. The source authenticates as the `events-sa` ServiceAccount we just created, and sinks its events straight into `broker-apiserversource`:

```yaml
oc create -f - <<EOF
apiVersion: sources.knative.dev/v1
kind: ApiServerSource
metadata:
  name: apiserversource
  labels:
    app: apiserversource
spec:
  mode: Resource
  resources:
    - apiVersion: kubevirt.io/v1
      kind: VirtualMachine
  serviceAccountName: events-sa
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: broker-apiserversource
EOF
```

From this point on, every `VirtualMachine` create or delete in the cluster shows up as a `dev.knative.apiserver.resource.add` / `dev.knative.apiserver.resource.delete` CloudEvent inside `broker-apiserversource`, looking exactly like the verbose payload shown earlier.

### Transforming the Event

Now for the piece that actually does the trimming: an `EventTransform` named `vmdata-transform`, sinking its output into the second `Broker`, `broker-eventtransform`:

```yaml
oc create -f - <<EOF
apiVersion: eventing.knative.dev/v1alpha1
kind: EventTransform
metadata:
  name: vmdata-transform
spec:
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: broker-eventtransform
  jsonata:
    expression: |
      {
        "specversion": specversion,
        "type": type,
        "source": source,
        "subject": subject,
        "id": id,
        "time": time,
        "kind": kind,
        "name": name,
        "namespace": namespace,
        "cpucores": data.spec.template.spec.domain.cpu.cores,
        "cpusockets": data.spec.template.spec.domain.cpu.sockets,
        "memory": data.spec.template.spec.domain.memory.guest,
        "datasource": data.spec.dataVolumeTemplates.spec.storage.resources.resources.storage,
        "storageclass": data.spec.dataVolumeTemplates.spec.storage.storageClassName,
        "network": data.spec.template.spec.networks.name
      }
EOF
```

Mapping this back to the raw event excerpt from earlier: `cpucores` and `cpusockets` come straight from `data.spec.template.spec.domain.cpu.cores`/`.sockets` (`4` and `2` for `rhel-vm-2`), `memory` from `data.spec.template.spec.domain.memory.guest` (`8Gi`), `datasource` and `storageclass` from the VM's `dataVolumeTemplates` entry (`30Gi` on `coe-netapp-san`), and `network` from `data.spec.template.spec.networks[].name` (`default`). Everything else in that few-hundred-line payload, status, resource versions, UID, the works, simply isn't referenced in the expression, so it's dropped.

Once this is in place, the event `broker-eventtransform` receives is a fraction of the size of the original, something like this (illustrative, using the same `rhel-vm-2` example as before):

```code
Context Attributes,
  specversion: 1.0
  type: dev.knative.apiserver.resource.add
  source: https://172.30.0.1:443
  subject: /apis/kubevirt.io/v1/namespaces/kubevirt-eventing/virtualmachines/rhel-vm-2
  id: 7a291e4c-6f0d-4b8a-9c3e-2d4f8b6a19d7
  time: 2025-08-19T09:41:52.318Z
Extensions,
  cpucores: 4
  cpusockets: 2
  datasource: 30Gi
  kind: VirtualMachine
  memory: 8Gi
  name: rhel-vm-2
  namespace: kubevirt-eventing
  network: default
  storageclass: coe-netapp-san
```

That's exactly the shape our downstream function needs, no more, no less.

### Triggers: Routing Add/Delete Events

The last piece connecting `broker-apiserversource` to `vmdata-transform` is a pair of `Trigger`s. A `Trigger` binds a `Broker` to a subscriber via an event-type filter: "when an event matching this filter shows up on this `Broker`, deliver it to this subscriber." Here, `trigger-transformer-vm-add` matches `dev.knative.apiserver.resource.add` and `trigger-transformer-vm-delete` matches `dev.knative.apiserver.resource.delete`, both forwarding to the `vmdata-transform` `EventTransform` we just created, with a small retry policy in case the transform is momentarily unavailable:

```yaml
oc create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  labels:
    eventing.knative.dev/broker: broker-apiserversource
  name: trigger-transformer-vm-add
spec:
  broker: broker-apiserversource
  filter:
    attributes:
      type: dev.knative.apiserver.resource.add
  subscriber:
    ref:
      apiVersion: eventing.knative.dev/v1alpha1
      kind: EventTransform
      name: vmdata-transform
  delivery:
    retry: 1
    backoffPolicy: linear
    backoffDelay: PT5S
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  labels:
    eventing.knative.dev/broker: broker-apiserversource
  name: trigger-transformer-vm-delete
spec:
  broker: broker-apiserversource
  filter:
    attributes:
      type: dev.knative.apiserver.resource.delete
  subscriber:
    ref:
      apiVersion: eventing.knative.dev/v1alpha1
      kind: EventTransform
      name: vmdata-transform
  delivery:
    retry: 1
    backoffPolicy: linear
    backoffDelay: PT5S
EOF
```

With this in place, the full pipeline is live end to end: `VirtualMachine` create/delete → `ApiServerSource` → `broker-apiserversource` → `Trigger` → `EventTransform` → `broker-eventtransform`. The only thing missing now is something actually subscribing to `broker-eventtransform` and doing something useful with those trimmed events, which is exactly where the Knative Function comes in next.

## The PostgreSQL Backend

Everything up to this point has been about getting events into the right shape and to the right place. The other half of the "CMDB-like PostgreSQL table" idea from the introduction is the database itself: a StatefulSet-backed PostgreSQL 16 instance sitting behind a `ClusterIP` `Service`. To be upfront about it, what's running here is homelab/demo-grade: a single replica backed by `ReadWriteOnce` `PersistentVolumeClaim`s, not something you'd take to production as-is. That's fine for this post though, because the interesting part isn't the `StatefulSet`, it's the schema. Swap this out for any Postgres instance or Operator you already have reachable from your cluster; as long as it can run the `CREATE TABLE` statement coming up below, it'll work just as well.

The one credential involved is the database password, held in a small `Secret`:

```yaml
oc create -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: postgresql-secret
  labels:
    app: postgres
type: Opaque
data:
  POSTGRES_PASSWORD: 'cmVkaGF0Cg=='
EOF
```

I'm skipping the full `PersistentVolumeClaim`/`StatefulSet`/`Service` YAML here (roughly 140 lines): it's a completely standard Kubernetes Postgres deployment with nothing KubeVirt- or Knative-specific about it, so pasting all of it would just be noise, adapt your own Postgres instance or Operator of choice instead.

### Initializing the `vmdb` Database

With PostgreSQL reachable, a one-off Kubernetes `Job` creates the `vmdb` database and a `virtual_machines` table whose columns mirror the trimmed event fields coming out of the `EventTransform` we defined earlier, `type`, `id`, `kind`, `name`, `namespace`, `time`, `cpucores`, `cpusockets`, `memory`, `storageclass` and `network`, letter for letter, minus a couple of CloudEvent envelope attributes (`specversion`, `source`, `subject`) and the `datasource` size field that simply aren't persisted:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: init-vmdb
  namespace: postgres
spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: init-vmdb
        image: registry.redhat.io/rhel9/postgresql-15
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: pg-credentials
              key: DB_HOST
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: pg-credentials
              key: DB_USER
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: pg-credentials
              key: DB_PASSWORD
        command:
        - /bin/bash
        - -c
        - |
          set -e

          echo "Checking database vmdb..."

          psql -h ${DB_HOST} -U ${DB_USER} -d postgres -tAc \
            "SELECT 1 FROM pg_database WHERE datname='vmdb'" | grep -q 1 \
            || psql -h ${DB_HOST} -U ${DB_USER} -d postgres -c \
            "CREATE DATABASE vmdb;"

          echo "Creating table virtual_machines..."

          psql -h ${DB_HOST} -U ${DB_USER} -d vmdb <<EOF
          CREATE TABLE IF NOT EXISTS public.virtual_machines (
              type TEXT,
              id TEXT,
              kind TEXT,
              name TEXT,
              namespace TEXT,
              time TEXT,
              cpucores TEXT,
              cpusockets TEXT,
              memory TEXT,
              storageclass TEXT,
              network TEXT
          );
          EOF

          echo "Database initialization completed."
```
