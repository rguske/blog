---
title: "VIC Part I: OVA Deployment"
date: 2018-07-13T13:18:38+02:00
draft: false
image: /img/Hamburg_Hafen1.JPG
tags:
- VIC
- Container
- VMware
- vSphere
---

**Environment Pre-Requisites:**

- vSphere Enterprise Plus license or vSphere Remote Office Branch Office (ROBO) Advanced (!)
 * Dependency on the vDistributed Switch
 * VIC also supports VMware NSX
- User with administrative credentials to vCenter
- Internet Access for downloading images
- min. two vDistributed Switch Port Groups
 * for public communication (VCH to external network)
 * for inter containers communication create a dedicated port group for use as the bridge network for each VCH
 * If DHCP is not available on these segments, please, request a range of free IP-Addresses.

---
**Note:** You´ll find all necessary pieces of information with regards to Licensing as well as Deployment Requirements on the official <a href="https://vmware.github.io/vic-product/assets/files/html/1.4/vic_vsphere_admin/vic_installation_prereqs.html" target="_blank">VIC Github Page</a>.

---
To start with, we have to download the latest bits from myvmware.com here --> <a href="https://my.vmware.com/en/web/vmware/info/slug/datacenter_cloud_infrastructure/vmware_vsphere_integrated_containers/1_4" target="_blank">VIC Version 1.4.1</a>. After having downloaded the 3,12 GB ova-file we´ll start provisioning the VIC Virtual Appliance over the vSphere Web Client onto your vSphere Datacenter, Cluster or ESXi Host. I suppose that you are already familiar with these steps but if not, please go <a href="https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html" target="_blank">here</a> first.

If you´re already familiar with it and you´re more interested in automting an OVA deployment, I´d recommend reading this <a href="http://cloudmaniac.net/ova-ovf-deployment-using-govc-cli/" target="_blank"> Post </a> by <a href="https://twitter.com/woueb" target="_blank"> Romain Decker </a>.
<a href="/img/posts/vic_getting_started/CapturFiles-20180607_104118.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_104118.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_111348.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_111348.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_111747.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_111747.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_111946.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_111946.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_112111.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_112111.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_112335.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_112335.jpg" height="600"</img></a>

At this point, I´d like to stress out again to the VIC documentation regarding the <a href="https://vmware.github.io/vic-product/assets/files/html/1.4/vic_vsphere_admin/deploy_vic_appliance.html" target"_blank">use of SSH</a>. SSH is needed when you perform upgrades or the following:

<a href="/img/posts/vic_getting_started/CapturFiles-20180619_084703.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180619_084703.jpg" width="700"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_112527.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_112527.jpg" height="600"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_112527.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_112527.jpg" height="600"</img></a>

If you decide to use static IP-Addresses like me, please use spaces and not commas to separate multiple DNS-Servers.

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_113830.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_113830.jpg" height="600"</img></a>

I´ve also decided to create an example user through the wizard, which gets the username prefix you´ve chosen in point 5 in this section. I´m fine with the predetermined prefix *vic*.

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_113931.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_113931.jpg" height="700"</img></a>

<a href="/img/posts/vic_getting_started/CapturFiles-20180607_113956.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180607_113956.jpg" height="600"</img></a>

Lean back and let the vCenter do its job... ... ...FINISHED!

By the way! If you think *"Hey this H5-Client Dark Theme looks very slick! Where I can toggle the switch?"* unfortunately one has to name, that this is not a feature in the vSphere H5-Client, it´s a Browser-Extension by Jens L. aka BeryJu and available for Chrome and Firefox. You´ll find him on <a href="https://github.com/BeryJu" target="_blank">Github</a> as well as on his <a href="https://beryju.org/en" target="_blank">Blog</a>. Thanks, Jens for the nice work.

---
**IMPORTANT!** 

When you add the extension, VMware will not provide support when you´re facing issues!
And - I´d recommend using only browsers where the language is set to English! In other cases, you could hit the issue <a href="https://github.com/BeryJu/dark-vcenter/issues/36" target="_blank">Other browser language than English breaks CSS inject #36</a>

---
Here you´ll find the extensions.

<a href="https://chrome.google.com/webstore/search/Dark%20vCenter" target="_blank">Dark-vCenter for Google Chrome</a>

<a href="https://addons.mozilla.org/en-US/firefox/addon/dark-vcenter/?src=search" target="_blank">Dark-vCenter for Mozilla Firefox</a>

---

The next step is to complete the VIC appliance installation through the establishment of the connection to our vCenter Server as well as Platform Service Controller. Here we have to enter the vCenter Server address (FQDN) and the Single Sign-on credentials for a vSphere administrator account. In my case, I´m using an embedded PSC and thus, I can leave the fields for the External PSC Instance empty.

<a href="/img/posts/vic_getting_started/CapturFiles-20180608_105035.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180608_105035.jpg" widht="800"</img></a>
If you´ve entered your credentials correctly, you´ll be forwarded to the VIC Getting Started page which you could always open by using the IP-Address or better using the FQDN (of course the short name as well) over port 9443. In my example https://vic01.lab.jarvis.local:9443/

<a href="/img/posts/vic_getting_started/CapturFiles-20180608_105155.jpg"><img src="/img/posts/vic_getting_started/CapturFiles-20180608_105155.jpg" width="800"</img></a>

---
Continue with:

<a href="/post/vic-part-ii-vsphere-client-plug-in">**VIC Part II: vSphere Client Plug-In**</a>

<a href="/post/vic-part-iii-deployment-of-a-virtual-container-host/">**VIC Part III: Deployment of a Virtual Container Host**</a>

<a href="/post/vic-part-iv-docker-run-a-container-vm">**VIC Part IV: docker run a Container-VM**</a>

---
Previous article:

<a href="/post/vmware-vsphere-integrated-containers-overview">**vSphere Integrated Containers: Overview**</a>