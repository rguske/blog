---
author: "Robert Guske"
authorLink: "/about/"
lightgallery: true
title: "Unable to Install Ovftool on Ubuntu 20"
date: 2021-03-09T14:33:34+01:00
draft: true
featuredImage: /img/projectnautilus_cover.jpg
categories: ["cat1", "cat2"]
tags:
- tag2
- tag2
---
## Introduction


[OVFTOOL 4.4.1](https://my.vmware.com/group/vmware/downloads/get-download?downloadGroup=OVFTOOL441)


uname -a 
Linux vdesk-ubuntu 5.8.0-43-generic #49~20.04.1-Ubuntu SMP Fri Feb 5 09:57:56 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux


Ubu  rguske  ~/Downloads                                                                                                                                  kubernetes-admin@kubernetes/default ⎈  64% hdd  1.81G RAM 
╰─ tree -L 1
.
├── avi
├── terraform-hacknite-lab1
├── vCenter_Event_Broker_Appliance_v0.6.0.ova
├── VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle



chmod +755 VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle 


Options:
    --gtk               Use the Gtk+ UI (Default)
    --console           Use the console UI
    --required          Displays only questions absolutely required
    --eulas-agreed      Agree to the EULA

    --ignore-errors

    --extract=DIR       Extract the contents of the bundle into DIR


sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --console --required --eulas-agreed

```
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


sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --gtk --required --eulas-agreed

```
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



sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --extract vmware-ovftool

```
Extracting VMware Installer...done.

(vmware-installer.py:234242): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(vmware-installer.py:234242): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",
Gtk-Message: Failed to load module "canberra-gtk-module"
```



~/Downloads/vmware-ovftool                                                                                                                   kubernetes-admin@kubernetes/default ⎈  64% hdd  1.79G RAM 
╰─ tree -L 2
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




ls -rtl

total 16K
drwxr-xr-x 4 root   root         4,0K Mär  9 14:07 .
drwxr-xr-x 5 rguske domain^users 4,0K Mär  9 14:07 ..
drwxr-xr-x 9 root   root         4,0K Mär  9 14:07 vmware-installer
drwxr-xr-x 6 root   root         4,0K Mär  9 14:07 vmware-ovftool


cd vmware-ovftool


sudo chmod +755 ovftool ovftool.bin


WORKS!!


sudo mv vmware-ovftool /usr/bin



sudo ln -s vmware-ovftool/ovftool ovftool


which ovftool
/usr/bin/ovftool






sudo ./VMware-ovftool-4.4.0-16360108-lin.x86_64.bundle --console --required --eulas-agreed

Extracting VMware Installer...done.
Traceback (most recent call last):
  File "/tmp/tmpf4xg4kbw.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.




less /var/log/vmware-installer

[2021-02-25 11:25:18,340] Installer running.
[2021-02-25 11:25:18,340] Command Line Arguments:
[2021-02-25 11:25:18,340] ['/usr/lib/vmware-installer/3.0.0/vmware-installer.py', '--set-setting', 'vmware-installer', 'libconf', '', '--install-component', '/tmp/vmis.Xh1rd2/install/vmware-installer', '--install-bundle', '/home/rguske/Downloads/./VMware-ovftool-4.4.0-16360108-lin.x86_64.bundle', '', '--console', '--required', '--eulas-agreed']
[2021-02-25 11:25:18,345] Using UI type console
[2021-02-25 11:25:18,347] System installer version is: 3.0.0.17287072
[2021-02-25 11:25:18,347] Running installer version is: 3.0.0.17287072
[2021-02-25 11:25:18,347] Opening database file /etc/vmware-installer/database
[2021-02-25 11:25:18,436] Could not locate installer App Control.
[2021-02-25 11:25:18,453] Error: Cannot load installer for component: vmware-installer.
[2021-02-25 11:25:18,454] Top level exception handler
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



sudo ./VMware-ovftool-4.4.0-16360108-lin.x86_64.bundle                                    
Extracting VMware Installer...done.

(vmware-installer.py:96561): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(vmware-installer.py:96561): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",
Gtk-Message: Failed to load module "canberra-gtk-module"
Traceback (most recent call last):
  File "/tmp/tmpijzp_ddd.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.


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

sudo apt search canberra-gtk-module 
Sorting... Done
Full Text Search... Done
libcanberra-gtk-module/focal 0.30-7ubuntu1 amd64
  translates GTK+ widgets signals to event sounds


sudo apt-get install libcanberra-gtk-module

sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --console --required --eulas-agreed

Extracting VMware Installer...done.
Traceback (most recent call last):
  File "/tmp/tmpcg3dtkug.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.

sudo apt-get install murrine-themes

[######################################################################] 100%
Installation was unsuccessful.



https://communities.vmware.com/t5/Open-Virtualization-Format-Tool/Error-Installing-OVFTool-on-Ubuntu-14-WorkStation-Guest/td-p/2232061

> I'd been having this problem on multiple versions of Ubuntu for quite some time too, but found the answer in an post regarding a similar issue with a different VMware product - it has to do with how the VMware installer tries to use the Curses library.  Try something like this to see if it fixes your issue:

TERM=dumb /bin/sh ./VMware-ovftool-4.2.0-4586971-lin.x86_64.bundle


sudo ./VMware-ovftool-4.4.0-16530037-lin.x86_64.bundle --extract ./vmware-ovftool



sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --gtk --required --eulas-agreed
Extracting VMware Installer...done.

(vmware-installer.py:147284): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(vmware-installer.py:147284): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",
Gtk-Message: Failed to load module "canberra-gtk-module"
Traceback (most recent call last):
  File "/tmp/tmpihijvlfg.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.






echo $GTK_PATH


pwd    
/usr/lib/x86_64-linux-gnu/gtk-2.0


export GTK_PATH=/usr/lib/x86_64-linux-gnu/gtk-2.0

 sudo ./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --gtk --required --eulas-agreed
Extracting VMware Installer...done.

(vmware-installer.py:149037): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(vmware-installer.py:149037): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",
Gtk-Message: Failed to load module "canberra-gtk-module"
Traceback (most recent call last):
  File "/tmp/tmpbbv6c1ad.vmis.env", line 132, in <module>
    exec(fileObj.read(), mod.__dict__)
  File "<string>", line 243
    except OSError, e:
                  ^
SyntaxError: invalid syntax
Exception info : [Component did not register an installer].
Check the log /var/log/vmware-installer for details.



[######################################################################] 100%
Installation was unsuccessful.





./VMware-ovftool-4.4.1-16812187-lin.x86_64.bundle --extract ./vmware-ovftool
Extracting VMware Installer...done.
Gtk-Message: Failed to load module "atk-bridge"




sudo apt-get install libatk-adaptor libgail-common                     
Reading package lists... Done
Building dependency tree       
Reading state information... Done
libgail-common is already the newest version (2.24.32-4ubuntu4).
libgail-common set to manually installed.
libatk-adaptor is already the newest version (2.34.2-0ubuntu2~20.04.1).
libatk-adaptor set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 26 not upgraded.


```
ls -rtl





{{< image src="/img/posts//" caption="Figure I: " src-s="/img/posts//" >}}


{{< admonition info "" true >}}

{{< /admonition >}}



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