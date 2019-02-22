---
title: "Event-driven interactions in vSphere using Functions as a Service"
date: 2019-02-14T17:40:38+01:00
draft: true
image: /img/vspheretagfn_cover.png
tags:
- vSphere
- Kubernetes
- FaaS
- Event-driven
---

When was the last time that you were so taken from a topic or better from the combination of several, that only when you think about it your face is filled with enthusiasm?

For me personally, it's only been a few days and it was in one of these afterwork-calls with my well appreciated colleague and friend <a href="https://mgasch.com" target="_blank">Michael Gasch</a> where we talked about things we were working on. I just had published my recent post on <a href="https://rguske.github.io/post/monitoring-container-vms-with-vrealize-operations-manager/" target="_blank">*"Monitoring Container VMs with vRealize Operations Manager"*</a> were I described how I built a dashboard for & with vRealize Operations Manager to monitor the utilization and performance for two of the major objects from vSphere Integrated Containers, the Virtual Container Host (VCH) and the Container-VMs (cVM).

Michael told me that he spent some time in digging deeper into **Functions as a Service**...

<center> {{< tweet 1055452330064334848 >}} </center>

which was far away from beeing touched by myself to be honest but he got me right away from the very beginning of his telling. For those of you who want to know more about the topic but haven't put your nose in it yet, like me before :wink:, you´ll find some links at the *Resources section* at the buttom.

I´m not going to intend to explain what a Function/ FaaS is all about, instead of it Michael and I thought that this post should serve a introduction of a project Michael is working on as well as to demonstrate how "just" two components are turning your vSphere environment into a *Event-driven construct* based on your *annotations*.

Before I´m going to start with the details and what should already be up and running, I`d like to quote the following to describe what it means when it comes to the topic *"Functions vs. Containers"*.

> **Container:**

> Create a container which has all the required (Application) dependencies pre-installed, put your application code inside of it and run it everywhere the container runtime is installed.

> **Serverless:**

> The most basic premise of a serverless setup is that the whole application--all its business logic--is implemented as functions and events.
> Applications get split up into different functionalities (or services), which are in turn triggered by events. You upload your function code and attach an event source to it.

> **Source:** <a href="https://serverless.com/blog/serverless-faas-vs-containers/" target="_blank">Serverless (FaaS) vs. Containers - when to pick which?</a>

---
# Introducing `pytag-fn`

I´ve mentioned at the beginning that my last post was on the topic *"Monitoring Container VMs with vRealize Operations Manager"* and that I had to build a dashboard to do so. A part of this dashboard is a *Object List* which gives the operator the ability to select a object, the Virtual Container Host - VCH. This VCH consists of two components. One is the VCH-VM which provides your Docker-API Endpoint and the other is a vSphere Resource-Pool (logical abstaction for flexible resource management).

One challange for me, while I was building this dashboard, was to find a way on which criteria these Resource-Pools will join a *Dynamic Custom Group* in vRealize Operations Manager (to appear automatically in the dashboard/ in the object list). Long story short, I decided to use vSphere Tags as a Membership-Criteria. But assigning a vSphere Tag is still a manually task and this is the point were Michaels function ***pytagfn*** or ***gotagfn*** as well the the ***vCenter Connector*** comes into play.

---
You´ll find the Github Repositories under the following links:

<a href="https://github.com/embano1/vcenter-connector" target="_blank">Github Repo **`vCenter Connector`**</a>

<a href="https://github.com/embano1/pytagfn" target="_blank">Github Repo **`pytag-fn`**</a>

<a href="https://github.com/embano1/gotagfn" target="_blank">Github Repo **`gotag-fn`**</a>

---

The following picture will give you an idea what will happen starting from 1:

<center><a href="/img/posts/201902_vspheretagfn/CapturFiles-20190217_111639.jpg"><img src="/img/posts/201902_vspheretagfn/CapturFiles-20190217_111639.jpg" width="900"></img></a></center>

Michael and I recorded a **VIDEO** to demonstrate what will happen and how slim it is.

{{< youtube  >}}

---
# Implementation prerequisites




```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[0] <> bosh vms

Instance                                         Process State  AZ         IPs              VM CID                                   VM Type      Active
harbor-app/08b12248-b670-4269-8a33-b38982091fe9  running        az-physic  192.168.100.171  vm-1a37adae-35ec-4d41-a1f0-ce0419bdea73  medium.disk  true

1 vms

Deployment 'pivotal-container-service-b6e54c653e34254c1403'

Instance                                                        Process State  AZ         IPs            VM CID                                   VM Type  Active
pivotal-container-service/970919db-dc83-4d5d-bbce-a53a5e8ebfe0  running        az-physic  192.168.100.3  vm-7914002f-d4bc-4fda-a267-93a2a3722e15  large    true

1 vms

Deployment 'service-instance_60afafe8-3652-4d5d-a931-441b13df5e32'

