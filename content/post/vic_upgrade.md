---
title: "Upgrade vSphere Integrated Containers"
date: 2018-08-08T00:08:02+02:00
draft: true
image: /img/vic_upgrade_post_cover.jpg
tags:
- VIC
- Container
- VMware
- vSphere
- Docker
---

```
./vic-machine-darwin create \
--name vch01 \
--container-name-convention mark7_{name} \
--syslog-address udp://lab-vrli-001.lab.jarvis.local:514 \
--compute-resource /Datacenter-South/host/nested-vSAN \
--image-store vsanDatastore \
--volume-store vsanDatastore/volumes:default \
--bridge-network vic-bridge \
--bridge-network-range 172.16.0.0/12 \
--public-network vic-public \
--dns-server 192.168.100.80 \
--container-network vic-container:container-net \
--container-network-gateway vic-container:192.168.100.1/24 \
--container-network-ip-range vic-container:192.168.100.0/24 \
--container-network-dns vic-container:192.168.100.80 \
--tls-cname vch01 \
--organization Jarvis \
--certificate-key-size 2048 \
--no-tlsverify \
--force \
--user adm.jarvis@LAB.JARVIS.LOCAL \
--thumbprint 4F:D3:9B:50:00:31:D9:84:9D:DA:CF:57:21:D6:0D:11:89:78:97:26 \
--target lab-vcsa67-001.lab.jarvis.local/Datacenter-South \
--ops-user jarvis@lab \
--ops-grant-perms \
--timeout 5m
```

```
INFO[0000] ### Installing VCH ####
INFO[0000] vSphere password for adm.jarvis@LAB.JARVIS.LOCAL:
INFO[0008] vSphere password for jarvis@lab:
INFO[0014] Generating self-signed certificate/key pair - private key in vch01/server-key.pem
WARN[0015] Configuring without TLS verify - certificate-based authentication disabled
INFO[0015] Validating supplied configuration
WARN[0015] If ops-user (jarvis@lab) belongs to the Administrators group, permissions on some resources might have been restricted
INFO[0016] vDS configuration OK on "vic-bridge"
INFO[0016] vDS configuration OK on "vic-container"
INFO[0016] vCenter settings check OK
INFO[0016] Firewall status: ENABLED on "/Datacenter-South/host/nested-vSAN/vesxi67-1.lab.jarvis.local"
INFO[0016] Firewall status: ENABLED on "/Datacenter-South/host/nested-vSAN/vesxi67-2.lab.jarvis.local"
INFO[0016] Firewall status: ENABLED on "/Datacenter-South/host/nested-vSAN/vesxi67-3.lab.jarvis.local"
INFO[0016] Firewall configuration OK on hosts:
INFO[0016] 	"/Datacenter-South/host/nested-vSAN/vesxi67-1.lab.jarvis.local"
INFO[0016] 	"/Datacenter-South/host/nested-vSAN/vesxi67-2.lab.jarvis.local"
INFO[0016] 	"/Datacenter-South/host/nested-vSAN/vesxi67-3.lab.jarvis.local"
INFO[0016] vCenter settings check OK
INFO[0016] License check OK on hosts:
INFO[0016]   "/Datacenter-South/host/nested-vSAN/vesxi67-1.lab.jarvis.local"
INFO[0016]   "/Datacenter-South/host/nested-vSAN/vesxi67-2.lab.jarvis.local"
INFO[0016]   "/Datacenter-South/host/nested-vSAN/vesxi67-3.lab.jarvis.local"
INFO[0016] DRS check OK on:
INFO[0016]   "/Datacenter-South/host/nested-vSAN"
INFO[0016]
INFO[0016] Creating Resource Pool "vch01"
INFO[0016] Creating appliance on target
INFO[0016] Network role "client" is sharing NIC with "public"
INFO[0016] Network role "management" is sharing NIC with "public"
INFO[0020] Creating directory [vsanDatastore] volumes
INFO[0021] Datastore path is [vsanDatastore] volumes
INFO[0022] Uploading images for container
INFO[0022] 	"bootstrap.iso"
INFO[0022] 	"appliance.iso"
WARN[0045] If ops-user (jarvis@lab) belongs to the Administrators group, permissions on some resources might have been restricted
INFO[0048] Waiting for IP information
INFO[0075] Waiting for major appliance components to launch
INFO[0075] Obtained IP address for client interface: "192.168.100.220"
INFO[0075] Checking VCH connectivity with vSphere target
INFO[0076] vSphere API Test: https://lab-vcsa67-001.lab.jarvis.local vSphere API target responds as expected
INFO[0089] Initialization of appliance successful
INFO[0089]
INFO[0089] VCH Admin Portal:
INFO[0089] https://192.168.100.220:2378
INFO[0089]
INFO[0089] Published ports can be reached at:
INFO[0089] 192.168.100.220
INFO[0089]
INFO[0089] Docker environment variables:
INFO[0089] DOCKER_HOST=192.168.100.220:2376
INFO[0089]
INFO[0089] Environment saved in vch01/vch01.env
INFO[0089]
INFO[0089] Connect to docker:
INFO[0089] docker -H 192.168.100.220:2376 --tls info
INFO[0089] Installer completed successfully
```

