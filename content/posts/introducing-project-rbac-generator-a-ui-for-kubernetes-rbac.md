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
