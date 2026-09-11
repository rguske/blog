# KubeVirt meets Eventing Blog Post Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish a new (draft) Hugo blog post, `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`, that walks through an end-to-end, OpenShift-flavored, reproducible deployment of event-driven VM lifecycle tracking with KubeVirt, Knative Eventing (ApiServerSource → Broker → EventTransform → Trigger), a Python Knative Function writing to PostgreSQL, and a companion read-only web frontend.

**Architecture:** One Markdown file, built incrementally section-by-section in the order defined by the design spec's Content Outline (12 sections, grouped into 9 tasks below). Two supporting images are copied into `static/`. Each task appends a contiguous, self-contained chunk of the post and is verified by running `hugo build -D` (drafts included) to catch shortcode/front-matter/Markdown errors, plus a manual checklist match against the design spec for that section.

**Adaptation note:** This is a content-authoring plan, not a software plan. There is no automated test suite for prose. "Verification" per task = (a) `hugo build -D` succeeds with no errors referencing this post's file, and (b) the section covers every bullet point in its content brief below (self-checked against the spec). Manifests, commands, and terminal output quoted in the post must be copied verbatim from the **Sources** listed per task — never invented. Because tone/voice consistency matters across the whole piece, this plan is intended for **inline execution by a single writer/agent in one continuous pass**, not parallel subagents per task (see Execution Handoff).

**Tech Stack:** Hugo (LoveIt theme), Markdown, Hugo shortcodes (`admonition`, `image`, `mermaid` not used here — static image instead).

## Global Constraints

- Style: first-person narrative tone, light emoji shortcodes (`:smile:`, `:rocket:`, etc.) used sparingly, matching `content/posts/bootable-containers-an-entire-operating-system-as-a-containerfile.md`.
- No `<!--more-->` shortcode — start directly with a narrative `##` section (current style).
- GitHub repo links formatted as: `<i class='fab fa-github fa-fw'></i> repository :point_right: [org/repo](url)`.
- Callouts use `{{< admonition info|note "Title" true >}} ... {{< /admonition >}}`.
- Images use `{{< image src="..." caption="Figure N: ..." src-s="..." >}}`.
- No fabrication of customer names or confidential details — the customer use case stays generic.
- Do not paste the full ~460-line PostgreSQL StatefulSet verbatim — condense, link back to the README/repo for the rest.
- `draft: true` in front matter until the user explicitly asks to publish.
- Front matter fields/order must match: `author`, `authorLink`, `lightgallery`, `title`, `description`, `date`, `draft`, `featuredImage`, `categories`, `tags`.
- Figure numbering (`Figure I`, `Figure II`, ...) is sequential across the whole post, Roman numerals, matching existing posts.

---

## Task 1: Scaffold the post file, front matter, and copy images

**Files:**
- Create: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md` (via `hugo new`)
- Create: `static/img/kubevirt_meets_eventing_cover.png` (copy)
- Create: `static/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png` (copy)

**Sources:**
- Diagram/cover source: `/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/excalidraw/kubevirt-meets-eventing.png`
- Front matter convention: `content/posts/bootable-containers-an-entire-operating-system-as-a-containerfile.md:1-19`

**Interfaces:**
- Produces: the post file with valid front matter that all later tasks append `##` sections to; the two image paths (`/img/kubevirt_meets_eventing_cover.png` and `/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png`) referenced by Task 2.

- [ ] **Step 1: Scaffold the post via Hugo**

```bash
cd "/Users/rguske/Documents/blog"
hugo new content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
```

- [ ] **Step 2: Replace the generated front matter**

Replace the auto-generated front matter block with:

```yaml
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
```

Leave the rest of the file body empty (later tasks append content directly below this block, no `<!--more-->`).

- [ ] **Step 3: Copy the cover image and diagram image**

