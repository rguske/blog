# Blog Post Design: "Introducing Project RBAC-Generator - A UI for Kubernetes RBAC"

## Summary

Create a short Hugo announcement post for [RBAC-Generator](https://github.com/rguske/rbac-generator). The first half says what the tool solves, that the UI is PatternFly 6, and which features it has. The second half is three copy-paste recipes: build the container image, run the published image, and deploy with the kustomize base. A few sentences at the end cover contributing.

The body uses the name **RBAC-Generator**, matching the repository. The word "Project" appears only in the title.

## Source Material

- Primary: `projects/rbac-generator/README.md` in the dev workspace.
- Screenshots: `projects/rbac-generator/static/rbac-generator1.png` (login), `rbac-generator2.png` (create), `rbac-generator3.png` (templates).
- Build facts: `Makefile` (`VERSION ?= v1.0`, `PLATFORMS ?= linux/amd64,linux/arm64`, `make image`, `make hash-password`).
- Deploy facts: `deploy/kustomize/base/` (`deployment.yaml`, `service.yaml`, `route.yaml`, `secret.example.yaml`). `secret.yaml` is gitignored and is not quoted in the post.

## Style Reference

Reviewed posts in `/Users/rguske/Documents/blog/content/posts/`:

- `bootable-containers-an-entire-operating-system-as-a-containerfile.md`
- `kubevirt-meets-eventing-automating-vm-lifecycle-data-with-knative-and-faas.md`
- `red-hat-anniversary.md`

Conventions for this post:

- Front matter matches recent posts: `author`, `authorLink`, `lightgallery: true`, `title`, `description`, `date` with `+02:00`, `draft`, `featuredImage`, `categories`, `tags`.
- First person, talking to the reader. Contractions. One idea per short paragraph. Resource names and commands in backticks.
- No origin story and no invented anecdote.
- One `admonition` info box for the core idea, titled "The core idea".
- Repository line: `<i class='fab fa-github fa-fw'></i> repository :point_right:` plus the link to `https://github.com/rguske/rbac-generator`.
- Each recipe: one sentence on why the commands are there, then a `shell` block.
- The build section includes a three-row table of UBI9 stages.
- Figures use captions "Figure I:", "Figure II:", "Figure III:" and the `image` shortcode with both `src` and `src-s`.
- The post ends with `## Contributing` and the line "Thanks for reading." No `## Conclusion` and no references list.
- No stack of emoji shortcodes. `:point_right:` on the repository line is the only one.
- No PatternFly component names (do not mention `LoginPage`). No frontend or backend internals, including `go:embed`, `go run`, and `npm run dev`.

## Scope Decisions

1. **Shape:** Announcement, then three recipes. A reader can stop after Features.
2. **Opening:** Direct. Hand-writing `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding` YAML is the problem. No personal hook.
3. **Contribute:** Apache License 2.0, clone the repository, issues and pull requests are welcome. No contribution process beyond that. The app repository has no `CONTRIBUTING` file, and this post does not add one.
4. **Build, run, and deploy** are all in the post. Local development and the test targets are not.
5. **Cover image:** Not created by this work. `featuredImage` points at `/img/rbac_generator_cover.png`. The post stays `draft: true` until that file exists and the author flips the draft.
6. **Screenshots:** Copy the three README PNGs into the blog. They appear in the Features section.

## Front Matter

```yaml
---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Introducing Project RBAC-Generator - A UI for Kubernetes RBAC"
description: "RBAC-Generator is a small PatternFly 6 UI for building Kubernetes and OpenShift Role, ClusterRole, RoleBinding, and ClusterRoleBinding resources without hand-writing YAML. This post covers the features, how to build the container image, how to run the published image, and how to deploy it."
date: 2026-09-30T21:00:00+02:00
draft: true
featuredImage: /img/rbac_generator_cover.png
categories: ["Platforms", "Open-Source"]
tags:
- Kubernetes
- OpenShift
- RBAC
- PatternFly
---
```

Create the file with:

```bash
hugo new content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md
```

Then replace the archetype front matter and body with the content specified below.

## Outline

1. `## Roles and Bindings, Without the YAML` — two short paragraphs. The four RBAC kinds are tedious to hand-write. RBAC-Generator is a small tool that builds them through a guided UI. Then this admonition, then the repository line:

   ```
   {{< admonition info "The core idea" true >}}
   **RBAC-Generator** builds Kubernetes and OpenShift `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding` resources through a guided PatternFly 6 UI, instead of hand-written YAML.
   {{< /admonition >}}
   ```

2. `## A PatternFly 6 UI` — the UI is PatternFly 6. It targets Kubernetes and OpenShift. A pasted or uploaded kubeconfig is optional, held only in memory for the session, and is what enables live discovery, server-side dry-run, and apply. Nothing is applied until the user reviews, dry-runs, and confirms.

3. `## Features` — a short list, not an implementation explanation:
   - Guided rule builder with cascading, searchable `apiGroups` → `resources` → `subresources` → `verbs`, backed by live API discovery (Custom Resources called out separately from built-ins) or a built-in static catalog when offline.
   - Always-on split pane: edit the form or the YAML and the other side updates, with inline errors for invalid YAML.
   - Persona templates that pre-fill a `ClusterRole` or a namespaced `Role`: Cluster-Admin, Cluster-Viewer, VirtualMachine-Admin, VirtualMachine-Viewer, Platform-Operator, Network-Engineer.
   - When connected: live API discovery, `ServiceAccount` lookup, server-side dry-run, and direct apply.
   - Read-only browse of existing `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding` resources, with one-click copy of the YAML. Browse does not edit or delete.
   - Light and dark mode, including the YAML editor.
   - A single shared login, username plus a bcrypt password hash, set with environment variables.
   - One container image, built entirely from Red Hat UBI9 images.

   Then the three figures, in this order:

   | Shortcode `src` and `src-s` | Caption |
   | --- | --- |
   | `/img/posts/202609_rbacgenerator/rbac-generator1.png` | Figure I: Login page |
   | `/img/posts/202609_rbacgenerator/rbac-generator2.png` | Figure II: Create page |
   | `/img/posts/202609_rbacgenerator/rbac-generator3.png` | Figure III: Templates |

4. `## Building the Image` — one sentence that `make image` builds `rbac-generator:v1.0` for `linux/amd64` and `linux/arm64`. The table:

   | Stage | Image | What it does |
   | --- | --- | --- |
   | UI build | `registry.access.redhat.com/ubi9/nodejs-22` | Builds the frontend |
   | Binary build | `registry.access.redhat.com/ubi9/go-toolset:1.25` | Builds the Go binary and embeds the UI |
   | Runtime | `registry.access.redhat.com/ubi9/ubi-micro` | Final image: the static binary only |

   Then one short paragraph on multi-arch: `podman build` on an Apple Silicon Mac would otherwise produce only arm64, and an amd64 cluster fails at runtime with `exec format error`. `make image` builds the local manifest `rbac-generator:v1.0`. The image people pull is `quay.io/rguske/rbac-generator:v1.0`. Both tags are the release version. `latest` is never used. Command block:

   ```shell
   make image
   ```

   Pre-built images are already at [quay.io/rguske/rbac-generator](https://quay.io/repository/rguske/rbac-generator). Building is for a customized image. Do not document `make push`, version-bump files, or the enterprise registry mirror.

5. `## Running the Published Image` — `make hash-password` is a Makefile target that runs `backend/cmd/hashpw`, so it needs a clone of the repository, not only the container image. The app has one username and one bcrypt hash (`APP_USERNAME`, `APP_PASSWORD_HASH`). Command block:

   ```shell
   git clone https://github.com/rguske/rbac-generator.git
   cd rbac-generator

   podman run --rm -p 8080:8080 \
     -e APP_USERNAME=admin \
     -e APP_PASSWORD_HASH="$(make hash-password PASSWORD=yourpassword)" \
     quay.io/rguske/rbac-generator:v1.0
   ```

   Then open `http://localhost:8080` and log in with `admin` / `yourpassword`. Do not reproduce the README's two ways of exporting the hash.

6. `## Deploying to OpenShift or Kubernetes` — manifests live in `deploy/kustomize/base/` (Deployment, Service, Route) and already reference `quay.io/rguske/rbac-generator:v1.0`. Steps:
   1. Copy `deploy/kustomize/base/secret.example.yaml` to `deploy/kustomize/base/secret.yaml` and set `APP_PASSWORD_HASH` from `make hash-password`.
   2. `kubectl apply -f deploy/kustomize/base/secret.yaml`
   3. `kubectl apply -k deploy/kustomize/base`

   One sentence: on vanilla Kubernetes, remove `route.yaml` from `kustomization.yaml` and add an Ingress. Do not paste Secret manifest contents.

7. `## Contributing` — the project is Apache License 2.0. Clone [rguske/rbac-generator](https://github.com/rguske/rbac-generator) and open an issue or a pull request. Then a blank line and `Thanks for reading.`

## File and Asset Changes

- Create `content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md` via `hugo new`, then replace the archetype body.
- Copy the three PNGs from `projects/rbac-generator/static/` to `static/img/posts/202609_rbacgenerator/` in the blog repo, keeping the filenames.
- Do not create `static/img/rbac_generator_cover.png`.

## Out of Scope

- Frontend and backend architecture, local `go run` / `npm run dev`, and test commands.
- The README's authentication essay, security-notes section, and design-history links.
- `make push`, release-version bump instructions, and the `registry.redhat.io` mirror.
- Adding a `CONTRIBUTING` file to the application repository.
- Generating or drawing the cover image.
- Setting `draft: false`.