```
export DOCKER_HOST=192.168.100.220:2376
```

```
docker --tls info
```

```
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: v1.3.1-16055-afdab46
Storage Driver: vSphere Integrated Containers v1.3.1-16055-afdab46 Backend Engine
VolumeStores: default
vSphere Integrated Containers v1.3.1-16055-afdab46 Backend Engine: RUNNING
 VCH CPU limit: 22800 MHz
 VCH memory limit: 30.49 GiB
 VCH CPU usage: 79 MHz
 VCH memory usage: 1.637 GiB
 VMware Product: VMware vCenter Server
 VMware OS: linux-x64
 VMware OS version: 6.7.0
 Registry Whitelist Mode: disabled.  All registry access allowed.
Plugins:
 Volume: vsphere
 Network: bridge container-net
 Log:
Swarm: inactive
Operating System: linux-x64
OSType: linux-x64
Architecture: x86_64
CPUs: 22800
Total Memory: 30.49GiB
ID: vSphere Integrated Containers
Docker Root Dir:
Debug Mode (client): false
Debug Mode (server): false
Registry: registry.hub.docker.com
Experimental: false
Live Restore Enabled: false
```

```
./vic-machine-darwin inspect \
--target lab-vcsa67-001.lab.jarvis.local/Datacenter-South \
--user adm.jarvis@LAB.JARVIS.LOCAL \
--thumbprint 4F:D3:9B:50:00:31:D9:84:9D:DA:CF:57:21:D6:0D:11:89:78:97:26 \
--name vch01
```

```
INFO[0000] vSphere password for adm.jarvis@LAB.JARVIS.LOCAL:
INFO[0004] ### Inspecting VCH ####
INFO[0005] Validating target
INFO[0005]
INFO[0005] VCH ID: VirtualMachine:vm-982
INFO[0005]
INFO[0005] Installer version: v1.3.1-16055-afdab46
INFO[0005] VCH version: v1.3.1-16055-afdab46
INFO[0005]
INFO[0005] VCH upgrade status:
INFO[0005] Installer has same version as VCH
INFO[0005] No upgrade available with this installer version
WARN[0005] Unable to identify address acceptable to host certificate
INFO[0005]
INFO[0005] VCH Admin Portal:
INFO[0005] https://192.168.100.220:2378
INFO[0005]
INFO[0005] Published ports can be reached at:
INFO[0005] 192.168.100.220
INFO[0005]
INFO[0005] Docker environment variables:
INFO[0005] DOCKER_HOST=192.168.100.220:2376
INFO[0005]
INFO[0005] Connect to docker:
INFO[0005] docker -H 192.168.100.220:2376 --tls info
INFO[0005] Completed successfully
```

```
--name vch-elk \
--container-name-convention elk_{name} \
--syslog-address udp://lab-vrli-001.lab.jarvis.local:514 \
--compute-resource /Datacenter-North \
--image-store lab-ds-001 \
--volume-store lab-ds-001/volumes:default \
--bridge-network vic-bridge \
--bridge-network-range 172.16.0.0/12 \
--public-network vic-public \
--dns-server 192.168.100.80 \
---tls-cname vch-elk \
--organization Jarvis \
--certificate-key-size 2048 \
--no-tlsverify \
--force \
--user adm.jarvis@LAB.JARVIS.LOCAL \
--thumbprint 4F:D3:9B:50:00:31:D9:84:9D:DA:CF:57:21:D6:0D:11:89:78:97:26 \
--target lab-vcsa67-001.lab.jarvis.local/Datacenter-North \
--ops-user jarvis@lab \
--ops-grant-perms \
```

