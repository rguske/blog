---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Unable to Install Ovftool on Ubuntu 20.04 LTS"
date: 2021-03-09T14:33:34+01:00
draft: true
featuredImage: /img/ovftool_cover.jpg
categories: ["Workaround", "Linux", "Desktop", "VMware"]
tags:
- March2021
- Linux
- Desktop
- Workaround
- VMware
---

## Introduction

The VMware OVF Tool [^1] is a powerful cli utility with which you can import and export Open Virtualization Format (OVF) packages to and from various VMware products. It's e.g. used for the creation of several great VMware Fling projects like the Demo Appliance for Tanzu Kubernetes Grid [^2], the VMware Appliance for Folding@Home [^3], the VMware Event Broker Appliance [^4] as well as for non-fling open source projects like e.g. the Netshoot Virtual Appliance [^5]. The Fling projects listed have one thing in particular in common and that is the main contributor - William Lam. If you weren't aware of what the tool is capable of yet, then check out William's work with it on his blog [^6].

I recently wanted to use the `ovftool` for validating a customer use-case and needed to install it on my "*Universal Workbench*" [^7], my Ubuntu 20.04 vDesk, which I am using for all my homelab work. As of writing this article, the latest available version for the ovftool is version 4.4.1 [^8].

## Installation Options

So, I downloaded the appropriate 64 bit binary for my Linux system, which is the `VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle` file and made it executable:

```shell
$ chmod +755 VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle
```

Various options are available for the installation itself (see: `ovftool --help`). The following listed are an abstract of the ones which are relevant for this post and which I've used for my installation tries.

```shell
Options:
    --gtk               Use the Gtk+ UI (Default)
    --console           Use the console UI
    --required          Displays only questions absolutely required
    --eulas-agreed      Agree to the EULA
    --ignore-errors
    --extract=DIR       Extract the contents of the bundle into DIR
```

## Installation was unsuccessful

Basically, you can decide if you'd like to use `--console` UI or the `--gtk` UI. My first attempt, and I used this option several installations before, was with the `--console` option. In a Terminal window I executed `sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --console --required --eulas-agreed` and got the following output as a failed result:

```shell
Extracting VMware Installer...done.
Traceback (most recent call last):
  File "/tmp/tmpt0pd6m09.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.
```

<mark>Installation was unsuccessful</mark>

I couldn't really figure out the output and went straight for the `--gtk` option as my 2nd attempt:

```shell
$ sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --gtk --required --eulas-agreed

Extracting VMware Installer...done.

(vmware-installer.py:233744): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(vmware-installer.py:233744): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",
Gtk-Message: Failed to load module "canberra-gtk-module"
Traceback (most recent call last):
  File "/tmp/tmp1nqoz5ld.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.
```

<mark>Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita"</mark>

<mark>Gtk-Message: Failed to load module "canberra-gtk-module"</mark>

Some research on "`Unable to locate theme engine in module_path: "adwaita"`" pointed me to presumable missing packages for my vDesk and therefore I followed some of the solution results which were for example:

```
$ sudo apt install gnome-themes-standard

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  gnome-themes-standard
0 upgraded, 1 newly installed, 0 to remove and 2 not upgraded.
Need to get 2.164 B of archives.
After this operation, 14,3 kB of additional disk space will be used.
Get:1 http://de.archive.ubuntu.com/ubuntu focal/universe amd64 gnome-themes-standard all 3.28-1ubuntu1 [2.164 B]
Fetched 2.164 B in 0s (10,8 kB/s)
Selecting previously unselected package gnome-themes-standard.
(Reading database ... 183276 files and directories currently installed.)
Preparing to unpack .../gnome-themes-standard_3.28-1ubuntu1_all.deb ...
Unpacking gnome-themes-standard (3.28-1ubuntu1) ...
Setting up gnome-themes-standard (3.28-1ubuntu1) ...
```

or

```shell
$ sudo apt search canberra-gtk-module

Sorting... Done
Full Text Search... Done
libcanberra-gtk-module/focal 0.30-7ubuntu1 amd64
  translates GTK+ widgets signals to event sounds
```

as well as `sudo apt-get install libcanberra-gtk-module` but nothing solved the problem nor changed the failure message. Also the `/var/log/vmware-installer` log couldn't help.

### Output /var/log/vmware-installer

