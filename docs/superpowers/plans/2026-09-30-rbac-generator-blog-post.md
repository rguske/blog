# RBAC-Generator Blog Post Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a draft Hugo post that introduces RBAC-Generator: what it solves, PatternFly 6, the features, then how to build, run, and deploy it.

**Architecture:** One Markdown post plus three copied screenshots. The post is an announcement (problem, UI, features) followed by three copy-paste recipes and a short contributing close. The cover image is a path only; this plan does not create the file.

**Tech Stack:** Hugo (LoveIt theme), Markdown, existing `image` and `admonition` shortcodes.

## Global Constraints

- Title is exactly `Introducing Project RBAC-Generator - A UI for Kubernetes RBAC`. The word `Project` appears only in that title.
- The body uses the name `RBAC-Generator`.
- `draft: true`. Do not create `static/img/rbac_generator_cover.png`. `featuredImage` is `/img/rbac_generator_cover.png`.
- Date is `2026-09-30T21:00:00+02:00`.
- Categories are `Platforms` and `Open-Source`. Tags are `Kubernetes`, `OpenShift`, `RBAC`, `PatternFly`.
- One admonition: `info`, title `The core idea`, body text exactly as in Task 2.
- Repository line uses `<i class='fab fa-github fa-fw'></i> repository :point_right:` and `https://github.com/rguske/rbac-generator`. `:point_right:` is the only emoji shortcode.
- Do not mention `LoginPage`, `go:embed`, `go run`, `npm run dev`, `make push`, or `registry.redhat.io`.
- No `## Conclusion` and no `## References`.
- Do not paste Secret manifest contents. Do not add a `CONTRIBUTING` file to the application repository.
- `make image` builds the local manifest `rbac-generator:v1.0` for `linux/amd64` and `linux/arm64`. The pull tag is `quay.io/rguske/rbac-generator:v1.0`. `latest` is never used.
- Screenshot shortcode paths are `/img/posts/202609_rbacgenerator/rbac-generator1.png`, `rbac-generator2.png`, and `rbac-generator3.png`, captions `Figure I: Login page`, `Figure II: Create page`, `Figure III: Templates`.
- Spec: `docs/superpowers/specs/2026-09-30-rbac-generator-blog-post-design.md`.

## File Structure

- `static/img/posts/202609_rbacgenerator/rbac-generator1.png` — login figure. Copied, not edited.
- `static/img/posts/202609_rbacgenerator/rbac-generator2.png` — create figure. Copied, not edited.
- `static/img/posts/202609_rbacgenerator/rbac-generator3.png` — templates figure. Copied, not edited.
- `content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md` — the only post. Front matter and all seven sections live in this one file.

Blog root for every command: `/Users/rguske/Documents/blog`.

Screenshot source: `/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/dev/projects/rbac-generator/static/`.

---

### Task 1: Copy the three screenshots

**Files:**
- Create: `static/img/posts/202609_rbacgenerator/rbac-generator1.png`
- Create: `static/img/posts/202609_rbacgenerator/rbac-generator2.png`
- Create: `static/img/posts/202609_rbacgenerator/rbac-generator3.png`

**Interfaces:**
- Consumes: the three PNGs in the RBAC-Generator repo `static/` directory.
- Produces: the three blog paths listed above. Task 2's `image` shortcodes point at these paths and no others.

- [ ] **Step 1: Copy the files**

Run from any directory:

```bash
mkdir -p "/Users/rguske/Documents/blog/static/img/posts/202609_rbacgenerator"
cp \
  "/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/dev/projects/rbac-generator/static/rbac-generator1.png" \
  "/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/dev/projects/rbac-generator/static/rbac-generator2.png" \
  "/Users/rguske/Library/CloudStorage/GoogleDrive-robert.guske@googlemail.com/My Drive/dev/projects/rbac-generator/static/rbac-generator3.png" \
  "/Users/rguske/Documents/blog/static/img/posts/202609_rbacgenerator/"
```

- [ ] **Step 2: Verify the copies**

```bash
file "/Users/rguske/Documents/blog/static/img/posts/202609_rbacgenerator/"*.png
```

Expected: three lines, each ending in `PNG image data`. Filenames `rbac-generator1.png`, `rbac-generator2.png`, `rbac-generator3.png`.

- [ ] **Step 3: Commit**

