---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "vSphere with Tanzu Supervisor Services Part II - Ingress with Contour for vSphere Pod based Applications"
date: 2023-03-13T21:09:46+01:00
draft: true
featuredImage: /img/supervisor-services-2-cover.png
categories: ["Platforms", "Open-Source", "Tools", "Security"]
tags:
- NSX
- Contour
- VMware
- Extensibility
- CloudNative
- Kubernetes
---

In [Part I](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-services-part-i-introduction-and-how-to/) of my blog series about the *Supervisor Services in vSphere with Tanzu* I introdcued the concept of vSphere Pods and how they come to be used for these.

## Zero-Trust by Default

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: hackazon-shop
  labels:
    app: hackazon-shop
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hackazon-shop
  template:
    metadata:
      labels:
        app: hackazon-shop
    spec:
      containers:
      - name: hackazon-shop
        image: projects.registry.vmware.com/tanzu_ese_poc/hackazon:1.0
        ports:
          - containerPort: 80
            protocol: TCP
```

```yaml
kind: Service
apiVersion: v1
metadata:
  name: hackazon-l4
  labels:
    app: hackazon-shop
    svc: hackazon-l4
spec:
  ports:
    - port: 80
  selector:
    app: hackazon-shop
  type: ClusterIP
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hackazon-ingress
  labels:
    app: hackazon-shop
  annotations:
    kubernetes.io/ingress.class: contour
spec:
  rules:
  - host: hackazon.cpod-nsxv8.az-stc.cloud-garage.net
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: hackazon-l4
            port:
              number: 80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: hackazon-app-network-policy
spec:
  podSelector:
    matchLabels:
      app: hackazon-shop
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: svc-contour-domain-c8
    - podSelector:
        matchLabels:
          app: contour
    ports:
    - protocol: TCP
      port: 80
```

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0.png" caption="Figure I: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_1.png" caption="Figure II: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_1.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_2.png" caption="Figure III: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_2.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_3.png" caption="Figure IV: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_3.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_4.png" caption="Figure V: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_4.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_5.png" caption="Figure VI: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_5.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_6.png" caption="Figure VII: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_6.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_7.png" caption="Figure VIII: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_7.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_8.png" caption="Figure IX: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_8.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_9.png" caption="Figure X: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_9.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_10.png" caption="Figure XI: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_10.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_11.png" caption="Figure XII: " src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_11.png" >}}

## Credits

Image by <a href="https://pixabay.com/users/geralt-9301/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3143432">Gerd Altmann</a> from <a href="https://pixabay.com//?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3143432">Pixabay</a>