```bash
cp "/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/excalidraw/kubevirt-meets-eventing.png" \
   "/Users/rguske/Documents/blog/static/img/kubevirt_meets_eventing_cover.png"

mkdir -p "/Users/rguske/Documents/blog/static/img/posts/202512_kubevirt_meets_eventing"
cp "/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/excalidraw/kubevirt-meets-eventing.png" \
   "/Users/rguske/Documents/blog/static/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png"
```

- [ ] **Step 4: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```

Expected: no errors printed for this post's file (Hugo build completes; front matter parses cleanly).

- [ ] **Step 5: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md \
        static/img/kubevirt_meets_eventing_cover.png \
        static/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png
git commit -m "post: scaffold KubeVirt meets Eventing draft with front matter and images"
```

---

## Task 2: Intro + Architecture-at-a-glance sections

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md` (append after front matter)

**Sources:**
- Original article intro framing: uploaded doc `kubevirt_meets_eventing-0.md` (paragraphs under "Monitoring Virtual Machines with Knative Eventing", authors line, EDA framing paragraph)
- Customer-problem framing: `kn-py-vmdata-psql-fn` README ("This function is based on a real customer use case...")
- Diagram: `/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png` (from Task 1)

**Interfaces:**
- Consumes: front matter + image paths from Task 1.
- Produces: `## From Concept to Reality` and `## Architecture at a Glance` headings (anchors `#from-concept-to-reality`, `#architecture-at-a-glance`) that the Resources section (Task 9) links back to conceptually.

**Content brief (write this prose during execution, hitting every bullet):**

- [ ] **Step 1: Write the "origin story" callout + intro**

Append a `##` section (choose a fitting header, e.g. `## From Concept to Reality`) that:
1. Opens with an `{{< admonition note "Where this started" true >}}` block crediting the original co-authored article — must literally link `[Monitoring Virtual Machines with Knative Eventing](https://knative.dev/blog/articles/kubevirt_meets_eventing/)` and name co-author Matthias Weßendorf — and states this post is the deeper, hands-on, OpenShift-flavored companion with full reproducible manifests plus a new web-frontend addition.
2. Below the admonition, narrates the real-world problem generically: teams running VMs on KubeVirt need an automatic, always-up-to-date record (e.g., a CMDB-like PostgreSQL table) of which VMs exist, with what specs, without polling.
3. Explicitly introduces "Functions-as-a-Service" as the execution model and Knative Eventing as the backbone connecting Kubernetes-native events to that function.

- [ ] **Step 2: Write the architecture section**

Append `## Architecture at a Glance` containing:
1. The image: `{{< image src="/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png" caption="Figure I: End-to-end event flow from VM lifecycle event to database" src-s="/img/posts/202512_kubevirt_meets_eventing/kubevirt-meets-eventing.png" >}}`
2. A short walk-through of the diagram's flow, naming each hop: Kubernetes API Server (event producer) → `ApiServerSource` → `Broker` → `EventTransform` → two `Trigger`s (filtered on `dev.knative.apiserver.resource.add` / `.delete`) → the `kn-py-vmdata-psql-fn` Function → PostgreSQL → (new, not in original diagram) the read-only web frontend.
3. One sentence flagging that the web-frontend piece is this post's addition on top of the original article's scope.

- [ ] **Step 3: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 4: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add intro and architecture-at-a-glance sections"
```

---

## Task 3: "What is the Event Transformer?" explainer section

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- `kn-py-vmdata-psql-fn` README: "Event-driven systems like Kubernetes produce a massive amount of events... The Knative Event-Transformer functionality trims the fat..." + the transformed example JSON payload shown there.
- Original article section "Trimming the fat from the Event-Payload" for the raw (untransformed) `ApiServerSource` CloudEvent example (`dev.knative.apiserver.resource.add`, `rhel-vm-2` example) — reuse structure/framing, not verbatim customer data.

**Interfaces:**
- Consumes: `## Architecture at a Glance` heading from Task 2 (this section follows immediately after it).
- Produces: `## What Is the Event Transformer?` heading, and the "raw vs. transformed" payload contrast that Task 6 (EventTransform manifest) will refer back to.