```bash
git -C "/Users/rguske/Documents/blog" add static/img/posts/202609_rbacgenerator
git -C "/Users/rguske/Documents/blog" commit -m "$(cat <<'EOF'
Add RBAC-Generator screenshots for the introduction post.

EOF
)"
```

Expected: one commit containing only those three PNGs.

### Task 2: Write the post

**Files:**
- Create: `content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md`

**Interfaces:**
- Consumes: the three image paths from Task 1.
- Produces: the post file at the path above, `draft: true`, seven `##` headings in the order below.

- [ ] **Step 1: Scaffold with Hugo**

```bash
cd "/Users/rguske/Documents/blog"
hugo new content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md
```

Expected: exit 0, and the file exists. If Hugo reports that the file already exists, continue to Step 2 and overwrite it.

- [ ] **Step 2: Replace the file with this exact content**

````markdown
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

## Roles and Bindings, Without the YAML

Hand-writing a `Role`, `ClusterRole`, `RoleBinding`, or `ClusterRoleBinding` gets tedious fast. The manifest is short, but `apiGroups`, resources, and verbs are easy to mistype, and you usually find that out after `kubectl apply`.

RBAC-Generator is a small tool that builds those four kinds through a guided UI. You review the YAML, dry-run it, and apply only when it looks right.

{{< admonition info "The core idea" true >}}
**RBAC-Generator** builds Kubernetes and OpenShift `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding` resources through a guided PatternFly 6 UI, instead of hand-written YAML.
{{< /admonition >}}

