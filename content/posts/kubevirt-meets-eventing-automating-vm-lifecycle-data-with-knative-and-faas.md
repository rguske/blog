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