```shell
$ less /var/log/vmware-installer


[2021-02-25 11:28:13,465] code for hash sha256 was not found.
Traceback (most recent call last):
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 147, in <module>
    globals()[__func_name] = __get_hash(__func_name)
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 97, in __get_builtin_constructor
    raise ValueError('unsupported hash type ' + name)
ValueError: unsupported hash type sha256
[2021-02-25 11:28:13,465] code for hash sha384 was not found.
Traceback (most recent call last):
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 147, in <module>
    globals()[__func_name] = __get_hash(__func_name)
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 97, in __get_builtin_constructor
    raise ValueError('unsupported hash type ' + name)
ValueError: unsupported hash type sha384
[2021-02-25 11:28:13,465] code for hash sha512 was not found.
Traceback (most recent call last):
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 147, in <module>
    globals()[__func_name] = __get_hash(__func_name)
  File "/tmp/vmis.Lu1nSs/install/vmware-installer/python/lib/hashlib.py", line 97, in __get_builtin_constructor
    raise ValueError('unsupported hash type ' + name)
ValueError: unsupported hash type sha512
[2021-02-25 11:28:13,494]
[2021-02-25 11:28:13,494]
[2021-02-25 11:28:13,494] Installer running.
[2021-02-25 11:28:13,494] Command Line Arguments:
[2021-02-25 11:28:13,494] ['/tmp/vmis.Lu1nSs/install/vmware-installer/vmware-installer.py', '--set-setting', 'vmware-installer', 'libconf', '', '--install-component', '/tmp/vmis.Lu1nSs/install/vmware-installer', '--install-bundle', '/home/rguske/Downloads/./VMware-ovftool-4.4.0-16360108-lin.x86_64.bundle', '']
[2021-02-25 11:28:13,740] Successfully loaded GTK libraries.
[2021-02-25 11:28:13,789] Using UI type gtk
[2021-02-25 11:28:13,795] System installer version is: 3.0.0.17287072
[2021-02-25 11:28:13,795] Running installer version is: 2.1.0.14928104
[2021-02-25 11:28:13,795] Running newer system installer.
[2021-02-25 11:28:14,151]
[2021-02-25 11:28:14,151]
[2021-02-25 11:28:14,151] Installer running.
[2021-02-25 11:28:14,151] Command Line Arguments:
[2021-02-25 11:28:14,151] ['/usr/lib/vmware-installer/3.0.0/vmware-installer.py', '--set-setting', 'vmware-installer', 'libconf', '', '--install-component', '/tmp/vmis.Lu1nSs/install/vmware-installer', '--install-bundle', '/home/rguske/Downloads/./VMware-ovftool-4.4.0-16360108-lin.x86_64.bundle', '']
[2021-02-25 11:28:14,151] Failed to load GTK libraries, falling back to console.
[2021-02-25 11:28:14,157] Using UI type console
[2021-02-25 11:28:14,158] System installer version is: 3.0.0.17287072
[2021-02-25 11:28:14,158] Running installer version is: 3.0.0.17287072
[2021-02-25 11:28:14,158] Opening database file /etc/vmware-installer/database
[2021-02-25 11:28:14,236] Could not locate installer App Control.
[2021-02-25 11:28:14,247] Error: Cannot load installer for component: vmware-installer.
[2021-02-25 11:28:14,248] Top level exception handler
Traceback (most recent call last):
  File "/usr/lib/vmware-installer/3.0.0/vmis/core/transaction.py", line 470, in RunThreadedTransaction
    txn.Execute(actions)
  File "/usr/lib/vmware-installer/3.0.0/vmis/core/transaction.py", line 252, in Execute
    i.Load(self.temp)
  File "/usr/lib/vmware-installer/3.0.0/vmis/core/install.py", line 325, in Load
    self._module, self._installer = LoadInstaller(self.component,
  File "/usr/lib/vmware-installer/3.0.0/vmis/core/env.py", line 406, in LoadInstaller
    raise ComponentError('Component did not register an installer', component)
vmis.core.errors.ComponentError: Component did not register an installer
```


## Workaround: Extract ovftool



`sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --extract ovftool && cd ovftool`

```shell
ls -rtl

total 16K
drwxr-xr-x 4 root   root         4,0K Mär  9 14:07 .
drwxr-xr-x 5 rguske domain^users 4,0K Mär  9 14:07 ..
drwxr-xr-x 9 root   root         4,0K Mär  9 14:07 vmware-installer
drwxr-xr-x 6 root   root         4,0K Mär  9 14:07 vmware-ovftool
```


```shell
$ tree -L 2
.
├── vmware-installer
│   ├── artwork
│   ├── bin
│   ├── bootstrap
│   ├── lib
│   ├── manifest.xml
│   ├── python
│   ├── sopython
│   ├── vmis
│   ├── vmis-launcher
│   ├── vmware-installer
│   ├── vmware-installer.py
│   ├── vmware-uninstall
│   └── vmware-uninstall-downgrade
└── vmware-ovftool
    ├── certs
    ├── env
    ├── icudt44l.dat
    ├── libcares.so.2
    ├── libcrypto.so.1.0.2
    ├── libcurl.so.4
    ├── libexpat.so
    ├── libgcc_s.so.1
    ├── libgoogleurl.so.59
    ├── libicudata.so.60
    ├── libicuuc.so.60
    ├── libssl.so.1.0.2
    ├── libssoclient.so
    ├── libstdc++.so.6
    ├── libvim-types.so
    ├── libvmacore.so
    ├── libvmomi.so
    ├── libxerces-c-3.2.so
    ├── libz.so.1
    ├── manifest.xml
    ├── open_source_licenses.txt
    ├── ovftool
    ├── ovftool.bin
    ├── README.txt
    ├── schemas
    ├── vmware.eula
    └── vmware-eula.rtf

11 directories, 31 files
```

```shell
sudo mv vmware-ovftool /usr/bin/ovftool
```

```shell
sudo chmod +x /usr/bin/ovftool/ovftool ovftool.bin
```

WORKS!!



`sudo ln -s vmware-ovftool/ovftool ovftool`

```shell
$ which ovftool

/usr/bin/ovftool
```



[^1]: [VMware ovftool {code} page](https://code.vmware.com/web/tool/4.4.0/ovf)
[^2]: [Demo Appliance for Tanzu Kubernetes Grid](https://flings.vmware.com/demo-appliance-for-tanzu-kubernetes-grid)
[^3]: [VMware Appliance for Folding@Home](https://flings.vmware.com/vmware-appliance-for-folding-home)
[^4]: [VMware Event Broker Appliance](https://vmweventbroker.io/kb/contribute-appliance)
[^5]: [Netshoot Virtual Appliance](https://github.com/josemzr/netshoot-virtual-appliance)
[^6]: [virtuallyGhetto](https://www.virtuallyghetto.com/ovf)
[^7]: [A Linux Development Desktop with VMware Horizon - Part I: Horizon](https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-i-horizon/)
[^8]: [Download OVFTOOL 4.4.1](https://my.vmware.com/group/vmware/downloads/get-download?downloadGroup=OVFTOOL441)
