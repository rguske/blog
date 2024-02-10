---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Rebuilding from the Break - Restoring the VMware Avi Controller"
date: 2024-02-10T09:30:58+01:00
draft: true
Description: ""
featuredImage: /img/recover_avi_ctrl_cover.png
categories: ["Platforms"]
tags:
- VMware
- Tanzu
- CloudNative
- Kubernetes
---

## Introduction



{{< image src="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_bad_service.png" caption="Figure I: " src-s="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_bad_service.png" >}}

{{< image src="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_sv_error.png" caption="Figure II: " src-s="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_sv_error.png" >}}

{{< image src="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_av_down.png" caption="Figure III: " src-s="/img/posts/202402_recover_avi_ctrl/202402_recover_avi_ctrl_av_down.png" >}}

```shell
ssh admin@avi-ctrl.jarvis.lab
Avi Cloud Controller

Avi Networks software, Copyright (C) 2013-2017 by Avi Networks, Inc.
All rights reserved.

Management:   10.10.70.10/24      UP
Gateway:      10.10.70.1          UP



admin@avi-ctrl.jarvis.lab's password:


The copyrights to certain works contained in this software are
owned by other third parties and used and distributed under
license. Certain components of this software are licensed under
the GNU General Public License (GPL) version 2.0 or the GNU
Lesser General Public License (LGPL) Version 2.1. A copy of each
such license is available at
http://www.opensource.org/licenses/gpl-2.0.php and
http://www.opensource.org/licenses/lgpl-2.1.php
Last login: Tue Feb  6 16:19:05 2024
```

```shell
$ journalctl

[...]

-- Logs begin at Tue 2024-02-06 18:18:43 UTC, end at Wed 2024-02-07 07:06:22 UTC. --
Feb 06 18:18:43 10-10-70-10 sshd[902]: error: connect_to localhost port 5025: failed.
```

```shell
systemctl list-units --state=failed

  UNIT              LOAD   ACTIVE SUB    DESCRIPTION
● motd-news.service loaded failed failed Message of the Day

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

1 loaded units listed.
```

```shell
systemctl status sshd
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/ssh.service.d
             └─restart.conf
     Active: active (running) since Tue 2024-02-06 16:23:04 UTC; 15h ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 647 (sshd)
      Tasks: 1 (limit: 28829)
     Memory: 16.1M
     CGroup: /system.slice/ssh.service
             └─647 sshd: /usr/sbin/sshd -D [listener] 0 of 50-100 startups

Feb 07 07:04:33 10-10-70-10 sshd[34599]: Connection from 10.10.42.154 port 59594 on 10.10.70.10 port 22 rdomain ""
Feb 07 07:04:35 10-10-70-10 sshd[34599]: Failed publickey for admin from 10.10.42.154 port 59594 ssh2: ED25519 SHA256:puJylWFl7vWE0qA6/Me0mxXyK/ROR/Sj6YnAQyE/afs
Feb 07 07:04:40 10-10-70-10 sshd[34599]: pam_exec(sshd:auth): /opt/avi/scripts/local_login.py failed: exit code 1
Feb 07 07:04:42 10-10-70-10 sshd[34599]: Accepted password for admin from 10.10.42.154 port 59594 ssh2
Feb 07 07:04:42 10-10-70-10 sshd[34599]: pam_unix(sshd:session): session opened for user admin by (uid=0)
Feb 07 07:04:42 10-10-70-10 sshd[34599]: User child is on pid 34635
Warning: journal has been rotated since unit was started, output may be incomplete.
```

`pam_exec(sshd:auth): /opt/avi/scripts/local_login.py failed: exit code 1`


https://avinetworks.com/docs/22.1/backup-and-restore-of-avi-vantage-configuration/

```shell
admin@10-10-70-10:/var/lib/avi/backups$ ls -rtl
total 10904
-rw-r--r-- 1 root root 3040701 Jan 26 11:00 backup_Default-Scheduler_20240126_110041.json
-rw-r--r-- 1 root root 3044239 Feb  1 11:00 backup_Default-Scheduler_20240201_110048.json
-rw-r--r-- 1 root root 2536819 Feb  2 11:00 backup_Default-Scheduler_20240202_110041.json
-rw-r--r-- 1 root root 2533528 Feb  3 11:00 backup_Default-Scheduler_20240203_110044.json
```


