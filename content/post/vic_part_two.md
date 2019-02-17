---
title: "vSphere Integrated Containers Part II: vSphere Client Plug-In"
date: 2018-07-13T13:17:23+02:00
draft: false
image: /img/vic_part_ii_cover.jpg
tags:
- VIC
- Container
- VMware
- vSphere
---

In this section, I´d like to give you a walkthrough on how to install the VIC vSphere Client Plug-In for the vCenter Server Appliance. I´m already using the vCenter Server Appliance aka vCSA in Version 6.7.0. In general, it is important to know that a minimum version of vCenter Server 6.5.0d is required to make use of the plug-in.

However! It is also worth to be mentioned that the plug-in is not necessary or a requirement to run VIC as well as for the deployment of a Virtual Container Host aka **VCH**. But it makes the initial deployment of a VCH easier at the first beginning until you are more and more familiar with the vic-machine utility.
The first step from the installation of the vSphere Client Plug-In is to download the vic-machine-bundle from the VIC-Appliance by using <a href="https://en.wikipedia.org/wiki/CURL" target="_blank">`curl`</a>. 

To set up the necessary commands from within the vCSA we first have to enable SSH over the Virtual Appliance Management Interface aka VAMI by using port 5480. By default SSH is disabled.

https://lab-vcsa67-001:5480

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_092710.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_092710.jpg" width="800"</img></a></center>

Toggle the switch to enable SSH Login on the vCSA

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_092734.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_092734.jpg" width="800"</img></a></center>

But wait! To make sure if everything went right after the installation, we should check before and after. And how can we check if there isn´t a vic plug-in before the installation and afterwards it gets shown? Correct - over the Managed Object Browser aka MOB.

https://lab-vcsa67-001/mob

You won´t find anything declared with vic- under *content/ ExtensionManager/ extensionList*.

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_095438.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_095438.jpg" width="800"</img></a></center>

Now let's establish a ssh-connection to the vCSA.

```
ssh root@192.168.100.72
```

Write and execute:

```
shell
```

After the login we now want to download and unpack the vic-bundle tar-file from the VIC Appliance to start the installation of the VIC vSphere Client Plug-In. You should just change my IP-Address to yours in the following lines.

```
export VIC_ADDRESS=192.168.100.160
export VIC_BUNDLE=vic_v1.4.0.tar.gz
curl -kL https://${VIC_ADDRESS}:9443/files/${VIC_BUNDLE} -o ${VIC_BUNDLE}
```

```
tar -zxf ${VIC_BUNDLE}
cd vic/ui/VCSA
```

By using the Linux command `ls` (list)`-ltr` (sort by change date) you should see the downloaded file.
The next and final step before the installation of the vCenter plug-in can begin, is the execution of the installation script.

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_094828.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_094828.jpg" width="800" </img></a></center>

Execute the install.sh script by entering:

```
./install.sh
```

...and provide the requested data (vCSA FQDN as well as vCenter Server Administrator Credentials). By the end it should look like this:

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_095702.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_095702.jpg" width="800"</img></a></center>

Now let´s check again the extensionList over the vCenter Managed Object Manager - MOB.

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180707_090912.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180707_090912.jpg" width="800"</img></a></center>

Voila!
Okay, now we are close to make use of the vic plug-in capabilities in our vSphere-WebClient but before we can go ahead, we need to restart the H5-Client as well as the Flex-based vSphere Web Client by running the following commands:

```
service-control --stop vsphere-ui && service-control --start vsphere-ui
service-control --stop vsphere-client && service-control --start vsphere-client
```

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_100412.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_100412.jpg" width="800"</img></a></center>

Because we want to hold our vCSA as clean as a Appliance should be, we have to get rid of our tracks by starting the cleaning process through the deletion of the unpacked tar-file.

```
rm *.tar.gz
rm -R vic
```

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_100743.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_100743.jpg" width="600"</img></a></center>

---
**Disable** SSH when you are finished!

---

The VIC plug-in should now be available under *Menu/ vSphere Integrated Containers*.

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_092711.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_092711.jpg" width="300"</img></a></center>

<center><a href="/img/posts/vic_getting_started/CapturFiles-20180616_101449.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180616_101449.jpg" width="800"</img></a></center>

Now we have established the **Integration** into **vSphere** but what about the **Containers**? It won´t last long...

---
**Continue with:**

<a href="/post/vsphere-integrated-containers-part-iii-deployment-of-a-virtual-container-host/">**vSphere Integrated Containers Part III: Deployment of a Virtual Container Host**</a>

<a href="/post/vsphere-integrated-containers-part-iv-docker-run-a-container-vm">**vSphere Integrated Containers Part IV: docker run a Container-VM**</a>

---
**Previous articles:**

<a href="/post/vsphere-integrated-containers-part-i-ova-deployment">**vSphere Integrated Containers Part I: OVA Deployment**</a>

<a href="/post/vmware-vsphere-integrated-containers-introduction">**vSphere Integrated Containers: Introduction**</a>