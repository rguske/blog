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

## Recap on vSphere with Tanzu Supervisor Services Part I

In [Part I](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-services-part-i-introduction-and-how-to/) of my blog series on the *Supervisor Services in vSphere with Tanzu (TKGS)*, I introdcued the overall concept, the idea, the requirements as well as how to register and install a new Supervisor Service in vSphere. [HERE](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-services-part-i-introduction-and-how-to/#supervisor-services)

Furthermore, I covered the feature vSphere Pods and how they come to beneficial use for the Supervisor Services. [HERE](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-services-part-i-introduction-and-how-to/#vsphere-pods)

In this second part, I'm going to demonstrate how the *Kubernetes Ingress Controller Service (Contour)* will be used for serving a vSphere Pods based web-shop application with Ingress functionalities. Also, I'm going over the NSX-site of the house in terms of Networking, Distributed Firewall as well as troubleshooting when using vSphere Pods based apps in TKGS.

## Kubernetes Ingress Controller Service Installed

The *Kubernetes Ingress Controller Service Contour* is already installed in my vSphere with Tanzu w/NSX environment. If you haven't installed a Supervisor Service from the associated Catalog before, and you also skipped reading Part I of this series, I'd recommend making yourself familiar with the installation first.

Start reading [HERE](https://rguske.github.io/post/vsphere-with-tanzu-supervisor-services-part-i-introduction-and-how-to/#add-new-service---contour).



Information such as the requested and by NSX Load Balancer assigned L4 Load Balancer IP address for the Envoy Service (*Figure I*).

*vSphere:*

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0a.png" caption="Figure I: Assigned External IP for Envoy" src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0a.png" >}}

*Kubernetes:*

```shell
kubectl -n svc-contour-domain-c8 get svc

NAME      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
contour   ClusterIP      10.96.0.164   <none>        8001/TCP                     58d
envoy     LoadBalancer   10.96.2.14    10.15.8.9     80:30077/TCP,443:32173/TCP   58d
```

This IP address will be used for creating your application DNS A-Records like e.g. shown in *Table I*.

| Name | Data |
| :--: | :--: |
| app1.mydomain.com | 10.15.8.9 |
| app2.mydomain.com | 10.15.8.9 |
| app3.mydomain.com | 10.15.8.9 |

<center><i> Table I: Application DNS A-Records </i></center>

I'll go over this more specific topic further down in this post.


{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0b.png" caption="Figure II: vSphere Namespace Compute Section Details" src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0b.png" >}}

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0c.png" caption="Figure III: vSphere Pod YAML Data" src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0c.png" >}}

## Example Web-Shop Application

To begin with, it's necessary to have an application of your choice deployed which works with an Ingress configuration. Mostly, these applications provide an user interface via a web browser and which often use Ingress to route traffic to their backend services.

A couple of popular and demo-proven example applications like e.g. Yelb or the ACME Fitness app can be found on [William Lam's](https://williamlam.com/) blog post: [Interesting Kubernetes application demos](https://williamlam.com/2020/06/interesting-kubernetes-application-demos.html).

I'm going to deploy a web-shop application which is called **Hackazon**. I like a lot using this example application, because when browsing the shop website, the names of the serving Pods are showing up on the shop-item of which they are responsible for (*Figure VI*).

{{< image src="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0.png" caption="Figure VI: Pod Name on each Shop Item" src-s="/img/posts/202303_supervisor_services_part_2/202303_supervisor_services_part_2_0.png" >}}

`upstream connect error or disconnect/reset before headers. reset reason: connection failure`

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

By browsing to the associated vSphere Namespace, a lot of valuable application deployment specific information can be picked up from here without having to interact with Kubernetes on the terminal (`kubectl get [...]`).

Further deployment specific information can be gathered at the vSphere Namespace as well. For example, if you need to know if any **Persistent Volume Claims** exists, go to the **Storage** section. If you need to know if **Network Policies** are created or which **Endpoints** exists, go to **Network**. If you are interested about **vSphere Pods**, **Deployments**, **Daemon Sets**, **Stateful Sets**, etc. go to **Compute** and select the specific category (*Figure II* & *Figure III*).


{{< mermaid >}}
graph LR;
    A( User ) -->| browsing | B(https://app1.mydomain.com)
    B -->|DNS resolution | C(NSX Load Balancer - 10.15.8.9)
    C ---|VIP assigned| E(Envoy Proxy - serviceType: LoadBalancer)
    E --- F(Contour)
    F -->|routed| G(Application Web GUI)
{{< /mermaid >}}

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

I'd like to **THANK** my very respected fellow [Andreas Marqvardsen](https://blog.andreasm.io/) who helped me getting a better understanding how networking with VMware NSX is used properly in vSphere with Tanzu (TKGS).