```shell
admin@10-10-70-10:/var/lib/avi/backups$ shell
Login: admin
Password:

[...]

The controller at ['https://localhost'] is not active. If this is not the controller address, please restart the shell using the --address flag to specify the address of the controller.
{"error": "Bad service"}

Wrong username and password combination. Exiting now; please try again later.
```

```shell
admin@10-10-70-10:/var/lib/avi/backups$ shell --address 10.10.70.10
Login: admin
Password:

The controller at ['https://10.10.70.10'] is not active. If this is not the controller address, please restart the shell using the --address flag to specify the address of the controller.
{"error": "Bad service"}
```

```shell
admin@10-10-70-10:/var/lib/avi/backups$ sudo docker run -it -v /tmp:/tmp ubuntu

[sudo] password for admin:
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
57c139bbda7e: Pull complete
Digest: sha256:e9569c25505f33ff72e88b2990887c9dcf230f23259da296eb814fc2b41af999
Status: Downloaded newer image for ubuntu:latest
root@9212247dfc35:/# sudo apt-get update
bash: sudo: command not found
root@9212247dfc35:/# apt-get update
Get:1 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
Get:3 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [1064 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [109 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy/universe amd64 Packages [17.5 MB]
Get:7 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [1401 kB]
Get:8 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 Packages [44.6 kB]
Get:9 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [1691 kB]
Get:10 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages [1792 kB]
Get:11 http://archive.ubuntu.com/ubuntu jammy/multiverse amd64 Packages [266 kB]
Get:12 http://archive.ubuntu.com/ubuntu jammy/restricted amd64 Packages [164 kB]
Get:13 http://archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 Packages [50.4 kB]
Get:14 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [1334 kB]
Get:15 http://archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [1728 kB]
Get:16 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1680 kB]
Get:17 http://archive.ubuntu.com/ubuntu jammy-backports/universe amd64 Packages [28.1 kB]
Get:18 http://archive.ubuntu.com/ubuntu jammy-backports/main amd64 Packages [50.4 kB]
Fetched 29.4 MB in 4s (6946 kB/s)
Reading package lists... Done
root@9212247dfc35:/#
```

```shell
apt-get install -y python3 python-pip
```

```shell
pip install virtualenv
```

https://avinetworks.com/docs/latest/cli-installing-the-cli-shell/

https://portal.avipulse.vmware.com/software/additional-tools



```shell
scp avi_shell.tar.gz admin@10.10.70.10:/home/admin
```

```shell
sudo docker ps
[sudo] password for admin:
CONTAINER ID   IMAGE     COMMAND       CREATED             STATUS             PORTS     NAMES
9212247dfc35   ubuntu    "/bin/bash"   About an hour ago   Up About an hour             crazy_einstein
```

```shell
sudo docker cp ./avi_shell.tar.gz crazy_einstein:/home
```

```shell
root@9212247dfc35:/home#  virtualenv avi_shell
created virtual environment CPython3.10.12.final.0-64 in 779ms
  creator CPython3Posix(dest=/home/avi_shell, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
    added seed packages: pip==23.3.1, setuptools==69.0.2, wheel==0.42.0
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
root@9212247dfc35:/home#
```

```shell
root@9212247dfc35:/home# cd avi_shell
root@9212247dfc35:/home/avi_shell# ls
bin  lib  pyvenv.cfg
root@9212247dfc35:/home/avi_shell#
```

```shell
root@9212247dfc35:/home/avi_shell# source ./bin/activate
(avi_shell) root@9212247dfc35:/home/avi_shell#
```


```shell
sudo tar -czf backups.tar *
```

```shell
scp admin@10.10.70.10:/var/lib/avi/backups/backups.tar ~/Downloads/avibackup
```