- [ ] **Step 1: Write the explainer section**

Append `## What Is the Event Transformer?` with:
1. One paragraph explaining that `ApiServerSource` forwards *very* verbose raw Kubernetes API objects as CloudEvents, making downstream processing (e.g., a small Python function) unnecessarily hard.
2. A fenced ` ```json ` block showing a trimmed/representative excerpt of the **raw** `dev.knative.apiserver.resource.add` event body (context attributes + a few key `data.spec...` fields: `cpu.cores`, `cpu.sockets`, `memory.guest`, `dataVolumeTemplates...storage`, `storageClassName`, `networks.name`) — model it on the original article's example but keep it short (don't reproduce the full nested JSON).
3. A sentence introducing `EventTransform` (Knative Eventing v1.18+) as a JSONata-based CRD that reshapes/trims events in-flight, sitting anywhere in the event flow.
4. Note this will be configured in the next sections to emit exactly the columns the `vmdb` PostgreSQL table needs.

- [ ] **Step 2: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 3: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add Event Transformer explainer section"
```

---

## Task 4: Event pipeline manifests (Brokers, RBAC, ApiServerSource, EventTransform, Triggers)

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources (copy manifests verbatim from):**
- `/Users/rguske/Documents/README.md` lines 1–224 (Brokers, ServiceAccount/Role/RoleBinding, ApiServerSource, EventTransform, Triggers `trigger-transformer-vm-add` / `trigger-transformer-vm-delete`)

