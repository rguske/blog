---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "vSphere with Tanzu - Troubleshooting HAProxy deployment"
date: 2020-11-16T22:59:14+01:00
draft: true
featuredImage: /img/haproxytroubleshooting_cover.jpg
categories: ["vSphere", "Tanzu", "Troubleshooting", "Kubernetes"]
tags:
- November2020
- vSphere
- Kubernetes
- Tanzu
- HAProxy
---
## Introduction

You may already heard and read about our latest changes regarding our Kubernetes offering(s) [vSphere with Tanzu](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-C163490C-BE03-4DFE-8A03-5316D3245765.html) also former known as *vSphere with Kubernetes*. Personally, I was totally excited and full of anticipation to make my hands dirty and to experience how this new deployment option --> [vSphere Networking](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-C3048E95-6E9D-4AC3-BE96-44446D288A7D.html) works as an alternative to [NSX-T](https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-B1388E77-2EEC-41E2-8681-5AE549D50C77.html) and how HAProxy is doing it's job within this construct. I don't know how you are dealing with new installations of the "NEW" but I always read the documentation first... ... NOT ... :speak_no_evil: ... but I should! :see_no_evil:

Honestly! Doesn't matter if we are talking about a Proof-of-Concept (PoC) or a test environment implementation, what counts by the end of the day, is a properly working solution and this is why we should be prepared best. Through this article, I will even more stress this point because I did a mistake which costs me a lot of time in troubleshooting the failure I've received. On the other hand and I would say "*as always*", it brought me some new insights.

## Preperations

This article will not describe the vSphere with Tanzu installation itself. For this, I would like to point you to this excellent [vSphere with Tanzu Quick Start Guide V1a](https://core.vmware.com/resource/vsphere-tanzu-quick-start-guide-v1a#_Toc53677531) which VMware is maintaining on the [Cloud Platform Tech Zone](https://core.vmware.com/).

Also! For being better prepared as well as for documentation purposes, we are providing a checklist in form of an Excel file which can be shared or forwarded to the networking engineer of your confidence (*Figure I*).

{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_105840.jpg" caption="Figure I: Network Stack Checklist" src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_105840.jpg" >}}

### It should be a Network(-ID)




| Subnet Mask | /28 | /27 | /26 | /25 | /24 |
|:---: |:---: |:---: | :---: | :---: | :---:
| Host IPs | 14 | 30 | 62 | 162 | 254 |

*\* : I excluded Network-ID and Broadcast-ID*



{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_094920.jpg" caption="Figure I: " src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_094920.jpg" >}}

{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_092255.jpg" caption="Figure I: " src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_092255.jpg" >}}



{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_104643.jpg" caption="Figure I: " src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_104643.jpg" >}}



{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_112102.jpg" caption="Figure I: " src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_112102.jpg" >}}

{{< image src="/img/posts/202011_haproxytrouble/CapturFiles-20201117_115703.jpg" caption="Figure I: " src-s="/img/posts/202011_haproxytrouble/CapturFiles-20201117_115703.jpg" >}}

`kubectl vsphere login --insecure-skip-tls-verify --vsphere-username administrator@jarvis.lab --server=10.10.18.1`

{{< admonition failure "ERROR" true >}}
ERRO[0005] Error occurred during HTTP request: Get https://10.10.18.19/wcp/loginbanner: dial tcp 10.10.18.19:443: connect: no route to host
There was an error when trying to connect to the server.\n
Please check the server URL and try again.FATA[0005] Error while connecting to host 10.10.18.19: Get https://10.10.18.19/wcp/loginbanner: dial tcp 10.10.18.19:443: connect: no route to host.
{{< /admonition >}}


{{< admonition info "Description" true >}}
Description
Deploy the Appliance with 3 nics: a Management network (Supervisor -> HAProxy dataplane), a single Workload network and a dedicated Frontend network. Load-balanced IPs are assigned on the Frontend network
{{< /admonition >}}




{{< admonition info "Description" true >}}
This IP must be outside of the Load Balancer IP Range
{{< /admonition >}}



{{< admonition info "Description" true >}}
As such, these ranges must not overlap with the IPs assigned for the appliance or any other VMs on the network.
{{< /admonition >}}