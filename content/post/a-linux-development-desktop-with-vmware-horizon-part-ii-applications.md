---
title: "A Linux Development Desktop with VMware Horizon - Part II: Applications "
date: 2020-02-27T16:33:48+01:00
draft: false
image: /img/devdesk_part_ii_cover.jpg
thumbnail: /img/devdesk_part_ii_thumbnail.jpg
tags:
- January2020
- Linux
- Horizon
- VDI
- Tools
---

**Previous articles:**

- <a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-i-horizon/">*A Linux Development Desktop with VMware Horizon - Part I: Horizon*</a>

# Applications

In this section I´m going to list a couple of applications which I´m using for my DevDesk and how you can easily install them from your shell. Of course, it´s not a must and it´s up to you which of them you´d like to install.

## Google Chrome - https://www.google.com/chrome/

Firefox is pre-installed on Ubuntu as well as on CentOS but I´m for years now with Chrome and I´m still satisfied.

<span style="color:#018914">**Ubuntu:**</span>

```shell
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb -P ~/Downloads
sudo dpkg -i ~/Downloads/google-chrome-stable_current_amd64.deb
```

<span style="color:#6003B6">**CentOS:**</span>

```shell
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm -P ~/Downloads
sudo yum install ~/Downloads/google-chrome-stable_current_x86_64.rpm
```

---

## Visual Studio Code (VSCode) - https://code.visualstudio.com/

One of my absolute favorite tools so far and indispensable for our DevDesk.

<span style="color:#018914">**Ubuntu:**</span>

**Repository and key:**

```shell
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
```

**Install:**

```shell
sudo apt-get install apt-transport-https && sudo apt-get update && sudo apt-get install code
```

<span style="color:#6003B6">**CentOS:**</span>

**Repository and key:**

```shell
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

**Install:**

```shell
sudo dnf check-update
sudo dnf install code
```

## [Extension] VSCode Settings sync

Link: https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync

VSCode has a REALLY rich extensibility model which gives you everything you need to work even more efficiently and faster. It´ll not be possible to avoid that more and more extensions and other niceties are added to your installation and as of today it´s not possible to have everything consistent across different systems or platforms through an Online-Account like you have e.g. for Google Chrome.

Thus I´d like to recommend using the *VSCode Settings sync* extension for it. It uses your GitHub account (required) token and Gist, to give you the ability to upload as well as download your Settings, Snippets, Themes and so forth.

Just search for the extension at the marketplace, install it, select LOGIN WITH GITHUB (get redirected) and follow the instructions. Since you´ve created your Gist-ID you´re ready to go to upload as well as to download your settings.

**Shortcuts**

1. Upload Key : Shift + Alt + U
2. Download Key : Shift + Alt + D

(on macOS: Shift + Option + U / Shift + Option + D)

**Set VSCode as default editor for Git**

Open your terminal and execut: `git config --global core.editor “code --wait”`

*Source: https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration*

---

## Docker Engine - Community

Another must!

<span style="color:#018914">**Ubuntu:**</span>

```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

<span style="color:#6003B6">**CentOS:**</span>

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm

sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli

sudo systemctl enable --now docker

systemctl status docker.service

sudo usermod -aG docker $USER
```

After adding your user to the docker group, you have to logout and login first, so that your group membership is re-evaluated.

## Docker Compose - https://docs.docker.com/compose/

A tool for deploying multi-container applications which services are specified in a YAML file.

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

---

## VMware Remote Console

Download: https://www.vmware.com/go/download-vmrc

## VMware OVF Tool

Download: https://code.vmware.com/web/tool/4.3.0/ovf

These two binaries were downloaded locally (to my Notebook) first, then got packed into a zip archive and copied via `scp` to my DevDesk(s).

```shell
scp ~/Downloads/tools.zip jarvis@devdesk-centos:/home/jarvis/Downloads
```

Unpack afterwards:

```shell
unzip tools.zip
```

**Note:** Installing the OVF Tool for <span style="color:#6003B6">**CentOS**</span> requires the installation of the *ncurses-compat-libs* package first.

```shell
sudo dnf install ncurses-compat-libs
```

Make them executable and start the installation:

```shell
chmod +x VMware-ovftool-4.3.0-14746126-lin.x86_64.bundle
sudo ./VMware-ovftool-4.3.0-14746126-lin.x86_64.bundle --console --required --eulas-agreed
```

```shell
chmod +x VMware-Remote-Console-11.0.0-15201582.x86_64.bundle
sudo ./VMware-Remote-Console-11.0.0-15201582.x86_64.bundle --console --required --eulas-agreed
```


---

## Enpass - https://www.enpass.io/

A secure vault which is available for all mobile and desktop platforms.

<span style="color:#018914">**Ubuntu:**</span>

```shell
sudo -i
echo "deb https://apt.enpass.io/ stable main" > \/etc/apt/sources.list.d/enpass.list
wget -O - https://apt.enpass.io/keys/enpass-linux.key | apt-key add -
exit
sudo apt update && sudo apt install enpass
```

<span style="color:#6003B6">**CentOS:**</span>

```shell
cd /etc/yum.repos.d/
sudo wget https://yum.enpass.io/enpass-yum.repo
sudo yum install enpass
cd ~
```

---

**KinD** - https://github.com/kubernetes-sigs/kind

> kind is a tool for running local Kubernetes clusters using Docker container "nodes".

<center><a href="/img/posts/201912_development_desktop/CapturFiles-20200227_015150.jpg"><img src="/img/posts/201912_development_desktop/CapturFiles-20200227_015150.jpg"></img></a></center>

<center>*Figure I: kind create cluster*</center>

**Octant** - https://github.com/vmware-tanzu/octant

> Octant is a tool for developers to understand how applications run on a Kubernetes cluster.

![Octant demo](/img/posts/201912_development_desktop/octant-demo.gif)

<center>*Source: https://github.com/vmware-tanzu/octant*</center>

---

**Continue with Part III:** <a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-iii-shell/">A Linux Development Desktop with VMware Horizon - Part III: High Way to Shell</a>

---

**Previous articles:**

<a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-i-horizon/">A Linux Development Desktop with VMware Horizon - Part I: Horizon</a>

---

## <center>**Thanks for reading.**</center>