**Interfaces:**
- Consumes: the raw/transformed payload contrast from Task 3.
- Produces: the deployed `broker-apiserversource`, `broker-eventtransform` Broker names, and `vmdata-transform` EventTransform name — referenced again in Task 6 (function's Trigger targets `broker-eventtransform`).

- [ ] **Step 1: Write "Setting the Stage: Brokers & RBAC" subsection**

Append `## Deploying the Event Pipeline` as a parent heading, then `### Setting the Stage: Brokers & RBAC` with:
1. One sentence on why two Brokers are used (one for raw `ApiServerSource` events, one for transformed events).
2. The two `Broker` manifests (`broker-apiserversource`, `broker-eventtransform`) as a single ` ```yaml ` block, copied from README.md lines 1–20.
3. One sentence on why RBAC is needed (the `ApiServerSource` needs to `get/list/watch` `virtualmachines`/`virtualmachineinstances`).
4. The `ServiceAccount`/`Role`/`RoleBinding` manifest as a ` ```yaml ` block, copied from README.md lines 20–53.

- [ ] **Step 2: Write "Wiring Up the ApiServerSource" subsection**

Append `### Wiring Up the ApiServerSource` with:
1. One paragraph explaining `mode: Resource` (watches full-resource state, not just generic K8s events) and that it's scoped to `kubevirt.io/v1` `VirtualMachine`.
2. The `ApiServerSource` manifest as a ` ```yaml ` block, copied from README.md lines 56–74, pointed at `broker-apiserversource`.

- [ ] **Step 3: Write "Transforming the Event" subsection**

Append `### Transforming the Event` with:
1. The `EventTransform` manifest (`vmdata-transform`) as a ` ```yaml ` block, copied from README.md lines 110–144, sinking to `broker-eventtransform`.
2. A short paragraph mapping each JSONata field (`cpucores`, `cpusockets`, `memory`, `datasource`, `storageclass`, `network`) back to the raw fields shown in Task 3's example.
3. A fenced ` ```json ` (or plain text `Context Attributes,` style) block showing the **transformed**, trimmed event, modeled on the "Example custom (transformed) event payload" in the original Knative article — reformatted for this post's field names.

- [ ] **Step 4: Write "Triggers: Routing Add/Delete Events" subsection**

Append `### Triggers: Routing Add/Delete Events` with:
1. One paragraph: a `Trigger` connects a `Broker` + event-type filter to a subscriber; here two Triggers route `dev.knative.apiserver.resource.add` and `.delete` from `broker-apiserversource` to the `vmdata-transform` EventTransform.
2. The two `Trigger` manifests (`trigger-transformer-vm-add`, `trigger-transformer-vm-delete`) as a single ` ```yaml ` block, copied from README.md lines 180–224.

- [ ] **Step 5: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 6: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add event pipeline manifests (Brokers, RBAC, ApiServerSource, EventTransform, Triggers)"
```

---

## Task 5: PostgreSQL backend section

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- `/Users/rguske/Documents/README.md` lines 308–451 (PostgreSQL `Secret`, `PersistentVolumeClaim`s, `StatefulSet`, `Service`) — condense, do not paste all ~140 lines verbatim.
- `/Users/rguske/Documents/README.md` lines 494–574 (Kubernetes `Job` `init-vmdb` that creates the `vmdb` database and `virtual_machines` table) — copy verbatim, this is short and important for readers to reproduce.

**Interfaces:**
- Consumes: nothing new (standalone infra section).
- Produces: the `vmdb` database + `virtual_machines` table referenced by Task 6 (function env vars) and Task 7 (validation queries).

- [ ] **Step 1: Write condensed PostgreSQL deployment subsection**

Append `## The PostgreSQL Backend` with:
1. One paragraph stating a StatefulSet-backed PostgreSQL 16 instance is used, with a note that this is homelab/demo-grade (single replica, `ReadWriteOnce` PVCs) — link out: "the full StatefulSet manifest is in my [notes]" is not needed; instead say readers can adapt any reachable Postgres instance, since the interesting part is the schema.
2. Only the `Secret` manifest (README.md lines 310–322) as a ` ```yaml ` block — small and necessary (`POSTGRES_PASSWORD`).
3. Skip pasting the full PVC/StatefulSet/Service YAML; instead one sentence: "Standard `StatefulSet` + `PersistentVolumeClaim`s + `ClusterIP Service` — nothing KubeVirt/Knative-specific here, so I won't paste all ~140 lines; adapt your own Postgres instance or Operator of choice."

- [ ] **Step 2: Write the DB/table init subsection**

Append `### Initializing the `vmdb` Database` with:
1. One sentence: a one-off Kubernetes `Job` creates the `vmdb` database and the `virtual_machines` table with columns matching the `EventTransform` JSONata output field-for-field.
2. The `Job` manifest as a ` ```yaml ` block, copied verbatim from README.md lines 494–574 (image `registry.redhat.io/rhel9/postgresql-15`, `CREATE TABLE IF NOT EXISTS public.virtual_machines (...)`).

- [ ] **Step 3: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 4: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add PostgreSQL backend section"
```

---

## Task 6: Deploying the `kn-py-vmdata-psql-fn` function

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- `/Users/rguske/Documents/README.md` lines 576–674 (`psql-secret` creation command, function `Service` manifest, `trigger-vm-add` / `trigger-vm-delete` Triggers targeting `broker-eventtransform`)
- [`rguske/knative-functions/kn-py-vmdata-psql-fn` README](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn) — for the GitHub repo link, the duplicate-event-skipping behavior description/example log output, and the local `podman run` + `curl` test example.

**Interfaces:**
- Consumes: `broker-eventtransform` from Task 4, `vmdb`/`virtual_machines` schema from Task 5.
- Produces: the deployed `kn-py-psql-vmdata-fn` Knative Service name, referenced by Task 7's validation commands.

- [ ] **Step 1: Write function intro + repo link**

Append `## Deploying the kn-py-vmdata-psql-fn Function` with:
1. One paragraph: the function is written in Python, receives the transformed CloudEvent, and writes/deletes the corresponding row in `virtual_machines`; it also skips duplicate event IDs (cite the repo's duplicate-detection behavior).
2. The repo link line: `<i class='fab fa-github fa-fw'></i> repository :point_right: [rguske/knative-functions/kn-py-vmdata-psql-fn](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn)`

- [ ] **Step 2: Write secret + deployment subsection**

Append content with:
1. The `psql-secret` creation command as a ` ```shell ` block, copied from README.md lines 578–585.
2. The function's Knative `Service` manifest as a ` ```yaml ` block, copied from README.md lines 590–634 (image `quay.io/rguske/kn-py-psql-vmdata-fn:v1.0`, env vars from `psql-secret`).
3. The two Triggers (`trigger-vm-add`, `trigger-vm-delete`) targeting `broker-eventtransform`, as a ` ```yaml ` block, copied from README.md lines 636–674.

- [ ] **Step 3: Write a short "duplicate events" callout**

Append `{{< admonition info "Skipping duplicate events" true >}} ... {{< /admonition >}}` summarizing (in your own words, not verbatim copy) the function repo README's duplicate-event log example (`🟡 Duplicate event detected, skipping`) and why it matters for at-least-once delivery semantics.

- [ ] **Step 4: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 5: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add kn-py-vmdata-psql-fn function deployment section"
```

---

## Task 7: Validating end-to-end

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- `/Users/rguske/Documents/README.md` lines 676–730 (VM creation loop is not in README directly but is present in the function repo README: `for i in $(seq 1 5); do oc process ...`) — use the function repo README's exact validation loop and `psql` output table.
- `kn-py-vmdata-psql-fn` README "Validating the Functionality" section — for the `oc process | oc apply` VM-creation loop and the 10-row `psql` output table.
- `/Users/rguske/Documents/README.md` lines 696–704 (pgAdmin container run command)

**Interfaces:**
- Consumes: `kn-py-psql-vmdata-fn` Service name from Task 6.
- Produces: confirmed populated `virtual_machines` table, consumed narratively by Task 8 (web frontend reads this same table).

- [ ] **Step 1: Write the VM creation + psql validation subsection**

Append `## Validating End-to-End` with:
1. One sentence introducing the validation approach: create/delete a few VMs, then query the DB directly.
2. The VM-creation loop command as a ` ```shell ` block: `for i in $(seq 1 5); do oc process -n openshift rhel9-server-medium -p NAME=vm${i} | oc apply -f - ; done;` (from the function repo README).
3. The `psql` query command + the 10-row example output table as a ` ```shell ` block, copied verbatim from the function repo README's "Validating the Functionality" section (5 `resource.add` + 5 `resource.delete` rows).

- [ ] **Step 2: Write the pgAdmin alternative subsection**

Append `### Reading the Data with pgAdmin (Optional)` with:
1. One sentence: pgAdmin is a nicer alternative to `psql` for browsing the table.
2. The `podman run` pgAdmin command as a ` ```shell ` block, copied from README.md lines 699–704.
3. An image placeholder: `{{< image src="" caption="Figure II: virtual_machines table browsed in pgAdmin" src-s="" >}}`

- [ ] **Step 3: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors (empty `src=""` in the image shortcode is valid Markdown/Hugo, just renders a broken image icon — acceptable for a draft placeholder).

- [ ] **Step 4: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add end-to-end validation section"
```

---

## Task 8: Closing the loop — the web frontend

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- [`rguske/postgresql-read-webapp`](https://github.com/rguske/postgresql-read-webapp) README — `kn service create` command, description of the app (simple read-only web UI over the `vmdb` Postgres DB).
- `/Users/rguske/Documents/README.md` lines 686–694 (`pg-credentials` secret + `kn service create postgresql-read-webapp` command)

**Interfaces:**
- Consumes: the populated `virtual_machines` table from Task 7.
- Produces: nothing further consumed (last content section before wrap-up).

- [ ] **Step 1: Write the web frontend section**

Append `## Closing the Loop: A Web Frontend to Read the Data` with:
1. One paragraph: introduce this as the piece not covered by the original Knative blog article — a lightweight read-only web app so non-`psql` users can browse VM lifecycle history too.
2. The repo link line: `<i class='fab fa-github fa-fw'></i> repository :point_right: [rguske/postgresql-read-webapp](https://github.com/rguske/postgresql-read-webapp)`
3. The `pg-credentials` secret command as a ` ```shell ` block, copied from README.md lines 676–681.
4. The `kn service create postgresql-read-webapp` command as a ` ```shell ` block, copied from README.md lines 686–694.
5. An image placeholder: `{{< image src="" caption="Figure III: postgresql-read-webapp displaying the virtual_machines table" src-s="" >}}`

- [ ] **Step 2: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 3: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add web frontend section"
```

---

## Task 9: Wrap-up, Resources, and final self-review pass

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`

**Sources:**
- All prior sections (internal consistency pass).
- [Original Knative blog article](https://knative.dev/blog/articles/kubevirt_meets_eventing/), [`rguske/knative-functions`](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn), [`rguske/postgresql-read-webapp`](https://github.com/rguske/postgresql-read-webapp), [Knative Eventing docs](https://knative.dev/docs/eventing/).

**Interfaces:**
- Consumes: every heading produced by Tasks 2–8 (final read-through).
- Produces: the finished draft post, ready for the user's own screenshot pass and eventual `draft: false` flip.

- [ ] **Step 1: Write the wrap-up section**

Append `## Wrap-Up` with:
1. A short recap: event-driven beats polling; `ApiServerSource` + `Broker` + `EventTransform` + `Trigger` is a reusable pattern for *any* Kubernetes custom resource, not just KubeVirt VMs.
2. One or two sentences on possible next steps (e.g., alerting on VM deletion, extending to other KubeVirt resource types, swapping the in-memory Broker for a Kafka-backed one for production).

- [ ] **Step 2: Write the Resources section**

Append `## Resources` with a bullet list linking:
- [Monitoring Virtual Machines with Knative Eventing](https://knative.dev/blog/articles/kubevirt_meets_eventing/) (the original article)
- [rguske/knative-functions/kn-py-vmdata-psql-fn](https://github.com/rguske/knative-functions/tree/main/kn-py-vmdata-psql-fn)
- [rguske/postgresql-read-webapp](https://github.com/rguske/postgresql-read-webapp)
- [Knative Eventing docs](https://knative.dev/docs/eventing/)
- [KubeVirt](https://kubevirt.io/)

- [ ] **Step 3: Full self-review pass**

Read the entire post top to bottom and confirm against the design spec (`docs/superpowers/specs/2026-09-11-kubevirt-meets-eventing-design.md`):
1. Every one of the 12 outlined content-outline items is present.
2. Figure numbering is sequential (I, II, III) and each `{{< image >}}` shortcode has a matching, non-duplicate caption.
3. No customer names or confidential details appear anywhere.
4. All GitHub/external links use the correct URLs listed in this plan (grep for `github.com/rguske` and `knative.dev` to double check).
5. Front matter `description` still accurately summarizes the final content.

- [ ] **Step 4: Final build verification**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | tail -30
```
Expected: build completes with no errors/warnings tied to this post's file or its images.

- [ ] **Step 5: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add wrap-up, resources, and finalize KubeVirt meets Eventing draft"
```

---

## Post-plan follow-ups (not part of this plan)

- User to capture and drop in the two placeholder screenshots (pgAdmin table view, web-frontend UI), then update the two empty `{{< image src="" ... >}}` shortcodes in Tasks 7 and 8.
- User to flip `draft: true` → `draft: false` when ready to publish.

---

## Task 4a (added post-hoc): Prerequisites section

**Inserted between the existing "What Is the Event Transformer?" section (Task 3) and "## Deploying the Event Pipeline" (Task 4).** Requested by the user after Tasks 1-9 were already complete: a reader following this post needs to know how to get Knative Serving/Eventing (or OpenShift Serverless) installed before the manifests in "Deploying the Event Pipeline" will apply cleanly.

**Files:**
- Modify: `content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md` (insert new `## Prerequisites` section immediately before the existing `## Deploying the Event Pipeline` heading)

**Sources (verified against live Red Hat docs on 2026-09-11):**
- [Installing the OpenShift Serverless Operator (CLI)](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.37/html/installing_openshift_serverless/install-serverless-operator) — `Namespace`/`OperatorGroup`/`Subscription` YAML, `oc apply -f serverless-subscription.yaml`, verify via `oc get csv`.
- [Installing Knative Serving by using YAML](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.37/html/installing_openshift_serverless/installing-knative-serving) — `KnativeServing` CR, `oc apply -f serving.yaml`, verify via `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template=...`.
- [Installing Knative Eventing by using YAML](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.37/html/installing_openshift_serverless/installing-knative-eventing) — `KnativeEventing` CR, `oc apply -f eventing.yaml`, verify via `oc get knativeeventing.operator.knative.dev/knative-eventing -n knative-eventing --template=...`.
- [Knative Quickstart](https://knative.dev/docs/install/quickstart-install/) and the [Knative Operator install docs](https://knative.dev/docs/install/operator/knative-with-operators/) — for the one-paragraph "not on OpenShift?" callout.

**Interfaces:**
- Consumes: nothing (standalone infra prerequisite section); positioned right after Task 3's "What Is the Event Transformer?" section.
- Produces: confirms `knative-serving` and `knative-eventing` namespaces/CRs exist, which every manifest from Task 4 onward assumes is already true.

- [ ] **Step 1: Write the Prerequisites section**

Insert `## Prerequisites` (as its own top-level section, before `## Deploying the Event Pipeline`) with:
1. One sentence: everything from here on assumes Knative Serving and Eventing (or, on OpenShift, the OpenShift Serverless Operator) plus KubeVirt/OpenShift Virtualization are already installed and healthy on the cluster — this section covers the Knative/Serverless side only (KubeVirt install is out of scope, link to [KubeVirt's own quickstart](https://kubevirt.io/quickstart_minikube/) or OpenShift Virtualization docs for readers who need it).
2. `### On OpenShift: the OpenShift Serverless Operator` subsection:
   - One sentence: the Operator manages both Knative Serving and Eventing (and the Kafka broker) as a single product, so it's the fastest path on OpenShift.
   - The `Namespace`/`OperatorGroup`/`Subscription` YAML as a ```yaml``` block (verbatim from the Red Hat doc above), followed by `oc apply -f serverless-subscription.yaml` as a ```shell``` block, then the `oc get csv` verification command + its expected example output line.
   - The `KnativeServing` CR YAML as a ```yaml``` block, `oc apply -f serving.yaml`, and the verification command `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'` with its expected `...=True` output lines.
   - The `KnativeEventing` CR YAML as a ```yaml``` block, `oc apply -f eventing.yaml`, and the equivalent verification command for `knativeeventing`.
3. `### Everywhere Else: Upstream Knative Serving and Eventing` subsection:
   - One short paragraph: on non-OpenShift Kubernetes, install upstream Knative Serving and Eventing directly — either via the [Knative Operator](https://knative.dev/docs/install/operator/knative-with-operators/) for a supported long-term install, or the [Knative Quickstart](https://knative.dev/docs/install/quickstart-install/) `kn` plugin for local experimentation only (not production).
   - No need to paste upstream YAML manifests in full — link out, this post's actual manifests from here on use `oc`/OpenShift conventions.

- [ ] **Step 2: Verify build**

```bash
cd "/Users/rguske/Documents/blog" && hugo build -D 2>&1 | grep -i "kubevirt-meets-eventing"
```
Expected: no errors.

- [ ] **Step 3: Commit**

```bash
git add content/posts/kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md
git commit -m "post: add Prerequisites section (OpenShift Serverless / upstream Knative install)"
```
