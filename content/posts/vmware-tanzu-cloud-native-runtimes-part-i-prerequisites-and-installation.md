---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "VMware Tanzu Cloud Native Runtimes Part I - Prerequisites and Installation"
date: 2023-05-16T11:38:31+02:00
draft: true
featuredImage: /img/pan-yunbo-EgL0EtzL0Wc-unsplash.jpg
categories: ["Platforms", "Serverless"]
tags:
- Knative
- VMware
- Tanzu
- CloudNative
- Kubernetes
- Harbor
- Carvel
---

## Introduction


## Prerequisites

Access to VMware Tanzu Network:

A Tanzu Network account to download Tanzu Application Platform packages.
Network access to https://registry.tanzu.vmware.com.
Cluster-specific registry:

A container image registry, such as Harbor or Docker Hub for application images, base images, and runtime dependencies. When available, VMware recommends using a paid registry account to avoid potential rate-limiting associated with some free registry offerings.

Recommended storage space for container image registry:

1 GB of available storage if installing Tanzu Build Service with the lite set of dependencies.
10 GB of available storage if installing Tanzu Build Service with the full set of dependencies, which are suitable for offline environments.

Allocate a wildcard subdomain for your developer’s applications. This is specified in the shared.ingress_domain key of the tap-values.yaml configuration file that you input with the installation. This wildcard must be pointed at the external IP address of the tanzu-system-ingress’s envoy service.

Installation requires Kubernetes cluster v1.24, v1.25 or v1.26