```shell
admin@10-10-70-10:/opt/avi/scripts$ sudo ./flushdb.sh
rm: cannot remove '/etc/luna/cert/server/*': No such file or directory
rm: cannot remove '/etc/luna/cert/client/*': No such file or directory
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
Creating SSH2 RSA key; this may take some time ...
3072 SHA256:OZDiQ2SqCXaWR5+Dd1PLc95ekDY6Ix5el0JPAHoc9Ck root@10-10-70-10 (RSA)
rescue-ssh.target is a disabled or a static unit, not starting it.


https://avinetworks.com/docs/22.1/backup-and-restore-of-avi-vantage-configuration/



sudo /opt/avi/scripts/restore_config.py --config /var/lib/avi/backups/backup_Default-Scheduler_20240203_110044.json --passphrase 'VMware1!' --flushdb

Traceback (most recent call last):
  File "/opt/avi/scripts/restore_config.py", line 98, in <module>
    user, token = get_username_token()
  File "/opt/avi/python/lib/avi/util/login.py", line 136, in get_username_token
    rsp = json.loads(response.stdout)
  File "/usr/lib/python3.8/json/__init__.py", line 357, in loads
    return _default_decoder.decode(s)
  File "/usr/lib/python3.8/json/decoder.py", line 340, in decode
    raise JSONDecodeError("Extra data", s, end)
json.decoder.JSONDecodeError: Extra data: line 1 column 9 (char 8)

restore_config.py on node (10-10-70-10) crash -- Pls investigate further.
core_archive.20240207_152903.tar.gz not collected due to similar stack-trace se
en=restore_config.py.20240207_152728.stack_trace


sudo /opt/avi/scripts/restore_config.py --config /tmp/backup_Default-Scheduler_20240203_110044.json --passphrase VMware1!
[sudo] password for admin:
Restore config must be run on a clean controller.
Please run with the --flushdb option or do a reboot clean to clear all config before restoring.

 sudo /opt/avi/scripts/restore_config.py --config /tmp/backup_Default-Scheduler_20240203_110044.json --passphrase VMware1! --flushdb
Restoring config will duplicate the environment of the config. If the environment is active elsewhere there will be conflicts.
                Please confirm if you still want to continue (yes/no)yes



======== Configuration Restored for 1-node cluster ========


[admin:10-10-70-10]: > show backupconfiguration Backup-Configuration
+-------------------------------+----------------------------------------------------------+
| Field                         | Value                                                    |
+-------------------------------+----------------------------------------------------------+
| uuid                          | backupconfiguration-34c64416-83d8-4f56-9775-f66489fe80ce |
| name                          | Backup-Configuration                                     |
| save_local                    | True                                                     |
| maximum_backups_stored        | 4                                                        |
| backup_passphrase             | <sensitive>                                              |
| remote_file_transfer_protocol | SCP                                                      |
| tenant_ref                    | admin                                                    |
+-------------------------------+----------------------------------------------------------+

https://avinetworks.com/docs/22.1/reset-to-factory-defaults/

[admin:10-10-70-10]: > show cluster
+-----------------+----------------------------------------------+
| Field           | Value                                        |
+-----------------+----------------------------------------------+
| uuid            | cluster-98e844bc-7e29-4d86-a927-88fd843940d1 |
| name            | cluster-0-1                                  |
| nodes[1]        |                                              |
|   name          | 10.10.70.10                                  |
|   ip            | 10.10.70.10                                  |
|   vm_uuid       | 00505687db02                                 |
|   vm_mor        | vm-4308                                      |
|   vm_hostname   | node1.controller.local                       |
|   interfaces[1] |                                              |
|     if_name     | eth0                                         |
|     mac_address | 00:50:56:87:db:02                            |
|     mode        | STATIC                                       |
|     ip          | 10.10.70.10/24                               |
|     gateway     | 10.10.70.1                                   |
|     labels[1]   | MGMT                                         |
|     labels[2]   | SE_SECURE_CHANNEL                            |
|     labels[3]   | HSM                                          |
+-----------------+----------------------------------------------+


{{< admonition info "" true >}}

{{< /admonition >}}