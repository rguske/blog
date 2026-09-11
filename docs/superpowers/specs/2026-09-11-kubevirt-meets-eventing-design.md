# Design: "KubeVirt meets Eventing" Blog Post

## Summary

A new Hugo blog post documenting an event-driven automation solution: whenever a
KubeVirt `VirtualMachine` is created or deleted on OpenShift, a chain of Knative
Eventing primitives (ApiServerSource → Broker → EventTransform → Trigger) filters
and trims the raw Kubernetes API event down to just the relevant VM metadata, then
delivers it to a Python Knative Function (FaaS) that persists it into PostgreSQL.
A second Knative Service provides a simple web frontend to read that data back out.

The post is based on a real customer engagement (generalized, no names) and is
built from the user's own draft notes (`~/Documents/README.md`), plus the two
existing GitHub repos documenting the function and webapp.

## Sources of truth

- `~/Documents/README.md` — draft manifests/commands for the full deployment
  (Brokers, RBAC, ApiServerSource, EventTransform/JSONata, VirtualMachinePool,
  PostgreSQL StatefulSet, DB init Job, function deployment, validation via psql/pgAdmin)
- [`rguske/knative-functions/kn-py-vmdata-psql-fn`](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn) — the Knative Function source + README (build/test/deploy instructions, sample transformed CloudEvent payload, duplicate-event handling)
- [`rguske/postgresql-read-webapp`](https://github.com/rguske/postgresql-read-webapp) — the read-only web frontend source + README (build/deploy instructions incl. Kubernetes Deployment/Service and Knative Service variants)
- Style reference: existing posts in `content/posts/`, most notably the two other
  Knative/FaaS posts (`event-driven-automation-with-project-harbor-and-knative.md`,
  `event-driven-interactions-with-vsphere-using-functions-as-a-service.md`) and the
  most recent post (`bootable-containers-an-entire-operating-system-as-a-containerfile.md`)
  for current frontmatter/formatting conventions.
- Pre-existing (unused) asset folder `static/img/posts/202512_kubevirt_meets_eventing/`
  and diagram source `~/Library/CloudStorage/GoogleDrive-.../excalidraw/kubevirt-meets-eventing.png`
  indicate this post idea was already earmarked.

## Frontmatter

```yaml
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "KubeVirt meets Eventing: Automating VM Lifecycle Data with Knative and FaaS"
description: "<2-3 sentence summary of the use case>"
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
```

- File: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`
- Created via `hugo new content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`
- No `<!--more-->` shortcode (matches newest post style), starts directly with a narrative `##` section.

## Narrative hook

Open with the real customer problem this solves: needing to track VM lifecycle
(creation/deletion) into a queryable system, framed generically without customer
names — matching how the function repo's own README already frames it ("This
function is based on a real customer use case"). Contrast polling vs. event-driven,
then introduce FaaS as the execution model and Knative Eventing as the backbone.

## Content outline

1. **Intro** — the customer problem, why event-driven beats polling, FaaS + Knative Eventing framing.
2. **Architecture at a glance** — embed `kubevirt-meets-eventing.png` diagram, walk through the flow: K8s API Server → `ApiServerSource` → `Broker` → `EventTransform` (JSONata) → `Trigger`(s) filtered on `resource.add`/`resource.delete`) → Function → PostgreSQL → web-frontend for reads.
3. **What is the Event Transformer?** — admonition callout on why raw K8s API events are noisy and how Event Transform trims them (reuse framing/example payload from the function repo README).
4. **Setting the stage: Brokers & RBAC** — Broker manifests (apiserversource + eventtransform brokers), ServiceAccount/Role/RoleBinding for watching `virtualmachines`/`virtualmachineinstances`.
5. **Wiring up the ApiServerSource** — manifest + explanation of `mode: Resource`.
6. **Transforming the event** — `EventTransform` JSONata manifest; before/after payload comparison (raw K8s event vs. trimmed CloudEvent, using the sample from the function repo README).
7. **Triggers: routing add/delete events** — the two `Trigger` manifests filtering on `dev.knative.apiserver.resource.add` / `.delete`.
8. **The PostgreSQL backend** — condensed StatefulSet/PVCs/Service (trimmed vs. the full ~460 lines in the README) + the `Job` that initializes the `vmdb` database/table.
9. **Deploying the `kn-py-vmdata-psql-fn` function** — secret creation, Knative Service + Triggers wiring the function to the transformed broker; link out to the repo for full source; mention duplicate-event skipping as a nice detail.
10. **Validating end-to-end** — creating VMs via `oc process | oc apply`, checking rows via `psql`, pgAdmin screenshot placeholder.
11. **Closing the loop: a web frontend to read the data** — deploy `postgresql-read-webapp` as a Knative Service, screenshot placeholder of the UI.
12. **Wrap-up & Resources** — recap the FaaS/event-driven value prop, possible next steps, links to both repos + Knative Eventing docs.

## Images

- Cover image: copy `kubevirt-meets-eventing.png` → `static/img/kubevirt_meets_eventing_cover.png`
- Same image reused inline in section 2 as the architecture diagram, placed at
  `static/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png`
- Placeholder `{{< image >}}` shortcodes (empty `src`, populated `caption`) for:
  - psql / pgAdmin query output screenshot
  - web-frontend UI screenshot
  Named `rguske-post-kubevirt-eventing-N.png`, to be filled in later by the user.

## Style notes (from existing posts)

- Personal, first-person narrative tone with light humor/emoji shortcodes (`:smile:`, `:rocket:`, etc.) used sparingly.
- `{{< admonition info|note "Title" true >}}...{{< /admonition >}}` shortcodes for callouts.
- `{{< image src="..." caption="Figure N: ..." src-s="..." >}}` shortcode for images.
- GitHub repo links formatted as: `<i class='fab fa-github fa-fw'></i> repository :point_right: [org/repo](url)`
- Code blocks tagged with `yaml`/`shell`/`code`/`json` as appropriate, generally reproducing real terminal output where available.
- Optional trailing `## Resources` section with links.

## Out of scope

- No fabrication of customer names or confidential details.
- Not pasting the full ~460-line PostgreSQL StatefulSet verbatim — condensed with a link back to the README/repos for full manifests.
- Not designing new screenshots/diagrams beyond the one existing excalidraw image — placeholders are used for anything the user hasn't captured yet.
