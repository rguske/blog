---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "vSphere with Tanzu - SupervisorControlPlaneVM stucks in state NotReady"
date: 2021-04-22T10:14:39+02:00
draft: true
featuredImage: /img/v7k8s_svcp_not_ready_cover.png
categories: ["Kubernetes", "Cloud Native", "VMware"]
tags:
- April2021
- Kubernetes
- Tanzu
- TKGs
- vSphere
---
## Introduction

Power outage related circumstances recently brought my vSphere with Tanzu Workload Management[^1] enabled homelab cluster into a not desired state which of course I had to get rid of and in the end did. And I do not mean that I have to protect my homelab for the next power outage by finally installing a UPS :wink:.

<center> {{< tweet 1384421366154285056 >}} </center>

What I actually mean is the following...

## Failed to get available workloads: bad gateway

The last couple of weeks I spent a lot of time using my Tanzu Kubernetes Cluster(s)[^2] to get my head around as well as my hands dirty on this awesome project Knative[^3]. Interacting with the vSphere Supervisor Cluster[^4] is necessary for a couple of reasons like e.g. the instantiation of a new Tanzu Kubernetes Cluster. This time, my attempt to login into mine unfortunately ends quicker than expected with the error message:

```shell
kubectl vsphere login --server=10.10.18.10 --vsphere-username administrator@jarvis.lab --insecure-skip-tls-verify

Password:
FATA[0004] Failed to get available workloads: bad gateway
Please contact your vSphere server administrator for assistance.
```

My first instinct was to quickly have a look at the Workload Management subsection in the vSphere Client (*Menu --> Workload Management or ctrl + alt + 7*) and here my suspicion that something isn't good at all was confirmed.

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_092403.jpg" caption="Figure I: Supervisor Cluster stucks in configuring state" src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_092403.jpg" >}}

I observed this `configuring` state for a while longer and I was hoping it gets fixed automagically but it doesn't.

> Seriously, I do believe that this is really due to various (again) circumstances which I was facing with my homelab and normally the desired state (`ready`) for the Supervisor Cluster or more specifically, for the Supervisor Control Plane VMs will be reached automatically.

## Troubleshooting mode on

### HealthState WCP service unhealthy

The next move I did was checking the state of my Tanzu Kubernetes Cluster in the vSphere Client but they were disappeared. Things went strange :ghost:.

Consequently, I checked the overall health of my vCenter Server as well as the HealthState of the services and especially the `wcp` service (Workload Control Plane). Checking the service state can be done in two ways:

1. vCenter Appliance Management Interface aka VAMI (vcenter url:5480)
2. via the `shell` - which requires an enabled and running `ssh` service

The `shell` output was the following:


```shell
root@vcsa [ ~ ]# vmon-cli --status wcp
Name: wcp
Starttype: AUTOMATIC
RunState: STARTED
RunAsUser: root
CurrentRunStateDuration(ms): 72360341
HealthState: UNHEALTHY
FailStop: N/A
MainProcessId: 15679
```

And here a little bit more coloured:

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_092113.jpg" caption="Figure II: vCenter Server service wcp state unhealthy " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_092113.jpg" >}}

Restarting the service was the only logical consequence I thought was appropriate and so I did:

```shell
root@vcsa [ ~ ]# vmon-cli --restart wcp
Completed Restart service request.
root@vcsa [ ~ ]# vmon-cli --status wcp
Name: wcp
Starttype: AUTOMATIC
RunState: STARTED
RunAsUser: root
CurrentRunStateDuration(ms): 9709
HealthState: HEALTHY
FailStop: N/A
MainProcessId: 46317
```

### Supervisor Control Plane node status `NotReady`

Having the `wcp` service back in operating state led me to start over from where I began. This time logging on into my Supervisor Cluster went well and I checked the state here again by simply executing `kubectl get nodes`.


```shell
kubectl get nodes
NAME                               STATUS     ROLES    AGE     VERSION
42100ed03ae877fd39716f909d57822e   NotReady   master   7d19h   v1.19.1+wcp.2
4210a2e2fa945c1a13308fac6d00ed96   Ready      master   7d19h   v1.19.1+wcp.2
4210c09deaaa5a2e2ddc86d3e0c0b0f3   Ready      master   7d19h   v1.19.1+wcp.2
```

With this new gains, I also checked the Virtual Machine state via the Remote Console and surprisingly, it seems that the power outages affected this SV VM badly. The monitor windows was swarmed with error messages saying `print_req_error: I/O error, dev sda, sector ...`

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CaptureFiles-20-04-22.jpg" caption="Figure III: Control Plane VM status NotReady" src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CaptureFiles-20-04-22.jpg" >}}

I was also trying to figure out if there's a way to get this fixed on the Operating System level but there was no way or at least there wasn't one for my expertise.

Just for documentation purposes, I put the output from `kubectl get nodes 42100ed03ae877fd39716f909d57822e -o yaml > degraded_node.yaml` as well as from `kubectl describe nodes 42100ed03ae877fd39716f909d57822e` in here as well.


```shell
kubectl get nodes 42100ed03ae877fd39716f909d57822e -o yaml > degraded_node.yaml
```

