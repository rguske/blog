---
title: "Monitoring Container VMs with vRealize Operations Manager"
date: 2019-01-21T23:00:09+01:00
draft: false
image: /img/vic_monitoring_cover.jpg
thumbnail: /img/vic_monitoring_thumbnail.jpg
tags:
- January2019
- VIC
- Container
- vSphere
- Monitoring
- vRealize Operations Manager
- Dashboard
---
Based on talks with customers who already made their experiences running containerized applications in test as well as in production and who are using vSphere Integrated Containers for their way to go, I decided to build a dashboard to monitor those workloads (Container-VMs) with VMware´s Cloud Management Solution, <a href="https://www.vmware.com/products/vrealize-operations.html" target="_blank">*vRealize Operations Manager (vR Ops)*</a>.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190120_030628.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190120_030628.jpg"></img></a></center>

<center>*Figure I: VIC monitoring high level overview*</center>

## **The Dashboard**
Treating Containers as <a href="https://en.wikipedia.org/wiki/First-class_citizen" target="_blank">"First-Class Citizens"</a> is what vSphere Integrated Containers can offer to IT-(Dev)Ops Teams through instantiating a Container-Image as a Virtual Machine.

> Therefore you don´t have to build out a separate, tailored infrastructure stack and can continue to leverage existing Scalability, Security and **Monitoring** capabilities.

> *Source: <a href="https://rguske.github.io/post/vmware-vsphere-integrated-containers-introduction/" target="_blank">VMware vSphere Integrated Containers: Introduction</a>*

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190114_100348.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190114_100348.jpg"></img></a></center>

I´ve splitted the dashboard into two parts because I wanted to get as most as valuable content displayed on the screen without scrolloing down the page (depending on your resolution :wink:). The upper part gives you more details about the **utilization** and properties of the Virtual Container Host itself. Thus resource pool as well as the Virtual Container Host VM (Docker endpoint) are covered by the <span style="color:blue">upper part</span>.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190114_103053.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190114_103053.jpg"></img></a></center>

The <span style="color:orange">lower part</span> is focused on the **performance** of each Container-VM (by selecting the cVM) running inside the resource pool, and by the end of the day, on the service which will be provided by the container.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190114_103149.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190114_103149.jpg"></img></a></center>

I also considered to build out each part in a separate dashboard, so one for the Virtual Container Host(s) and another one for the Container-VM(s), but nevertheless I decided to go with the "All-in-One" variant.

## Download via VMware {code}
If you like the dashboard and you are interested to monitor your containerized applications in terms of performance and utilization, you can download it through the following link: https://code.vmware.com/samples?id=5283

**But** before you can make use of it you have to do some preperations first.

## Things you have to do first before the import

**In vSphere**

1. Create a new <a href="https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vcenterhost.doc/GUID-E8E854DD-AA97-4E0C-8419-CE84F93C4058.html" target="_blank">vSphere-Tag</a>

Open up the vSphere-Client and create a new <a href="https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vcenterhost.doc/GUID-BA3D1794-28F2-43F3-BCE9-3964CB207FB6.html?hWord=N4IghgNiBcIC5gOYAIDGY4FNEHsBOAniAL5A" target="_blank">vSphere-Tag Category</a> as well as a vSphere-Tag called *Virtual Container Host* and assign the Tag to every Virtual Container Host **Resource Pool(!)**.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_101420.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_101420.jpg" width="550"></img></a></center>

I´m going to explain the reason why we need this in a moment.

**In vR Ops**

1. Create a <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-43ACB276-C9A3-490A-B65E-3D0EA66E8AC8.html" target="_blank">Group Type</a> to categorize objects in vR Ops.
2. Create a <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-7CD02B2B-4BB1-4BD3-B24E-5F630B94A7EF.html" target="_blank">Dynamic Custom Group</a> for dynamic assignment of Virtual Container Hosts through a membership criteria.
3. Create two new <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-75E5B784-6CE7-4048-AD77-89A9E21C2AC2.html" target="blank">Metric Configurations</a> to define a specific set of metrics for the VCH as well as for the cVM.
4. Import two new <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-BC389BD4-8F24-42D9-82B1-C33A87F738B6.html?hWord=N4IghgNiBcIGoEsCmB3AziAvkA" target="_blank">*Views*</a> (VCH properties & cVM properties)

Let´s start with **#1**, the creation of a *Group Type*, a superior group for our *Dynamic Custom Group* which we´ll configure in step **#2**. When you´re already logged in into vR Ops go to *Administration* and select *Group Types* which is under *Configuration* on the left side. Create a new one and call it *vSphere Integrated Containers* for example.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_102513.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_102513.jpg"></img></a></center>

It is followed by the mentioned *Dynamic Custom Group* which is not found under *Administration* but under *Environment* and then *Groups and Applications*. Create a new one and call it e.g. *Virtual Container Hosts*.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20181204_102917.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20181204_102917.jpg"></img></a></center>

