# Blog Post Design: "Bootable Containers - An Entire Operating System as a Containerfile"

## Summary

Create a new Hugo blog post at `content/posts/` adapting the learning guide
from `bootable-containers/README.md` (and `bootable-containers/cicd/README.md`)
into a post that matches Robert Guske's established writing style, and ties
the post to his ContainerDays 2026 Hamburg talk on the same topic.

## Source Material

- Primary: `/Users/rguske/robert.guske@googlemail.com - Google Drive/My Drive/dev/openshift/bootable-containers/README.md`
- Secondary (CI/CD section): `.../bootable-containers/cicd/README.md`
- Screenshots available in `.../bootable-containers/static/`:
  `webpage.png`, `webpage2.png`, `github-actions.png`, `github-actions2.png`,
  `pipelinerun1.png`, `pipelinerun2.png`, `gitops1.png`
- Context: LinkedIn post about inheriting the ContainerDays 2026 Hamburg
  session from Cedric Clyburn and Paulo Menon —
  https://www.linkedin.com/posts/robertguske_containerdays-cds26-hamburg-activity-7500564361191727105-8sj2

## Style Reference (derived from existing posts)

Reviewed: `installing-red-hat-openshift-in-a-disconnected-air-gapped-environment.md`,
`red-hat-anniversary.md`, `database-resurrection-reviving-postgres-on-vmware-vcenter.md`,
`youtube-hugo-shortcode-workaround-when-youtube-wont-work.md`, and the
Hugo archetype at `archetypes/default.md`.

Conventions to follow:
- Front matter fields: `author`, `authorLink`, `lightgallery: true`, `title`,
  a rich 2-3 sentence `description`, `date` (ISO w/ offset), `draft`,
  `featuredImage` (`/img/<slug>_cover.png`), `categories` (array), `tags` (list).
- First-person, casual-technical tone; personal narrative hook before/around
  the first `## ` heading; dry humor; emoji shortcodes (`:fire:`,
  `:sweat_smile:`, `:white_check_mark:`, etc.); occasional German-English
  quirks left as-is.
- Structure: personal hook -> concepts -> resources/references (quoted
  blurbs + links) -> hands-on section(s) with real shell/code blocks and
  terminal output -> troubleshooting/"battle scars" -> `## Conclusion` ->
  "Thanks for reading."
- Shortcodes: `{{< admonition info|note|warning|quote|success "Title" true >}}...{{< /admonition >}}`
  and `{{< image src="..." caption="Figure I: ..." src-s="..." >}}`.
- Images stored under `static/img/posts/YYYYMM_topic/` in the blog repo.

## Scope Decisions (confirmed with user)

1. **Depth:** Concept overview + Demo 1 (build the bootc web server) +
   Demo 3 (convert to qcow2, package, deploy to OpenShift Virtualization,
   with explicit explanation of how the qcow2 disk becomes a PVC via a
   `DataVolume`) + a brief nod to Demo 4 (atomic upgrade/rollback).
2. **CI/CD:** Cover all three pipeline flavors from `cicd/README.md`
   (GitHub Actions, OpenShift Pipelines/Tekton, GitOps/ArgoCD) briefly —
   one paragraph + one screenshot each. No deep Tekton troubleshooting
   walkthrough.
3. **Troubleshooting:** Include a short, curated "battle scars" section
   with 2-3 war stories pulled from the README/cicd README, not the full
   troubleshooting log:
   - `bootc-image-builder` requiring a native Linux x86_64 host (doesn't
     work reliably via Podman Machine on macOS).
   - `bootc-image-builder no longer pulls automatically` / `image not known`
     gotcha (must `podman pull` first).
   - RWX + `volumeMode: Block` requirement for live migration on this
     cluster's block storage class.
4. **Intro:** Must reference the ContainerDays 2026 Hamburg talk — inherited
   from Cedric Clyburn (https://www.linkedin.com/in/cedricclyburn/) and
   Paulo Menon (https://www.linkedin.com/in/paulomenon/), delivered last
   week, recorded, recording to follow (placeholder note, no URL yet).
   Link to the LinkedIn post for full context.
5. **Cover image:** Generated (not a screenshot) — a minimalist dark
   illustration of a shipping container with a terminal prompt "booting"
   into a server, red/white/black palette. Saved to the blog repo as
   `static/img/bootable_containers_cover.png`.

## Front Matter

```yaml
---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Bootable Containers - An Entire Operating System as a Containerfile"
description: "A hands-on look at bootable containers (bootc): building an entire OS as a Containerfile, converting it to a bootable disk image, deploying it to OpenShift Virtualization, and automating the whole pipeline with GitHub Actions, OpenShift Pipelines, and GitOps."
date: 2026-09-08T13:00:00+02:00
draft: true
featuredImage: /img/bootable_containers_cover.png
categories: ["Platforms", "Open-Source", "Virtualization"]
tags:
- RedHat
- OpenShift
- Containers
- Linux
- bootc
- KubeVirt
- Automation
---
```

## Outline

1. `## From an Inherited Session in Hamburg to This Post` — ContainerDays
   2026 Hamburg story, link to LinkedIn post, note recording is pending.
2. `## What Are Bootable Containers?` — Package Mode vs Image Mode,
   comparison table, "why bootable containers" bullet list.
3. `## Key Components` — `bootc` (status/switch/upgrade/rollback),
   `bootc-image-builder` (output types table), base images used.
4. `## Demo 1: Building a Bootable Web Server` — Containerfile-driven
   multi-arch build & push, quick local `podman run` smoke test.
5. `## From Container Image to Virtual Machine` — convert to qcow2
   (macOS/Linux caveat), package as `Containerfile.ocpv`, deploy via
   `oc apply -k openshift-virtualization/`, explain the DataVolume -> PVC
   -> VM flow explicitly. Include `{{< image >}}` of `webpage.png`.
6. `## Bonus: Atomic Upgrade & Rollback` — short callout of `bootc switch`
   / `bootc rollback` (v1.0 red -> v1.1 purple -> rollback). Include
   `{{< image >}}` of `webpage2.png`.
7. `## Automating It: Three Flavors of CI/CD` — one paragraph + one
   screenshot each for GitHub Actions (`github-actions2.png`), OpenShift
   Pipelines/Tekton (`pipelinerun1.png`/`pipelinerun2.png`), GitOps/ArgoCD
   (`gitops1.png`).
8. `## Battle Scars` — the 3 curated troubleshooting items above, in
   admonition/code-block style.
9. `## Conclusion` + "Thanks for reading."
10. `## References` — pulled from the README's References section.

## File/Asset Changes

- Create: `content/posts/bootable-containers-an-entire-operating-system-as-a-containerfile.md`
  (via `hugo new content/posts/<slug>.md` from the archetype, then filled in).
- Create: `static/img/bootable_containers_cover.png` (generated cover image).
- Create: `static/img/posts/202609_bootablecontainers/` with copies of
  `webpage.png`, `webpage2.png`, `github-actions2.png`, `pipelinerun1.png`,
  `pipelinerun2.png`, `gitops1.png` from the `bootable-containers` repo.

## Out of Scope

- Full reproduction of every command/step from the README (all 4 demos in
  full detail, complete Tekton troubleshooting log, full environment
  variable tables) — the post links to the `bootable-containers` GitHub
  repo for the exhaustive version.
- Publishing (`draft: false`) — left as `draft: true` for the user to
  review and flip before publishing.
- Embedding the ContainerDays talk recording — not yet available; a
  placeholder note is included instead.
