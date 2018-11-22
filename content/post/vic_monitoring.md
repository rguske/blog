---
title: "Monitor vSphere Integrated Containers"
date: 2018-11-10T13:24:09+01:00
draft: true
image: /img/vic_monitoring_cover.jpg
tags:
- VIC
- Container
- vSphere
- Monitoring
- vROps
- Dashboard
---
Treating Containers as <a href="https://en.wikipedia.org/wiki/First-class_citizen" target="_blank">First-Class Citizens</a> is what vSphere Integrated Containers can leverage to IT-Ops Teams through instantiating a Container-Image as a Virtual Machine, as I´ve already described it in more details in my previous posts.

> Therefore you don´t have to build out a separate, tailored infrastructure stack and can continue to leverage existing Scalability, Security and **Monitoring** capabilities.

#### **Monitoring** is the keyword for this post
Based on several talks with customers and colleagues who already made their experiences running containerized applications in production and are using vSphere Integrated Containers for their way to go, I´ve thought it´s time to create a Dashboard to monitor those workloads with VMware´s Cloud-Management Solution, vRealize Operations Manager (vROps), to show the value, because they are virtual machines. The granularity vROps has inside vSphere will help us here to collect and show performance and utilization metrics.

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181220_125457.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181220_125457.jpg" width="850"></img></a>

The dashboard is splitted into two "parts". The upper part gives you more details about the utilization and properties of the Virtual Container Host itself. This means that both one and the other part, Resource Pool and VCH-VM (Docker endpoint) are covered by the upper part.

The lower part gives you more details of the performance and VM-properties of a specific service of an application (instantiated Container-VM).

<a href="/img/posts/201811_post_monitoring/dashboard_parts.jpg"><img src="/img/posts/201811_post_monitoring/dashboard_parts.jpg" width="850"></img></a>

I´ve also considered to build out each part in a seperate dashboard, so one for the VCH and another one for the Container-VM, but nevertheless I thought its better to handle it with an "All-in-One" variant.

###### Things you have to do first before the import

1. <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-43ACB276-C9A3-490A-B65E-3D0EA66E8AC8.html" target="_blank">Group Type</a> (To categorize your objects in vR Ops)
2. <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-7CD02B2B-4BB1-4BD3-B24E-5F630B94A7EF.html" target="_blank">Dynamic Custom Group</a> (Dynamic assignment of VCH´s through membership criteria)
3. <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-75E5B784-6CE7-4048-AD77-89A9E21C2AC2.html" target="blank">Metric Configuration</a> (To define a specific set of metrics)

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_102513.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_102513.jpg" width="850"></img></a>

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_102917.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_102917.jpg" width="850"></img></a>

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_103022.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_103022.jpg" width="850"></img></a>

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_103104.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_103104.jpg" width="400"></img></a>

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_104030.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_104030.jpg" width="850"></img></a>

Name: rg_vch_resource_overview

```
<?xml version="1.0" encoding="UTF-8"?>
<AdapterKinds>
    <AdapterKind adapterKindKey="VMWARE">
        <ResourceKind resourceKindKey="ResourcePool">
            <Metric attrkey="summary|total_number_vms" label="# cVM" unit="cVM" yellow="" orange="" red="" link=""/>
            <Metric attrkey="summary|number_running_vms" label="# running cVM" unit="cVM" yellow="" orange="" red="" link=""/>
            <Metric attrkey="cpu|corecount_provisioned" label="# CPU Cores" unit="cores" yellow="" orange="" red="" link="" isProperty="true"/>
            <Metric attrkey="config|cpuAllocation|limit" label="CPU Limit" unit="MHz" yellow="" orange="" red="" link="" isProperty="true"/>
            <Metric attrkey="config|cpuAllocation|reservation" label="CPU Reservation" unit="GHz" yellow="" orange="" red="" link="" isProperty="true"/>
            <Metric attrkey="config|memoryAllocation|limit" label="Mem Limit" unit="MB" yellow="" orange="" red="" link="" isProperty="true"/>
            <Metric attrkey="config|memoryAllocation|reservation" label="Mem Reservation" unit="MB" yellow="" orange="" red="" link="" isProperty="true"/>
            <Metric attrkey="mem|consumed_average" label="Mem consumed" unit="MB" yellow="" orange="" red="" link=""/>
            <Metric attrkey="mem|overhead_average" label="VM Mem overhead" unit="MB" yellow="" orange="" red="" link=""/>
            <Metric attrkey="OnlineCapacityAnalytics|capacityRemainingPercentage" label="Capacity Remaining" unit="%" yellow="" orange="" red="" link=""/>
        </ResourceKind>
    </AdapterKind>
</AdapterKinds>
```

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_104204.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_104204.jpg" width="850"></img></a>

Name: 
```
<?xml version="1.0" encoding="UTF-8"?>
<AdapterKinds>
    <AdapterKind adapterKindKey="VMWARE">
        <ResourceKind resourceKindKey="VirtualMachine">
            <Metric attrkey="config|hardware|num_Cpu" label="# vCPU" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="cpu|usagemhz_average" label="CPU Usage" unit="MHz" yellow="" orange="" red="" link=""/>
            <Metric attrkey="cpu|usage_average" label="CPU Usage Avg" unit="MHz" yellow="" orange="" red="" link=""/>
            <Metric attrkey="cpu|readyPct" label="CPU Ready" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="cpu|costopPct" label="CPU Costop" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="mem|usage_average" label="Mem Usage" unit="%" yellow="" orange="" red="" link=""/>
            <Metric attrkey="mem|vmMemoryDemand" label="Mem Demand" unit="KB" yellow="" orange="" red="" link=""/>
            <Metric attrkey="mem|balloonPct" label="Mem Ballooning" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="mem|swapped_average" label="Mem Swapped" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="net|received_average" label="Net RX" unit="" yellow="" orange="" red="" link=""/>
            <Metric attrkey="net|transmitted_average" label="Net TX" unit="" yellow="" orange="" red="" link=""/>
        </ResourceKind>
    </AdapterKind>
</AdapterKinds>
```

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_110238.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_110238.jpg" width="850"></img></a>

<a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_105712.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_105712.jpg" width="850"></img></a>

If you would like to make use of what the vCommunity already made publicly available or (even better) if you would like to share your own creations, please go here: https://vrealize.vmware.com/sample-exchange/

The vR Ops-Team will appreciate this :thumbsup:

{{< tweet 1072175444370305025 >}}

And if you are already a *vR Ops-Ninja* and you are not already on this list, reach out to <a href="https://twitter.com/johnddias" target="_blank">John Dias</a> and he´ll put your name on the list as well.

{{< tweet 1068561326216228865 >}}

Here´s the list: https://twitter.com/johnddias/lists/vrops-friends1

---

I´d like to explain you how I´ve created a Dasboard in vRealize Operations Manager to get an overview or to *monitor* the utilization of a <a href="https://rguske.github.io/post/vsphere-integrated-containers-part-iii-deployment-of-a-virtual-container-host/" target="_blank">Virtual Container Host</a> as well as the performance of the <a href="https://rguske.github.io/post/vsphere-integrated-containers-part-iv-docker-run-a-container-vm/" target="_blank">Container-VM´s</a> which are running inside the (VCH-)Resource-Boundary.

Part of the <a href="https://www.vmware.com/products/vrealize-operations.html" target="_blank">Advanced Edition</a>.