One striking aspect about *Dynamic Custom Groups* is the fact that we can define the *Membership* based on a specific criteria. During my first tests, I´ve defined the membership based on the *Prefix* of a Virtual Container Host name like **vch-**app01 for example. Asking me naming conventions is an important topic and every organization has its own, so it won´t fit for everyone and therefore I had to find another criteria to let the dynamic assignment to this group happen.

I thought *vSphere-Tags* is a perfect alternative as a criteria. Consequently, we have to configure it for the *Object Type Resource Pool* like the screenshot below shows.

*Properties* --> *Summary|vSphere Tag* is *Virtual Container Host*

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190107_101933.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190107_101933.jpg" width="850"></img></a></center>

Hit *Preview* to see if it works.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190107_110525.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190107_110525.jpg" width="400"></img></a></center>

The next step, step **#3**, is the configuration of a specific metric set which will only show us what we´ll see in our dashboard reagrding the Virtual Container Host Resource Pool as well as the Container-VM. To configure those, go to *Administration* and *Configuration* again and select this time *Metric Configurations*.

Select the folder *ReskndMetric* and hit the plus sign above to create a new one. I´ve named the first of my two configurations *vch_resource_utilization*. Of course you can title it as you like but we have to keep in mind that we have to select this one after the dashboard import.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_103948.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_103948.jpg" width="400"></img></a></center>

After assigning the name, please copy the following xml code and paste it into your new metric configuration.

```xml
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

The second metric configuration is specific for the performance of the Container-VM. I´ve named it *container_vm_performance*.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_103638.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_103638.jpg" width="400"></img></a></center>

Copy the xml code and again simply paste it into.

```xml
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

The last point (**#4**) before we are ready to go is the <a href="https://docs.vmware.com/en/vRealize-Operations-Manager/7.0/com.vmware.vcom.core.doc/GUID-42F41582-E2EE-4BD3-9751-F65C886E1118.html" target="_blank">import</a> of the two custom views. Click ***Dashboards***, and then in the left pane click ***Views***. Click on the gear icon, select ***Import View*** and navigate to the folder where you´ve downloaded the files to. To ensure that the dashboard will work smoothly after importing it check if the import of the two custom *Views* were successfull. Just filter by *"vm properties"* and your search should result in the following:

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_105533.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_105533.jpg" width="800"></img></a></center>

---

## Conclusion

vRealize Operations Manager has evolved greatly over time and building custom dashboards according to your own ideas and demands is quite fun. Let´s just take the new *Dashboard Canvas* which was shipped with version 7.0 as an example. It´s not all about how cool it looks like but rather than the simplicity it brings during the creation of an dashboard.

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_111340.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_111340.jpg"></img></a></center>

<center>*Widget interactions of the Virtual Container Host(s) dashboard*</center>

Here are two posts I would like to recommend to read with regard to:

1. <a href="https://blogs.vmware.com/management/2018/10/dashboards-made-easy-with-vrealize-operations.html" target="_blank">Dashboards Made Easy with vRealize Operations</a>

2. <a href="https://blogs.vmware.com/management/2018/10/bigger-stronger-faster-dashboards-using-vrealize-operations-7-0.html" target="_blank">Bigger, Stronger, Faster Dashboards using vRealize Operations 7.0</a>

The <a href="https://twitter.com/search?q=%23vcommunity&src=typd" target="_blank">#vCommunity</a> is more active than ever on sharing great work and if you like to make use of what the #vCommunity already cooked or even better if you like to share your own creations, please go here: https://vrealize.vmware.com/sample-exchange/ and give it a shout-out on Twitter. Our great Product-Manager´s will hear you :sunglasses:.

<center> {{< tweet-single 1079442191972483073 >}} </center>

<center> {{< tweet 1081415672712679429 >}} </center>

And in case you are already a *vR Ops-Ninja* and you are not on this list https://twitter.com/johnddias/lists/vrops-friends1 reach out to <a href="https://twitter.com/johnddias" target="_blank">John Dias</a> and he´ll put your name on the list as well.

<center> {{< tweet 1068561326216228865 >}} </center>

<center><a href="/img/posts/201811_post_monitoring/CapturFiles-20190117_085401.jpg"><img src="/img/posts/201811_post_monitoring/CapturFiles-20190117_085401.jpg" width="150"></img></a></center>

<center>**THANK YOU**</center>

---

## Resources

**Article**                                                                          | **Link**
------------------------------------------------------------------------------------ |----------------------------------------------------------------------------
**[@VMware Blog] Dashboards Made Easy with vRealize Operations**                     | https://blogs.vmware.com/management/2018/10/dashboards-made-easy-with-vrealize-operations.html
**[@VMware Blog] Bigger, Stronger, Faster Dashboards using vRealize Operations 7.0** | https://blogs.vmware.com/management/2018/10/bigger-stronger-faster-dashboards-using-vrealize-operations-7-0.html
**[@VMware Blog] VMware´s Cloud Management Blog**                                    | https://blogs.vmware.com/management/
**[@VMware Blog] VMware´s Cloud-Native Blog**                                        | https://blogs.vmware.com/cloudnative