Instance                                     Process State  AZ         IPs            VM CID                                   VM Type      Active
master/857c17e2-dd6a-45ce-9dc6-98636714bda8  running        az-physic  192.168.100.4  vm-e6d30c98-3db6-493b-a937-bed04199005e  medium.disk  true
worker/064a70fd-554a-484c-9490-5353cc75963b  running        az-physic  192.168.100.6  vm-e6a09c22-28c0-4660-a19d-02c3e4b25693  medium.disk  true
worker/978c6352-b3ef-4378-85bb-4c485550e440  running        az-physic  192.168.100.7  vm-7d44d561-0834-4ff5-91ae-e46f7b9184b9  medium.disk  true
worker/bb37b528-d3f2-476f-828c-afdc24cb950f  running        az-physic  192.168.100.5  vm-abe9def5-2e9f-4a37-b71c-f364cc46cdcd  medium.disk  true
```
---

```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[0] <> pks cluster pks-cluster-1

Name:                     pks-cluster-1
Plan Name:                small
UUID:                     60afafe8-3652-4d5d-a931-441b13df5e32
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance provisioning completed
Kubernetes Master Host:   pks-cluster-1.lab.jarvis.local
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  192.168.100.4
Network Profile Name:
```

---

```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[130] <> kubectl get nodes -o wide
NAME                                   STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
61c164ea-1851-436a-96dd-7e02498e5eab   Ready    <none>   13d   v1.12.4   192.168.100.6   192.168.100.6   Ubuntu 16.04.5 LTS   4.15.0-43-generic   docker://18.6.1
db9a3d25-b788-4a19-a4b1-7d80769ca938   Ready    <none>   13d   v1.12.4   192.168.100.7   192.168.100.7   Ubuntu 16.04.5 LTS   4.15.0-43-generic   docker://18.6.1
f0876e76-f7ad-4fb9-9b9f-c4519cbbea8f   Ready    <none>   21d   v1.12.4   192.168.100.5   192.168.100.5   Ubuntu 16.04.5 LTS   4.15.0-43-generic   docker://18.6.1
```

---

## Install OpenFaaS on Kubernetes
https://docs.openfaas.com/deployment/kubernetes/
https://github.com/openfaas/faas-netes/tree/master/chart/openfaas

<a href="https://github.com/openfaas/faas-netes" target="_blank">faas-netes Github Repo</a>

```
helm upgrade openfaas --install openfaas/openfaas \
--namespace openfaas  \
--set functionNamespace=openfaas-fn \
--set serviceType=NodePort \
--set basic_auth=true \
--set operator.create=true
```

```
faas-cli deploy
```

File: connector-dep.yml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: vcenter
    component: vcenter-connector
  name: vcenter-connector
spec:
  replicas: 1
    template:
      metadata:
        labels:
          app: vcenter
          component: vcenter-connector
      spec:
        containers:
        - name: vcenter
          image: embano1/faas-vcconnector:0.2
          command: ["./connector"]
          args: ["-vcenter", "https://vCenterIP", "-vc-user", "user@vsphere.local", "-vc-pass", "pw", "-insecure", "-gateway", "http://gateway.openfaas:8080"]
#  env:
#    - name: basic_auth
#      value: "true"
#    - name: secret_mount_path
#      value: "/var/secrets/"
#  volumeMounts:
#      - name: gateway-basic-auth
#        readOnly: true
#        mountPath: "/var/secrets/"
# if you do not have authentication enabled for OpenFaaS comment this out
# volumes:
# - name: gateway-basic-auth
#  secret:
#    secretName: gateway-basic-auth
```

```
kubectl -n openfaas create -f connector-dep.yml
```

```
kubectl -n openfaas logs vcenter-connector-6b79bf4696-bkp4z -f
```

```
kubectl get $(kubectl create -f /Users/rguske/_DEV/pks/vcenter-connector/yaml/kubernetes -o name | grep service)
```
---

VIC (Project - VIC PoC)
```
docker run --name embano1-govc -it vic01.lab.jarvis.local:443/vic-poc/embano1-govc:0ee42d3 sh
```
I don´t made use of the `--rm` option to alwyas jump back to the cVM if necessaray by using `docker exec`:
```
docker exec -it embano1-govc sh
```

```
┌─[rguske@rguske-a01] - [~/_DEV/vic/vic150]
└─[0] <> docker ps -a
CONTAINER ID        IMAGE                                                         COMMAND                  CREATED             STATUS              PORTS               NAMES
cf84d088c541        vic01.lab.jarvis.local:443/vic-poc/powerclicore:ubuntu16.04   "/usr/bin/pwsh"          10 days ago         Up 10 days                              powercli_demo
e8983201494c        vic01.lab.jarvis.local:443/vic-poc/embano1-govc:0ee42d3       "sh"                     2 weeks ago         Up 2 weeks                              embano1-govc
```