```yaml
## shortened
[...]

conditions:
  - lastHeartbeatTime: "2021-04-19T14:54:14Z"
    lastTransitionTime: "2021-04-19T17:25:35Z"
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: MemoryPressure
  - lastHeartbeatTime: "2021-04-19T14:54:14Z"
    lastTransitionTime: "2021-04-19T17:25:35Z"
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: DiskPressure
  - lastHeartbeatTime: "2021-04-19T14:54:14Z"
    lastTransitionTime: "2021-04-19T17:25:35Z"
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown
    type: PIDPressure
  - lastHeartbeatTime: "2021-04-19T14:54:14Z"
    lastTransitionTime: "2021-04-19T17:25:35Z"
    message: Kubelet stopped posting node status.
    reason: NodeStatusUnknown
    status: Unknown

[...]

```

```shell
## shortened

kubectl describe nodes 42100ed03ae877fd39716f909d57822e

Name:               42100ed03ae877fd39716f909d57822e
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=42100ed03ae877fd39716f909d57822e
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 14 Apr 2021 13:56:36 +0200
Taints:             node.kubernetes.io/unreachable:NoExecute
                    node-role.kubernetes.io/master:NoSchedule
                    node.kubernetes.io/unreachable:NoSchedule
Unschedulable:      false
Lease:              Failed to get lease: leases.coordination.k8s.io "42100ed03ae877fd39716f909d57822e" is forbidden: User "sso:Administrator@jarvis.lab" cannot get resource "leases" in API group "coordination.k8s.io" in the namespace "kube-node-lease"
Conditions:
  Type             Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----             ------    -----------------                 ------------------                ------              -------
  MemoryPressure   Unknown   Mon, 19 Apr 2021 16:54:14 +0200   Mon, 19 Apr 2021 19:25:35 +0200   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure     Unknown   Mon, 19 Apr 2021 16:54:14 +0200   Mon, 19 Apr 2021 19:25:35 +0200   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure      Unknown   Mon, 19 Apr 2021 16:54:14 +0200   Mon, 19 Apr 2021 19:25:35 +0200   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready            Unknown   Mon, 19 Apr 2021 16:54:14 +0200   Mon, 19 Apr 2021 19:25:35 +0200   NodeStatusUnknown   Kubelet stopped posting node status.

[...]

```

## vSphere ESXi Agent Manager

If the desired state won't be reached again automatically and if I cannot fix this on the VM itself on my own, what option remains then?

Well, with the great hint from my well appreciated colleague [Dominik Zorgnotti](https://www.why-did-it.fail/), who was pointing me to the **vSphere ESXi Agent Manager (EAM)**[^5], I was able to fix this pretty simple and fast. To be honest, I was never in the situation to make use of the EAM before and therefore it wasn't on my radar at all but I was quiet happy to get this component known now. You will find the EAM in the vSphere Client under *Menu -> Administration -> vCenter Server Extensions -> vSphere ESXi Agent Manager*.

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_093111.jpg" caption="Figure IV: vSphere ESXi Agent Manager" src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_093111.jpg" >}}



{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095429.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095429.jpg" >}}


{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095346.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095346.jpg" >}}



{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095620.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095620.jpg" >}}

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095523.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_095523.jpg" >}}

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_100834.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_100834.jpg" >}}

{{< image src="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_100748.jpg" caption="Figure I: " src-s="/img/posts/202104_v7k8s_supervisorvm_not_ready/CapturFiles-20210422_100748.jpg" >}}













```shell
kubectl get nodes
NAME                               STATUS   ROLES    AGE     VERSION
42103deb5d43d5f75f4623a336165684   Ready    master   3m37s   v1.19.1+wcp.2
4210a2e2fa945c1a13308fac6d00ed96   Ready    master   7d20h   v1.19.1+wcp.2
4210c09deaaa5a2e2ddc86d3e0c0b0f3   Ready    master   7d20h   v1.19.1+wcp.2
```








| | | |
|:---: |:---: |:---: | :---: |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |


{{< mermaid >}}
graph LR;
    subgraph HAProxy Appliance
    A[ Management ]
    B[ Workload ]
    C[ Frontend / VIPs ]
    end
    subgraph Traffic from
    G( Client / Service )
    end
    subgraph Load Balancer VIPs
    E[ Supervisor Cluster VMs ]
    F[ Guest Cluster VMs ]
    end
    A & B & C --- E
    B & C --- F
    G -.- C -.-> F & E
{{< /mermaid >}}

[^1]: [vSphere Workload Domain](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-152BE7D2-E227-4DAA-B527-557B564D9718.html)
[^2]: [What Is a Tanzu Kubernetes Cluster?](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-DC22EA6A-E086-4CFE-A7DA-2654891F5A12.html)
[^3]: [Knative Homepage](https://www.knative.dev)
[^4]: [Configuring and Managing a Supervisor Cluster](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-21ABC792-0A23-40EF-8D37-0367B483585E.html)
[^5]: [vSphere ESXi Agent Manager](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.monitoring.doc/GUID-D56ABFF4-4529-409C-9AA2-8D8D4E235601.html)