```
root@vic02 [ /etc/vmware/upgrade ]# ./upgrade.sh
-------------------------------
VIC Appliance Upgrade to v1.4.1
-------------------------------

2018-08-16 21:20:30 [=] Values containing $ (dollar sign), ` (backquote), \' (single quote), " (double quote), and \ (backslash) will not besubstituted properly.
2018-08-16 21:20:30 [=] Change any input (passwords) containing these values before running this script.
Enter vCenter Server FQDN or IP: 192.168.178.72
Enter vCenter Administrator Username: admin@jarvis.local
Enter vCenter Administrator Password:
If using an external PSC, enter the FQDN of the PSC instance (leave blank otherwise):
If using an external PSC, enter the PSC Admin Domain (leave blank otherwise):

Please verify the vCenter IP and TLS fingerprint: 192.168.178.72 4F:D3:9B:50:00:31:D9:84:9D:DA:CF:57:21:D6:0D:11:89:78:97:26
Is the fingerprint correct? (y/n): y
Enter vCenter Datacenter of the old VIC appliance: Datacenter-North
Enter old VIC appliance IP: 192.168.100.160
Enter old VIC appliance username: root
2018-08-16 21:21:25 [=]
-------------------------
Starting upgrade 2018-08-16 21:20:30 +0000 UTC

2018-08-16 21:21:26 [=] Please enter the VIC appliance password for root@192.168.100.160
Password:
2018-08-16 21:21:36 [=]
2018-08-16 21:21:36 [=] Detected old appliance's version as v1.3.1.
2018-08-16 21:21:36 [=] If the old appliance's version is not detected correctly, please enter "n" to abort the upgrade and contact VMware support.
2018-08-16 21:21:36 [=]
2018-08-16 21:21:36 [=] Do you wish to proceed with upgrade? [y/n]
y
2018-08-16 21:21:43 [=] Continuing with upgrade
2018-08-16 21:21:43 [=]
2018-08-16 21:21:45 [=] Waiting for old VIC appliance to power off
2018-08-16 21:22:00 [=] Waiting for old VIC appliance to power off
2018-08-16 21:22:18 [=] Migrating old disks to new VIC appliance...
2018-08-16 21:22:19 [=] Copying old data disk. Please wait.
2018-08-16 21:22:25 [=] Copying old database disk. Please wait.
2018-08-16 21:22:32 [=] Copying old log disk. Please wait.
2018-08-16 21:22:41 [=] Finished attaching migrated disks to new VIC appliance
2018-08-16 21:22:41 [=] Preparing upgrade environment
2018-08-16 21:22:41 [=] Disabling and stopping Admiral and Harbor
2018-08-16 21:22:41 [=] Waiting for register appliance...
2018-08-16 21:22:51 [=] Waiting for register appliance...
2018-08-16 21:23:01 [=] Waiting for register appliance...
2018-08-16 21:23:40 [=] Finished preparing upgrade environment
2018-08-16 21:23:40 [=]
-------------------------
Starting Admiral Upgrade 2018-08-16 21:20:30 +0000 UTC

2018-08-16 21:23:40 [=] Performing pre-upgrade checks
2018-08-16 21:23:40 [=] Starting Admiral upgrade
2018-08-16 21:24:28 [=] Updating Admiral configuration
2018-08-16 21:24:28 [=] Restarting Admiral
2018-08-16 21:25:17 [=]
-------------------------
Starting Harbor Upgrade 2018-08-16 21:20:30 +0000 UTC

2018-08-16 21:25:17 [=] Performing pre-upgrade checks
2018-08-16 21:25:17 [=] Starting Harbor upgrade
2018-08-16 21:25:17 [=] [=] Shutting down Harbor
2018-08-16 21:25:17 [=] [=] Migrating Harbor configuration and data
2018-08-16 21:25:17 [=] Testing database credentials...
2018-08-16 21:25:18 [=] Backing up harbor config...
2018-08-16 21:25:23 [=] [=] Successfully migrated Harbor configuration and data
2018-08-16 21:25:23 [=] Harbor upgrade complete
2018-08-16 21:25:23 [=] Starting Harbor
2018-08-16 21:25:33 [=] Enabling and starting Admiral and Harbor
2018-08-16 21:25:33 [=]
2018-08-16 21:25:33 [=] -------------------------
2018-08-16 21:25:33 [=] Upgrade completed successfully. Exiting.
2018-08-16 21:25:33 [=] -------------------------
2018-08-16 21:25:33 [=]
```