Export variables
```
export GOVC_INSECURE=true
export GOVC_URL='https://user@vsphere.local:userpw!@vcenterurl'
```

Create vSphere Tag categorie
```
./govc tags.category.create vchcat
```

Create vSphere Tag in Categorie
```
./govc tags.create -c vchcat vchtag
```

List all tags
```
./govc tags.ls
```

**Show URN of a specific Tag**
```
./govc tags.info
```

```
┌─[rguske@rguske-a01] - [~/_DEV/vic/vic150]
└─[0] <> docker exec -it embano1-govc sh
/ # export GOVC_INSECURE=true
/ # export GOVC_URL='https://admin@jarvis.local:VMware1234!@192.168.178.72'
/ # ./govc tags.ls
Virtual Container Host
demotag1
```

```
/ # ./govc tags.info
Name:           Virtual Container Host
  ID:           urn:vmomi:InventoryServiceTag:a1bf0cae-4753-40c7-841a-11695c99c9ed:GLOBAL
  Description:
  Category:     Virtual Container Host
  UsedBy: []
Name:           demotag1
  ID:           urn:vmomi:InventoryServiceTag:019c0a9e-0672-48f5-ac2a-e394669e2916:GLOBAL
  Description:
  Category:     democat1
  UsedBy: []
```

---

```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[0] <> kubectl get namespaces -o wide
NAME          STATUS   AGE
default       Active   21d
demo          Active   19d
kube-public   Active   21d
kube-system   Active   21d
openfaas      Active   19d
openfaas-fn   Active   19d
pks-system    Active   21d
```

```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[0] <> kubectl get pods -n openfaas
NAME                                READY   STATUS    RESTARTS   AGE
alertmanager-ccd8559-4jqkc          1/1     Running   0          13d
gateway-5d67c8847f-tjxm5            2/2     Running   7          19d
nats-54548d79dd-5v72w               1/1     Running   0          13d
prometheus-6c7d64d46b-4l7b6         1/1     Running   2          19d
queue-worker-765b6b774f-n5k44       1/1     Running   1          13d
vcenter-connector-7f57fbdcc-p7jrk   1/1     Running   0          12d
```

{{< highlight yaml >}}
provider:
  name: faas
  gateway:                          #FaaS Gateway (http://ip:port)
functions:
  pytag-fn:
    lang: python3
    handler: ./pytag-fn
    image: embano1/pytag-fn:0.1
    environment:
      VC:                           #vCenter Server IP-Adress
      VC_USERNAME:                  #vCenter Username
      VC_PASSWORD:                  #User Password
      TAG_URN:                      #vSphere Tag URN
      TAG_ACTION: attach            #attach or detach
    annotations:
      topic: resource.pool.created  #topic key-value annotation
{{< /highlight >}}

```
┌─[rguske@rguske-a01] - [~/_DEV]
└─[0] <> kubectl get pods -n openfaas-fn
NAME                        READY   STATUS    RESTARTS   AGE
pytag-fn-85bd47b68d-lss69   1/1     Running   0          12d
```

<center><a href="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115308.jpg"><img src="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115308.jpg" width="900"></img></a></center>

<center><a href="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115819.jpg"><img src="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115819.jpg" width="900"></img></a></center>

<center><a href="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115924.jpg"><img src="/img/posts/201902_vspheretagfn/CapturFiles-20190217_115924.jpg" width="900"></img></a></center>

<center><a href="/img/posts/201902_vspheretagfn/CapturFiles-20190217_120037.jpg"><img src="/img/posts/201902_vspheretagfn/CapturFiles-20190217_120037.jpg" width="900"></img></a></center>

## Resources
Article   | Link
-------------  | -------------
**[@Github/embano1] Embano1 (Michael Gasch) Github Repo** | https://github.com/embano1
**[@Github/embano1] vcenter-connector** | https://github.com/embano1/vcenter-connector
**[@Github/embano1] pytagfn** | https://github.com/embano1/pytagfn
**[@Github/embano1] gotagfn** | https://github.com/embano1/pytagfn
**[@mgasch.com] Events, the DNA of Kubernetes** | https://www.mgasch.com/post/k8sevents/
**[@blog.alexellis.io] Get started with OpenFaaS and KinD** | https://blog.alexellis.io/get-started-with-openfaas-and-kind/
**[@docs.openfaas.com] Deployment guide for Kubernetes** | https://docs.openfaas.com/deployment/kubernetes/
**[@cloud.vmware.com] VMware PKS Landing Page** | https://cloud.vmware.com/vmware-pks
**[@Pivotal.io] Pivotal PKS Landing Page** | https://pivotal.io/platform/pivotal-container-service
**[@medium.com] Search "Functions as a Service"** | https://medium.com/search?q=functions%20as%20a%20service


{{< tweet 902580607468789761 >}}