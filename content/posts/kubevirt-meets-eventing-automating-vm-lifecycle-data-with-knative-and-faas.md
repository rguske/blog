---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "KubeVirt meets Eventing: Automating VM Lifecycle Data with Knative and FaaS"
description: "A hands-on, end-to-end walkthrough of tracking KubeVirt virtual machine lifecycle events (create/delete) with Knative Eventing's ApiServerSource, Broker, EventTransform and Trigger, persisting the trimmed CloudEvents into PostgreSQL via a Python Knative Function and reading them back out through a small web frontend. Written as the deeper, hands-on, OpenShift-flavored companion to the original Knative blog article on the same topic, with full copy-pasteable manifests end-to-end."
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

## Prerequisites

Everything from here on assumes Knative Serving and Eventing (or, on OpenShift, the OpenShift Serverless Operator) plus KubeVirt/OpenShift Virtualization are already installed and healthy on the cluster; this section covers only the Knative/Serverless half of that equation, getting KubeVirt itself running is a separate exercise and out of scope here, [KubeVirt's own quickstart](https://kubevirt.io/quickstart_minikube/) is the place to start if you need it.

### On OpenShift: the OpenShift Serverless Operator

On OpenShift, the OpenShift Serverless Operator is the fastest path to a healthy cluster: it manages Knative Serving, Knative Eventing, and the Knative broker for Apache Kafka as a single product, so there's one Operator lifecycle to watch instead of three.

Start by subscribing the cluster to the Operator with a `Namespace`, `OperatorGroup`, and `Subscription`, saved as `serverless-subscription.yaml`:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-serverless
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: serverless-operators
  namespace: openshift-serverless
spec: {}
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: serverless-operator
  namespace: openshift-serverless
spec:
  channel: stable
  name: serverless-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

```shell
oc apply -f serverless-subscription.yaml
```

Give it a moment, then confirm the cluster service version has reached `Succeeded`:

```shell
oc get csv
```

```text
NAME                          DISPLAY                        VERSION   REPLACES                      PHASE
serverless-operator.v1.25.0   Red Hat OpenShift Serverless   1.25.0    serverless-operator.v1.24.0   Succeeded
```

With the Operator in place, install Knative Serving by applying a minimal `KnativeServing` CR (`serving.yaml`):

```yaml
apiVersion: operator.knative.dev/v1beta1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
```

```shell
oc apply -f serving.yaml
```

```shell
oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'
```

```text
DependenciesInstalled=True
DeploymentsAvailable=True
InstallSucceeded=True
Ready=True
```

Same pattern for Knative Eventing (`eventing.yaml`), this is the piece that actually matters for everything below, `Broker`, `Trigger`, `ApiServerSource`, and `EventTransform` all live here:

```yaml
apiVersion: operator.knative.dev/v1beta1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
```

```shell
oc apply -f eventing.yaml
```

```shell
oc get knativeeventing.operator.knative.dev/knative-eventing -n knative-eventing --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'
```

Once that reports `InstallSucceeded=True` and `Ready=True` alongside the same result for `knativeserving`, the cluster is ready for the `oc create -f -` manifests coming up next.

### Everywhere Else: Upstream Knative Serving and Eventing

Not on OpenShift? Install upstream Knative Serving and Eventing directly. For a supported, long-term install on any Kubernetes cluster, the [Knative Operator](https://knative.dev/docs/install/operator/knative-with-operators/) gives you the same CRD-driven approach used above. For quick local experimentation, the [Knative Quickstart](https://knative.dev/docs/install/quickstart-install/)'s `kn` plugin spins up a `kind`/`minikube` cluster with Serving and Eventing already wired together in a couple of commands, useful for kicking the tyres, not for production. Either way, no manifests to paste here, everything from "Deploying the Event Pipeline" onward is plain `oc`/`kubectl` resources and doesn't care which install path got you to a healthy `knative-serving`/`knative-eventing` pair.

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

## Deploying the `kn-py-vmdata-psql-fn` Function

With the pipeline delivering trimmed events to `broker-eventtransform` and `vmdb` standing by to receive them, the last piece is the function that actually connects the two. `kn-py-vmdata-psql-fn` is a small Python Knative Function: it receives the transformed CloudEvent and, depending on the event `type`, either inserts a new row into `virtual_machines` (`dev.knative.apiserver.resource.add`) or removes the matching one (`dev.knative.apiserver.resource.delete`). It also keeps track of the CloudEvent `id`s it has already handled, so if the same event ever gets redelivered, it recognizes the duplicate and skips it rather than writing (or deleting) the row twice.

<i class='fab fa-github fa-fw'></i> repository :point_right: [rguske/knative-functions/kn-py-vmdata-psql-fn](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn)

The function needs the same DB connection details as the `init-vmdb` `Job` from earlier, held in their own `Secret`:

```shell
oc create secret generic psql-secret \
  --from-literal=db_host="192.168.50.50" \
  --from-literal=db_port="5432" \
  --from-literal=db_name="vmdb" \
  --from-literal=db_user="postgres" \
  --from-literal=db_password="redhat"
```

With the secret in place, deploy the function itself as a Knative `Service`. Each `DB_*` environment variable is sourced straight from `psql-secret`, and just like the pipeline's other single-purpose components, `min`/`maxScale` are both pinned to `1`:

```yaml
oc create -f - <<EOF
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: kn-py-psql-vmdata-fn
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "1"
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
        - image: quay.io/rguske/kn-py-psql-vmdata-fn:v1.0
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: psql-secret
                  key: db_host
            - name: DB_PORT
              valueFrom:
                secretKeyRef:
                  name: psql-secret
                  key: db_port
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                  name: psql-secret
                  key: db_name
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: psql-secret
                  key: db_user
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: psql-secret
                  key: db_password
EOF
```

The last step is hooking the function up to `broker-eventtransform` with the same add/delete `Trigger` pattern used earlier for the transformer: `trigger-vm-add` matches `dev.knative.apiserver.resource.add`, `trigger-vm-delete` matches `dev.knative.apiserver.resource.delete`, and both point at the `kn-py-psql-vmdata-fn` `Service` we just created:

```yaml
oc create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  labels:
    eventing.knative.dev/broker: broker-eventtransform
  name: trigger-vm-add
spec:
  broker: broker-eventtransform
  filter:
    attributes:
      type: dev.knative.apiserver.resource.add
  subscriber:
    ref:
      apiVersion:  serving.knative.dev/v1
      kind: Service
      name: kn-py-psql-vmdata-fn
  delivery:
    retry: 1
    backoffPolicy: linear
    backoffDelay: PT5S
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  labels:
    eventing.knative.dev/broker: broker-eventtransform
  name: trigger-vm-delete
spec:
  broker: broker-eventtransform
  filter:
    attributes:
      type: dev.knative.apiserver.resource.delete
  subscriber:
    ref:
      apiVersion:  serving.knative.dev/v1
      kind: Service
      name: kn-py-psql-vmdata-fn
  delivery:
    retry: 1
    backoffPolicy: linear
    backoffDelay: PT5S
EOF
```

With that applied, the pipeline is complete end to end: `VirtualMachine` create/delete → `ApiServerSource` → transform → `broker-eventtransform` → `Trigger` → `kn-py-psql-vmdata-fn` → `virtual_machines`.

{{< admonition info "Skipping duplicate events" true >}}
Knative Eventing's delivery guarantee is at-least-once, not exactly-once, so the same CloudEvent can legitimately show up at the function's door more than once, a retry after a slow response, a redelivery after a brief network hiccup, and so on. Left unchecked, a repeated `add` event would simply run the same `INSERT` again. `kn-py-vmdata-psql-fn` guards against this by remembering the CloudEvent `id` it has already processed and short-circuiting on a repeat, logging a line to that effect and returning a "skipped" response instead of touching the database a second time. It's a small check, but it's what keeps `virtual_machines` an accurate mirror of cluster state instead of quietly drifting under retries.
{{< /admonition >}}

## Validating End-to-End

With every piece deployed, the only thing left is proof: create and delete a handful of VMs, then query `vmdb` directly to confirm `virtual_machines` actually tracked them.

```shell
for i in $(seq 1 5); do oc process -n openshift rhel9-server-medium -p NAME=vm${i} | oc apply -f - ; done;
```

I ran that loop to spin up five VMs, deleted them again, and then connected with `psql` to check whether both the adds and the deletes made it into the table:

```shell
psql -U postgres -h 10.32.98.110 -p 5432 -d vmdb -c 'SELECT * FROM "virtual_machines"'

Password for user postgres:
                 type                  |                  id                  |      kind      |    name    |    namespace    |           time           | cpucores | cpusockets | memory | storageclass | network
---------------------------------------+--------------------------------------+----------------+------------+-----------------+--------------------------+----------+------------+--------+--------------+---------
 dev.knative.apiserver.resource.add    | 6ee40bfe-7c9d-445c-943e-5ddb8f4fd47c | VirtualMachine | rguske-vm1 | rguske-eventing | 2025-04-10T08:59:25.407Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.add    | c769cae2-6743-4923-afe5-46b44af5a5f7 | VirtualMachine | rguske-vm2 | rguske-eventing | 2025-04-10T08:59:26.695Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.add    | 39c73b7d-4842-4e72-9329-5c8b8d14d12f | VirtualMachine | rguske-vm3 | rguske-eventing | 2025-04-10T08:59:27.995Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.add    | 66af44d6-6b97-4a51-a06c-29f7a775e78b | VirtualMachine | rguske-vm4 | rguske-eventing | 2025-04-10T08:59:29.203Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.add    | f2561dd5-2020-47b3-86f3-038b943c5d10 | VirtualMachine | rguske-vm5 | rguske-eventing | 2025-04-10T08:59:31.309Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.delete | 8915c52d-4e54-4b2c-a5a2-34ac3aea70d9 | VirtualMachine | rguske-vm1 | rguske-eventing | 2025-04-10T09:09:48.276Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.delete | dcbc4779-184e-43d7-af2e-afc3b3762e16 | VirtualMachine | rguske-vm2 | rguske-eventing | 2025-04-10T09:09:48.396Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.delete | febeab4e-bd9d-451e-b0df-b2dc4e2d83c6 | VirtualMachine | rguske-vm3 | rguske-eventing | 2025-04-10T09:09:48.515Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.delete | 3d166a04-29f1-4968-aaf2-ac5657b79570 | VirtualMachine | rguske-vm4 | rguske-eventing | 2025-04-10T09:09:48.624Z | 1        | 1          | 2Gi    |              | default
 dev.knative.apiserver.resource.delete | e873eadc-c54b-4df9-aa70-9b1dfb864128 | VirtualMachine | rguske-vm5 | rguske-eventing | 2025-04-10T09:09:48.755Z | 1        | 1          | 2Gi    |              | default
(10 rows)
```

Five `add` rows, five `delete` rows, each carrying the CloudEvent `id` that made it unique, exactly what the pipeline was built to produce.

### Reading the Data with pgAdmin (Optional)

`psql` gets the job done, but if you'd rather browse `virtual_machines` than type SQL, pgAdmin is a friendlier alternative:

```shell
podman run -p 80:80 \
    -e 'PGADMIN_DEFAULT_EMAIL=rguske@redhat.com' \
    -e 'PGADMIN_DEFAULT_PASSWORD=redhat' \
    -d dpage/pgadmin4:9.2.0
```

{{< image src="" caption="Figure II: virtual_machines table browsed in pgAdmin" src-s="" >}}

## Closing the Loop: A Web Frontend to Read the Data

Both of the options above still expect you to be comfortable with `psql` or willing to stand up pgAdmin just to peek at a table, and that's exactly the gap I flagged back in the intro as *my own addition* on top of the original Knative blog article: a small, read-only web frontend for `vmdb`. Nothing fancy, no write access, no auth beyond what sits in front of it, just a page that queries `virtual_machines` and renders it so anyone on the team can check what's running (or what used to be) without ever touching a terminal.

<i class='fab fa-github fa-fw'></i> repository :point_right: [rguske/postgresql-read-webapp](https://github.com/rguske/postgresql-read-webapp)

Deploying it follows the same pattern as the rest of this pipeline. First, the app needs the same DB credentials the init `Job` used earlier, packaged as a `secret`:

```shell
oc -n rguske-eventing create secret generic pg-credentials \
  --from-literal=DB_HOST=10.32.98.110 \
  --from-literal=DB_USER=postgres \
  --from-literal=DB_PASSWORD='redhat'
```

Then the app itself ships as a Knative Service, scaling to zero when nobody's looking and back up on the next request:

```shell
kn service create postgresql-read-webapp \
  --image=quay.io/rguske/psql-read-webapp:v1.1 \
  --env-from secret:pg-credentials \
  --env DB_NAME=vmdb \
  --env DB_PORT=5432 \
  --scale-min=0 \
  --scale-max=2
```

{{< image src="" caption="Figure III: postgresql-read-webapp displaying the virtual_machines table" src-s="" >}}

## Wrap-Up

Stepping back, the actual takeaway here has very little to do with VMs specifically. The interesting bit is the pattern: `ApiServerSource` watching a resource, a `Broker` routing what it hears, `EventTransform` trimming the noise, and `Trigger`s filtering by event type before handing off to a function. Swap `VirtualMachine` for `Deployment`, `Pod`, `Namespace`, or any other Kubernetes resource, built-in or a CRD of your own, and the exact same four building blocks apply. That's the real win of going event-driven instead of polling: you stop writing "check every N minutes and diff against last time" scripts, and you start reacting the moment something actually changes.

From here, a few natural next steps come to mind: hooking an alerting path onto VM deletion events so someone actually gets notified when a VM disappears, extending the same pipeline to other KubeVirt resource types (`VirtualMachineInstance`, `DataVolume`), or, for anyone taking this beyond a homelab, swapping the in-memory `Broker`s used throughout this post for a Kafka-backed `Broker` to get durable, replayable event storage in production.

## Resources

- [Monitoring Virtual Machines with Knative Eventing](https://knative.dev/blog/articles/kubevirt_meets_eventing/) - the original article this post builds on
- <i class='fab fa-github fa-fw'></i> [rguske/knative-functions/kn-py-vmdata-psql-fn](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn)
- <i class='fab fa-github fa-fw'></i> [rguske/postgresql-read-webapp](https://github.com/rguske/postgresql-read-webapp)
- [Knative Eventing docs](https://knative.dev/docs/eventing/)
- [KubeVirt](https://kubevirt.io/)