<i class='fab fa-github fa-fw'></i> repository :point_right: [rguske/rbac-generator](https://github.com/rguske/rbac-generator)

## A PatternFly 6 UI

The UI is [PatternFly 6](https://www.patternfly.org/). The same app targets Kubernetes and OpenShift.

Connecting a cluster is optional. Paste or upload a kubeconfig and RBAC-Generator holds that text in memory for the session. That connection is what enables live API discovery, a server-side dry-run, and apply. Nothing is applied until you review it, dry-run it, and confirm.

## Features

- A guided rule builder with cascading, searchable dropdowns: `apiGroups` → `resources` → `subresources` → `verbs`. Live API discovery backs those lists when you are connected, and Custom Resources are called out separately from built-ins. Offline, the same builder uses a built-in static catalog.
- An always-on split pane. Edit the form or the YAML and the other side updates, with inline errors when the YAML is invalid.
- Persona templates that pre-fill either a `ClusterRole` or a namespaced `Role`: Cluster-Admin, Cluster-Viewer, VirtualMachine-Admin, VirtualMachine-Viewer, Platform-Operator, Network-Engineer, and Storage-Admin.
- With a cluster connected: live API discovery, `ServiceAccount` lookup, server-side dry-run, and direct apply.
- Read-only browse of existing `Role`, `ClusterRole`, `RoleBinding`, and `ClusterRoleBinding` resources, with one-click copy of the YAML. Browse does not edit or delete.
- Light and dark mode, and the YAML editor follows it.
- A single shared login. One username and a bcrypt hash of the password, both set with environment variables.
- One container image, built entirely from Red Hat UBI9 images.

{{< image src="/img/posts/202609_rbacgenerator/rbac-generator1.png" caption="Figure I: Login page" src-s="/img/posts/202609_rbacgenerator/rbac-generator1.png" >}}

{{< image src="/img/posts/202609_rbacgenerator/rbac-generator2.png" caption="Figure II: Create page" src-s="/img/posts/202609_rbacgenerator/rbac-generator2.png" >}}

{{< image src="/img/posts/202609_rbacgenerator/rbac-generator3.png" caption="Figure III: Templates" src-s="/img/posts/202609_rbacgenerator/rbac-generator3.png" >}}

## Building the Image

`make image` builds a local manifest named `rbac-generator:v1.0` for `linux/amd64` and `linux/arm64`. Every stage is a Red Hat UBI9 image.

| Stage | Image | What it does |
| --- | --- | --- |
| UI build | `registry.access.redhat.com/ubi9/nodejs-22` | Builds the frontend |
| Binary build | `registry.access.redhat.com/ubi9/go-toolset:1.25` | Builds the Go binary and embeds the UI |
| Runtime | `registry.access.redhat.com/ubi9/ubi-micro` | Final image: the static binary only |

Multi-arch is deliberate here. A plain `podman build` on an Apple Silicon Mac produces an arm64 image only, and an amd64 node then fails at runtime with `exec format error`. `make image` builds the local manifest `rbac-generator:v1.0`. The image you pull is `quay.io/rguske/rbac-generator:v1.0`. Both tags are the release version. `latest` is never used.

```shell
make image
```

Pre-built images for the release are already published at [quay.io/rguske/rbac-generator](https://quay.io/repository/rguske/rbac-generator). Build your own when you are customizing the app.

## Running the Published Image

`make hash-password` is a Makefile target that runs `backend/cmd/hashpw`, so you need a clone of the repository, not only the container image. The app has one username, `APP_USERNAME`, and one bcrypt hash of the password, `APP_PASSWORD_HASH`.

```shell
git clone https://github.com/rguske/rbac-generator.git
cd rbac-generator

podman run --rm -p 8080:8080 \
  -e APP_USERNAME=admin \
  -e APP_PASSWORD_HASH="$(make hash-password PASSWORD=yourpassword)" \
  quay.io/rguske/rbac-generator:v1.0
```

Open http://localhost:8080 and log in with `admin` / `yourpassword`.

## Deploying to OpenShift or Kubernetes

The manifests under `deploy/kustomize/base/` ship a Deployment, a Service, and a Route. They already reference `quay.io/rguske/rbac-generator:v1.0`.

1. Copy `deploy/kustomize/base/secret.example.yaml` to `deploy/kustomize/base/secret.yaml` and set `APP_PASSWORD_HASH` to the output of `make hash-password`.
2. Apply the secret: `kubectl apply -f deploy/kustomize/base/secret.yaml`
3. Apply the base: `kubectl apply -k deploy/kustomize/base`

On vanilla Kubernetes, remove `route.yaml` from `kustomization.yaml` and add an Ingress.

## Contributing

RBAC-Generator is licensed under the [Apache License, Version 2.0](https://github.com/rguske/rbac-generator/blob/main/LICENSE). Clone [rguske/rbac-generator](https://github.com/rguske/rbac-generator) and open an issue or a pull request.

Thanks for reading.
````

- [ ] **Step 3: Check required lines and forbidden lines**

```bash
POST="/Users/rguske/Documents/blog/content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md"
rg -n "The core idea|Figure I: Login page|Figure II: Create page|Figure III: Templates|make image|quay.io/rguske/rbac-generator:v1.0|Thanks for reading.|draft: true" "$POST"
echo "--- forbidden (expect no output) ---"
rg -n "LoginPage|go:embed|go run|npm run dev|make push|registry.redhat.io|^## Conclusion|^## References" "$POST" || true
echo "--- Project (expect the title line only) ---"
rg -n "Project" "$POST"
```

Expected: the first `rg` prints matches for every required phrase. The forbidden search prints nothing. `Project` matches only the `title:` line.

- [ ] **Step 4: Commit**

```bash
git -C "/Users/rguske/Documents/blog" add content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md
git -C "/Users/rguske/Documents/blog" commit -m "$(cat <<'EOF'
Add a draft introduction to RBAC-Generator.

EOF
)"
```

Expected: one commit containing only that Markdown file.

### Task 3: Render the draft

**Files:**
- Test: `content/posts/introducing-project-rbac-generator-a-ui-for-kubernetes-rbac.md`
- Test: `static/img/posts/202609_rbacgenerator/rbac-generator1.png`
- Test: `static/img/posts/202609_rbacgenerator/rbac-generator2.png`
- Test: `static/img/posts/202609_rbacgenerator/rbac-generator3.png`

**Interfaces:**
- Consumes: the post from Task 2 and the images from Task 1.
- Produces: a successful Hugo render. No file changes.

- [ ] **Step 1: Build drafts in memory**

```bash
cd "/Users/rguske/Documents/blog"
hugo --buildDrafts --renderToMemory
```

Expected: exit 0. Hugo prints a page count and does not report a shortcode or Markdown error for `introducing-project-rbac-generator-a-ui-for-kubernetes-rbac`.

- [ ] **Step 2: Confirm the draft was part of the build**

```bash
cd "/Users/rguske/Documents/blog"
hugo --buildDrafts --renderToMemory 2>&1 | rg -n "introducing-project-rbac-generator|ERROR|error" || true
```

Expected: no `ERROR` lines. If Hugo's summary does not name the post, confirm the file is still `draft: true` and that Step 1 exited 0. That is enough: LoveIt does not have to print the slug.

- [ ] **Step 3: Do not commit**

This task changes no files. Do not create an empty commit. Do not set `draft: false`. Do not add `static/img/rbac_generator_cover.png`.