Resource requirements [here](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.5/tap/prerequisites.html#resource-requirements-4)

Tanzu CLI - [here](https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.5/tap/install-tanzu-cli.html#install-or-update-the-tanzu-cli-and-plugins-3)

```shell
k get nodes
NAME                                              STATUS   ROLES           AGE    VERSION
rguske-tkc-eventing-dbrk9-d6x9m                   Ready    control-plane   107m   v1.24.9+vmware.1
rguske-tkc-eventing-np-1-tkjn6-76d89d49cc-6429k   Ready    <none>          102m   v1.24.9+vmware.1
rguske-tkc-eventing-np-1-tkjn6-76d89d49cc-dvvp2   Ready    <none>          99m    v1.24.9+vmware.1
rguske-tkc-eventing-np-1-tkjn6-76d89d49cc-k5v77   Ready    <none>          101m   v1.24.9+vmware.1
```

```shell
k -n tkg-system get deploy
NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
kapp-controller                         1/1     1            1           108m
tanzu-capabilities-controller-manager   1/1     1            1           106m
```

```shell
k -n secretgen-controller get deploy
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
secretgen-controller   1/1     1            1           107m
```

### Tanzu Packages - Image Bundles Relocation

`tanzu isolated-cluster download-bundle`

```shell
tanzu isolated-cluster download-bundle \
--source-repo projects.registry.vmware.com/tkg \
--tkg-version v2.1.1
```

```shell
tanzu isolated-cluster upload-bundle \
--source-directory /home/vmware/stuff/packages \
--destination-repo harbor01.cpod-nsxv8.az-stc.cloud-garage.net/packages \
--ca-certificate /home/vmware/stuff/harbor01/ca.crt
```

I skipped downloading the full bundle of packages since it requires a decent amount of disk space...

I will configure the online repository. Go on with TAP and Rabbit.

#### TAP - Image Bundle Relocation

[Sharing Container Images using the Docker CLI or Carvel's imgpkg](https://rguske.github.io/post/sharing-container-images-using-the-docker-cli-or-carvels-imgpkg/)

```shell
export IMGPKG_REGISTRY_HOSTNAME_0='registry.tanzu.vmware.com' \
export IMGPKG_REGISTRY_USERNAME_0='rguske@vmware.com' \
export IMGPKG_REGISTRY_PASSWORD_0='**********' \
export IMGPKG_REGISTRY_HOSTNAME_1='harbor01.cpod-nsxv8.az-stc.cloud-garage.net' \
export IMGPKG_REGISTRY_USERNAME_1='admin' \
export IMGPKG_REGISTRY_PASSWORD_1='VMware1!' \
export TAP_VERSION='1.5.0' \
export REGISTRY_CA_PATH='/home/vmware/stuff/harbor01/ca.crt'
```

{{< mermaid >}}
graph LR;
    A( Operator ) -->| imgpkg copy -b VMware Repo-URL --to-tar bundle.tar | B( Locally )
    B --> A
    A -->| imgpkg copy --tar bundle.tar --to-repo Harbor Repo-URL | C( Registry )
{{< /mermaid >}}

```shell
imgpkg copy \
  -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:$TAP_VERSION \
  --to-tar tap-packages-$TAP_VERSION.tar \
  --include-non-distributable-layers
```

```shell
imgpkg copy \
  --tar tap-packages-$TAP_VERSION.tar \
  --to-repo $IMGPKG_REGISTRY_HOSTNAME/tap \
  --include-non-distributable-layers \
  --registry-ca-cert-path $REGISTRY_CA_PATH
```

```shell
ls -rtl

total 8,5G
drwxrwxr-x  2 vmware vmware 4,0K avril 24 12:21 .
drwxrwxr-x 16 vmware vmware  16K avril 24 12:16 ..
-rw-rw-r--  1 vmware vmware 8,5G avril 24 12:32 tap-packages-1.5.0.tar
```

Make sure that you have enough space otherwise (Harbor specific) you will see:


Watch progress rolling and succeeding:

```shell
imgpkg copy --tar tap-packages-1.5.0.tar --to-repo harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0 --registry-ca-cert-path=/home/vmware/stuff/harbor01/ca.crt
copy | importing 249 images...

8.36 GiB / 8.37 GiB [-------------------------------------------------------------------------------------------------------->] 99.96% 22.22 MiB p/s
copy |
copy | done uploading images
copy | Warning: Skipped the followings layer(s) due to it being non-distributable. If you would like to include non-distributable layers, use the --include-non-distributable-layers flag
copy |  - Image: harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0@sha256:611b2607a583117a3c01826c04cce331de37b3a9a1e5e77f7cc908aa8f0c5e28
copy |    Layers:
copy |      - sha256:5c9d6483dab113d2d9b50fdf3156622aa2ec3d6faaed5664d15a5568032d1714
copy |  - Image: harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0@sha256:7196d4d2f445dcc656edf899dbea2d91c911bd08d1081ead7e86b3cf9afdf641
copy |    Layers:
copy |      - sha256:5c9d6483dab113d2d9b50fdf3156622aa2ec3d6faaed5664d15a5568032d1714
copy | Tagging images

Succeeded
```


Alternatively...

{{< mermaid >}}
graph LR;
    A( Operator ) -->| imgpkg copy -b VMware Repo-URL --to-repo Harbor Repo-URL | B( Registry )
{{< /mermaid >}}


```shell
imgpkg copy \
  -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:$TAP_VERSION \
  --to-repo harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-$TAP_VERSION \
  --registry-ca-cert-path=$REGISTRY_CA_PATH
```

```shell
[...]

copy | will export registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:fc3fb1e5890711a7d39ae077bfca73e36ca5f6f6c827484d87076dd14741ff26
copy | exported 249 images
copy | importing 249 images...

8.37 GiB / 8.37 GiB [--------------------------------------------------------------------------------------------------------] 100.00% 42.76 MiB p/s
copy |
copy | done uploading images
copy | Warning: Skipped the followings layer(s) due to it being non-distributable. If you would like to include non-distributable layers, use the --include-non-distributable-layers flag
copy |  - Image: harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0@sha256:611b2607a583117a3c01826c04cce331de37b3a9a1e5e77f7cc908aa8f0c5e28
copy |    Layers:
copy |      - sha256:5c9d6483dab113d2d9b50fdf3156622aa2ec3d6faaed5664d15a5568032d1714
copy |  - Image: harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0@sha256:7196d4d2f445dcc656edf899dbea2d91c911bd08d1081ead7e86b3cf9afdf641
copy |    Layers:
copy |      - sha256:5c9d6483dab113d2d9b50fdf3156622aa2ec3d6faaed5664d15a5568032d1714
copy | Tagging images

Succeeded
```


### RabbitMQ Image Bundle Relocation


```shell
export RABBITMQ_VERSION='1.4.0' \
export REGISTRY_CA_PATH='/home/vmware/stuff/harbor01/ca.crt'
```

```shell
docker login registry.tanzu.vmware.com -u rguske@vmware.com
Password: *******
WARNING! Your password will be stored unencrypted in /home/vmware/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

```shell
imgpkg copy \
  -b registry.tanzu.vmware.com/p-rabbitmq-for-kubernetes/tanzu-rabbitmq-package-repo:$RABBITMQ_VERSION \
  --to-repo harbor01.cpod-nsxv8.az-stc.cloud-garage.net/rabbitmq/rabbitmq-package-$RABBITMQ_VERSION \
  --registry-ca-cert-path=$REGISTRY_CA_PATH
```

```shell
[...]
copy | exported 40 images
copy | importing 40 images...

1.47 GiB / 1.47 GiB [------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------>] 99.93% 4.29 MiB p/s
copy |
copy | done uploading images
copy | Tagging images

Succeeded
```

### Adding TAP Package Repositories

```shell
k create ns tap-packages

namespace/tap-packages created
```

```shell
tanzu package repository add tap-1.5.0 --url harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0 -n tap-packages
ℹ   Adding package repository 'tap-1.5.0'
ℹ   Validating provided settings for the package repository
ℹ   Creating package repository resource
ℹ   Waiting for 'PackageRepository' reconciliation for 'tap-1.5.0'
\ 'PackageRepository' resource install status: Reconciling!

Please consider using 'tanzu package repository update' to update the package repository with correct settings

ℹ   'PackageRepository' resource install status: Reconciling


Error: resource reconciliation failed: vendir: Error: Syncing directory '0':
  Syncing directory '.' with imgpkgBundle contents:
    Fetching image tags:
      Imgpkg: exit status 1 (stderr: imgpkg: Error: Error while preparing a transport to talk with the registry:
  Unable to create round tripper:
    Get "https://harbor01.cpod-nsxv8.az-stc.cloud-garage.net/v2/": x509: certificate signed by unknown authority
)
. Reconcile failed: Fetching resources: Error (see .status.usefulErrorMessage for details)
```

{{< admonition bug "x509 Certificate Error" true >}}
Get "https://harbor01.cpod-nsxv8.az-stc.cloud-garage.net/v2/": x509: certificate signed by unknown authority
{{< /admonition >}}

Blog Post [HERE](https://rguske.github.io/post/deploy-tanzu-packages-from-a-private-registry/#optional-kapp-controller-secret-vs-configmap)

```yaml
kubectl -n tkg-system create -f - <<EOF
---
apiVersion: v1
kind: Secret
metadata:
  # Name must be `kapp-controller-config` for kapp controller to pick it up
  name: kapp-controller-config
  # Namespace must match the namespace kapp-controller is deployed to
  namespace: tkg-system
stringData:
  # A cert chain of trusted ca certs. These will be added to the system-wide
  # cert pool of trusted ca's (optional)
  caCerts: |
    -----BEGIN CERTIFICATE-----
    MIIDWzCCAkOgAwIBAgIRAMODWpLzIWy3JocU4JBtIrIwDQYJKoZIhvcNAQELBQAw

    [...]
    -----END CERTIFICATE-----
  # The url/ip of a proxy for kapp controller to use when making network
  # requests (optional)
  httpProxy: ""
  # The url/ip of a tls capable proxy for kapp controller to use when
  # making network requests (optional)
  httpsProxy: ""
  # A comma delimited list of domain names which kapp controller should
  # bypass the proxy for when making requests (optional)
  noProxy: ""
  # A comma delimited list of domain names for which kapp controller, when
  # fetching images or imgpkgBundles, will skip TLS verification. (optional)
  dangerousSkipTLSVerify: ""
EOF
```

```shell
tanzu package repository delete tap-1.5.0 -n tap-packages

[...]
```

After...

```shell
tanzu package repository add tap-1.5.0 --url harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0:v1.5.0 -n tap-packages
ℹ   Adding package repository 'tap-1.5.0'
ℹ   Validating provided settings for the package repository
ℹ   Creating package repository resource
ℹ   Waiting for 'PackageRepository' reconciliation for 'tap-1.5.0'
ℹ   'PackageRepository' resource install status: Reconciling
ℹ   'PackageRepository' resource install status: ReconcileSucceeded
ℹ   'PackageRepository' resource successfully reconciled
ℹ  Added package repository 'tap-1.5.0' in namespace 'tap-packages'

tanzu package repository list -n tap-packages

  NAME       REPOSITORY                                                          TAG       STATUS               DETAILS
  tap-1.5.0  harbor01.cpod-nsxv8.az-stc.cloud-garage.net/tap/tap-packages-1.5.0  (1.5.0)  Reconcile succeeded
```

A lot of packages...

```shell
tanzu package available list -n tap-packages

  NAME                                                 DISPLAY-NAME                                                              SHORT-DESCRIPTION                                                                 LATEST-VERSION
  cert-manager.tanzu.vmware.com                        cert-manager                                                              Certificate management                                                            2.3.0
  contour.tanzu.vmware.com                             contour                                                                   An ingress controller                                                             1.22.5+tap.1.5.0
  external-dns.tanzu.vmware.com                        external-dns                                                              This package provides DNS synchronization functionality.                          0.12.2+vmware.4-tkg.2
  fluent-bit.tanzu.vmware.com                          fluent-bit                                                                Fluent Bit is a fast Log Processor and Forwarder                                  1.9.5+vmware.1-tkg.1
  fluxcd-helm-controller.tanzu.vmware.com              Flux Helm Controller                                                      Helm controller is one of the components in FluxCD GitOps toolkit.                0.28.1+vmware.1-tkg.1
  fluxcd-kustomize-controller.tanzu.vmware.com         Flux Kustomize Controller                                                 Kustomize controller is one of the components in Fluxcd GitOps toolkit.           0.32.0+vmware.1-tkg.1
  fluxcd-source-controller.tanzu.vmware.com            Flux Source Controller                                                    The source-controller is a Kubernetes operator, specialised in artifacts          0.33.0+vmware.1-tkg.1
                                                                                                                                 acquisition from external sources such as Git, Helm repositories and S3 buckets.
  grafana.tanzu.vmware.com                             grafana                                                                   Visualization and analytics software                                              7.5.17+vmware.1-tkg.1
  harbor.tanzu.vmware.com                              harbor                                                                    OCI Registry                                                                      2.6.3+vmware.1-tkg.1
  multus-cni.tanzu.vmware.com                          multus-cni                                                                This package provides the ability for enabling attaching multiple network         3.8.0+vmware.2-tkg.2
                                                                                                                                 interfaces to pods in Kubernetes
  prometheus.tanzu.vmware.com                          prometheus                                                                A time series database for your metrics                                           2.37.0+vmware.2-tkg.1
  whereabouts.tanzu.vmware.com                         whereabouts                                                               A CNI IPAM plugin that assigns IP addresses cluster-wide                          0.5.4+vmware.1-tkg.1
  accelerator.apps.tanzu.vmware.com                    Application Accelerator for VMware Tanzu                                  Used to create new projects and configurations.                                   1.5.1
  api-portal.tanzu.vmware.com                          API portal                                                                A unified user interface for API discovery and exploration at scale.              1.3.0
  apis.apps.tanzu.vmware.com                           API Auto Registration for VMware Tanzu                                    A TAP component to automatically register API exposing workloads as API entities  0.3.0
                                                                                                                                 in TAP GUI.
  apiserver.appliveview.tanzu.vmware.com               Application Live View ApiServer for VMware Tanzu                          App for validating secure access control to application live view components      1.5.1
  app-scanning.apps.tanzu.vmware.com                   Supply Chain Security Tools - App Scanning (Alpha)                        Scan for vulnerabilities within Kubernetes native Supply Chains.                  0.0.1-alpha.4
  application-configuration-service.tanzu.vmware.com   Application Configuration Service                                         Application Configuration Service                                                 2.0.0
  backend.appliveview.tanzu.vmware.com                 Application Live View for VMware Tanzu                                    App for monitoring and troubleshooting running apps                               1.5.1
  bitnami.services.tanzu.vmware.com                    bitnami-services                                                          Helm-based services from Bitnami                                                  0.1.0
  buildservice.tanzu.vmware.com                        Tanzu Build Service                                                       Tanzu Build Service enables the building and automation of containerized          1.10.8
                                                                                                                                 software workflows securely and at scale.
  carbonblack.scanning.apps.tanzu.vmware.com           VMware Carbon Black for Supply Chain Security Tools - Scan                Default scan templates using VMware Carbon Black                                  1.2.0-beta.1
  cartographer.tanzu.vmware.com                        Cartographer                                                              Kubernetes native Supply Chain Choreographer.                                     0.7.1+tap.1
  cnrs.tanzu.vmware.com                                Cloud Native Runtimes                                                     Cloud Native Runtimes is a serverless runtime based on Knative                    2.2.0
  connector.appliveview.tanzu.vmware.com               Application Live View Connector for VMware Tanzu                          App for discovering and registering running apps                                  1.5.1
  controller.source.apps.tanzu.vmware.com              Tanzu Source Controller                                                   Tanzu Source Controller enables workload create/update from source code.          0.7.0
  conventions.appliveview.tanzu.vmware.com             Application Live View Conventions for VMware Tanzu                        Application Live View convention server                                           1.5.1
  crossplane.tanzu.vmware.com                          crossplane                                                                Crossplane is a framework for building cloud native control planes without        0.1.1
                                                                                                                                 needing to write code.
  developer-conventions.tanzu.vmware.com               Tanzu App Platform Developer Conventions                                  Developer Conventions                                                             0.10.0
  eventing.tanzu.vmware.com                            Eventing                                                                  Eventing is an event-driven architecture platform based on Knative Eventing       2.2.1
  external-secrets.apps.tanzu.vmware.com               External Secrets Operator                                                 External Secrets Operator is a Kubernetes operator that integrates external       0.6.1+tap.6
                                                                                                                                 secret management systems.
  fluxcd.source.controller.tanzu.vmware.com            Flux Source Controller                                                    The source-controller is a Kubernetes operator, specialised in artifacts          0.27.0+tap.10
                                                                                                                                 acquisition from external sources such as Git, Helm repositories and S3 buckets.
  grype.scanning.apps.tanzu.vmware.com                 Grype for Supply Chain Security Tools - Scan                              Default scan templates using Anchore Grype                                        1.5.0
  learningcenter.tanzu.vmware.com                      Learning Center for Tanzu Application Platform                            Guided technical workshops                                                        0.2.7
  metadata-store.apps.tanzu.vmware.com                 Supply Chain Security Tools - Store                                       Post SBoMs and query for image, package, and vulnerability metadata.              1.5.0
  namespace-provisioner.apps.tanzu.vmware.com          Namespace Provisioner                                                     Automatic Provisioning of Developer Namespaces.                                   0.3.1
  ootb-delivery-basic.tanzu.vmware.com                 Tanzu App Platform Out of The Box Delivery Basic                          Out of The Box Delivery Basic.                                                    0.12.5
  ootb-supply-chain-basic.tanzu.vmware.com             Tanzu App Platform Out of The Box Supply Chain Basic                      Out of The Box Supply Chain Basic.                                                0.12.5
  ootb-supply-chain-testing-scanning.tanzu.vmware.com  Tanzu App Platform Out of The Box Supply Chain with Testing and Scanning  Out of The Box Supply Chain with Testing and Scanning.                            0.12.5
  ootb-supply-chain-testing.tanzu.vmware.com           Tanzu App Platform Out of The Box Supply Chain with Testing               Out of The Box Supply Chain with Testing.                                         0.12.5
  ootb-templates.tanzu.vmware.com                      Tanzu App Platform Out of The Box Templates                               Out of The Box Templates.                                                         0.12.5
  policy.apps.tanzu.vmware.com                         Supply Chain Security Tools - Policy Controller                           Policy Controller enables defining of a policy to restrict unsigned container     1.4.0
                                                                                                                                 images.
  scanning.apps.tanzu.vmware.com                       Supply Chain Security Tools - Scan                                        Scan for vulnerabilities and enforce policies directly within Kubernetes native   1.5.2
                                                                                                                                 Supply Chains.
  service-bindings.labs.vmware.com                     Service Bindings for Kubernetes                                           Service Bindings for Kubernetes implements the Service Binding Specification.     0.9.1
  services-toolkit.tanzu.vmware.com                    Services Toolkit                                                          The Services Toolkit enables the management, lifecycle, discoverability and       0.10.1
                                                                                                                                 connectivity of Service Resources (databases, message queues, DNS records,
                                                                                                                                 etc.).
  snyk.scanning.apps.tanzu.vmware.com                  Snyk for Supply Chain Security Tools - Scan                               Default scan templates using Snyk                                                 1.0.0-beta.8
  spring-boot-conventions.tanzu.vmware.com             Tanzu Spring Boot Conventions Server                                      Default Spring Boot convention server.                                            1.5.1
  spring-cloud-gateway.tanzu.vmware.com                Spring Cloud Gateway                                                      Spring Cloud Gateway                                                              2.0.0-tap.3
  sso.apps.tanzu.vmware.com                            AppSSO                                                                    Application Single Sign-On for Tanzu                                              3.1.0
  tap-auth.tanzu.vmware.com                            Default roles for Tanzu Application Platform                              Default roles for Tanzu Application Platform                                      1.1.0
  tap-gui.tanzu.vmware.com                             Tanzu Application Platform GUI                                            web app graphical user interface for Tanzu Application Platform                   1.5.1
  tap-telemetry.tanzu.vmware.com                       Telemetry Collector for Tanzu Application Platform                        Tanzu Application Plaform Telemetry                                               0.5.0-build.3
  tap.tanzu.vmware.com                                 Tanzu Application Platform                                                Package to install a set of TAP components to get you started based on your use   1.5.0
                                                                                                                                 case.
  tekton.tanzu.vmware.com                              Tekton Pipelines                                                          Tekton Pipelines is a framework for creating CI/CD systems.                       0.41.0+tap.8
  workshops.learningcenter.tanzu.vmware.com            Workshop Building Tutorial                                                Workshop Building Tutorial                                                        0.2.6
```

Notice, that `cert-manager`, `Contour`, etc. are listed as well and which are normally included/available via the TKG-Standard repository. This isn't needed in our case and I'd more important in terms of supportability, is in fact the detail that the TAP packages are offering a specific version for some of the standard paclages like e.g. Contour:

| Tanzu Standard Packages Contour Version | TAP Packages Contour Version |
| :-: | :-: |
| 1.22.3+vmware.1-tkg.1 | 1.22.5+tap.1.5.0 |



### Adding RabbitMQ Package Repositories

```shell
k create ns rabbitmq

namespace/rabbitmq created
```

```shell
tanzu package repository add rabbitmq-1.4.0 --url harbor01.cpod-nsxv8.az-stc.cloud-garage.net/rabbitmq/rabbitmq-package-1.4.0:1.4.0 -n rabbitmq
ℹ   Adding package repository 'rabbitmq-1.4.0'
ℹ   Updating package repository 'rabbitmq-1.4.0'
ℹ   Getting package repository 'rabbitmq-1.4.0'
ℹ   Validating provided settings for the package repository
ℹ   Updating package repository resource
ℹ   Waiting for 'PackageRepository' reconciliation for 'rabbitmq-1.4.0'
ℹ   'PackageRepository' resource install status: Reconciling
ℹ   'PackageRepository' resource install status: ReconcileSucceeded


ℹ  Updated package repository 'rabbitmq-1.4.0' in namespace 'rabbitmq'
```

```shell
tanzu package available list -n rabbitmq

  NAME                                          DISPLAY-NAME               SHORT-DESCRIPTION                                                                 LATEST-VERSION
  cert-manager.tanzu.vmware.com                 cert-manager               Certificate management                                                            1.10.1+vmware.1-tkg.2
  contour.tanzu.vmware.com                      contour                    An ingress controller                                                             1.22.3+vmware.1-tkg.1
  external-dns.tanzu.vmware.com                 external-dns               This package provides DNS synchronization functionality.                          0.12.2+vmware.4-tkg.2
  fluent-bit.tanzu.vmware.com                   fluent-bit                 Fluent Bit is a fast Log Processor and Forwarder                                  1.9.5+vmware.1-tkg.1
  fluxcd-helm-controller.tanzu.vmware.com       Flux Helm Controller       Helm controller is one of the components in FluxCD GitOps toolkit.                0.28.1+vmware.1-tkg.1
  fluxcd-kustomize-controller.tanzu.vmware.com  Flux Kustomize Controller  Kustomize controller is one of the components in Fluxcd GitOps toolkit.           0.32.0+vmware.1-tkg.1
  fluxcd-source-controller.tanzu.vmware.com     Flux Source Controller     The source-controller is a Kubernetes operator, specialised in artifacts          0.33.0+vmware.1-tkg.1
                                                                           acquisition from external sources such as Git, Helm repositories and S3 buckets.
  grafana.tanzu.vmware.com                      grafana                    Visualization and analytics software                                              7.5.17+vmware.1-tkg.1
  harbor.tanzu.vmware.com                       harbor                     OCI Registry                                                                      2.6.3+vmware.1-tkg.1
  multus-cni.tanzu.vmware.com                   multus-cni                 This package provides the ability for enabling attaching multiple network         3.8.0+vmware.2-tkg.2
                                                                           interfaces to pods in Kubernetes
  prometheus.tanzu.vmware.com                   prometheus                 A time series database for your metrics                                           2.37.0+vmware.2-tkg.1
  whereabouts.tanzu.vmware.com                  whereabouts                A CNI IPAM plugin that assigns IP addresses cluster-wide                          0.5.4+vmware.1-tkg.1
  cert-manager.rabbitmq.tanzu.vmware.com        cert-manager-rabbitmq      Certificate management                                                            1.5.3+rmq
  rabbitmq.tanzu.vmware.com                     VMware Tanzu RabbitMQ      Commercial VMware Tanzu RabbitMQ                                                  1.4.0
```

## Installation


### Installing Cert-Manager


```shell
 tanzu package install cert-manager --package-name cert-manager.tanzu.vmware.com --version 2.3.0 -n tap-packages --wait

ℹ   Installing package 'cert-manager.tanzu.vmware.com'
ℹ   Getting package metadata for 'cert-manager.tanzu.vmware.com'
ℹ   Creating service account 'cert-manager-tap-packages-sa'
ℹ   Creating cluster admin role 'cert-manager-tap-packages-cluster-role'
ℹ   Creating cluster role binding 'cert-manager-tap-packages-cluster-rolebinding'
ℹ   Creating package resource
ℹ   Waiting for 'PackageInstall' reconciliation for 'cert-manager'
ℹ   'PackageInstall' resource install status: Reconciling
ℹ   'PackageInstall' resource install status: ReconcileSucceeded
ℹ
 Added installed package 'cert-manager'
```

The `kubectl` way:

```yaml
kubectl create -f - <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cert-manager-sa
  namespace: tap-packages
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: cert-manager-sa
    namespace: tap-packages
---
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageInstall
metadata:
  name: cert-manager
  namespace: tap-packages
spec:
  serviceAccountName: cert-manager-sa
  packageRef:
    refName: cert-manager.tanzu.vmware.com
    versionSelection:
      constraints: 2.3.0
  values:
  - secretRef:
      name: cert-manager-data-values
---
apiVersion: v1
kind: Secret
metadata:
  name: cert-manager-data-values
  namespace: tap-packages
stringData:
  values.yml: |
    ---
    namespace: cert-manager
EOF
```

### Installing Contour Package

contour-data-values.yaml

```yaml
---
infrastructure_provider: vsphere
namespace: tanzu-system-ingress
contour:
 configFileContents: {}
 useProxyProtocol: false
 replicas: 2
 pspNames: "vmware-system-restricted"
 logLevel: info
envoy:
 service:
   type: LoadBalancer
   annotations: {}
   nodePorts:
     http: null
     https: null
   externalTrafficPolicy: Cluster
   disableWait: false
 hostPorts:
   enable: true
   http: 80
   https: 443
 hostNetwork: false
 terminationGracePeriodSeconds: 300
 logLevel: info
 pspNames: null
certificates:
 duration: 8760h
 renewBefore: 360h
```



```shell
tanzu package install contour --package-name contour.tanzu.vmware.com --version 1.22.5+tap.1.5.0 -n tap-packages --values-file contour-data-values.yaml --wait
ℹ   Installing package 'contour.tanzu.vmware.com'
ℹ   Getting package metadata for 'contour.tanzu.vmware.com'
ℹ   Creating service account 'contour-tap-packages-sa'
ℹ   Creating cluster admin role 'contour-tap-packages-cluster-role'
ℹ   Creating cluster role binding 'contour-tap-packages-cluster-rolebinding'
ℹ   Creating secret 'contour-tap-packages-values'
ℹ   Creating package resource
ℹ   Waiting for 'PackageInstall' reconciliation for 'contour'
ℹ   'PackageInstall' resource install status: Reconciling
ℹ   'PackageInstall' resource install status: ReconcileSucceeded
ℹ
 Added installed package 'contour'
```

The `kubectl` way:

```yaml
kubectl create -f - <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: contour-sa
  namespace: tap-packages
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contour-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: contour-sa
    namespace: tap-packages
---
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageInstall
metadata:
  name: contour
  namespace: tap-packages
spec:
  serviceAccountName: contour-sa
  packageRef:
    refName: contour.tanzu.vmware.com
    versionSelection:
      constraints: 1.22.5+tap.1.5.0
  values:
  - secretRef:
      name: contour-values
---
apiVersion: v1
kind: Secret
metadata:
  name: contour-values
  namespace: tap-packages
stringData:
  values.yaml: |
    contour:
     configFileContents: {}
     useProxyProtocol: false
     replicas: 3
     pspNames: vmware-system-restricted
     logLevel: info
    envoy:
     service:
       type: LoadBalancer
       annotations: {}
       nodePorts:
         http: null
         https: null
       externalTrafficPolicy: Cluster
       disableWait: false
     hostPorts:
       enable: false
       http: 80
       https: 443
     hostNetwork: false
     terminationGracePeriodSeconds: 300
     logLevel: info
     pspNames: null
    certificates:
     duration: 8760h
     renewBefore: 360h
EOF
```


```shell
tanzu package installed list -A

  NAME                                            PACKAGE-NAME                                 PACKAGE-VERSION                STATUS               NAMESPACE
  cert-manager                                    cert-manager.tanzu.vmware.com                2.3.0                          Reconcile succeeded  tap-packages
  contour                                         contour.tanzu.vmware.com                     1.22.5+tap.1.5.0               Reconcile succeeded  tap-packages
```


{{< image src="/img/posts//" caption="Figure I: " src-s="/img/posts//" >}}




---

Blogpost cover by [unsplash](https://unsplash.com/photos/EgL0EtzL0Wc)