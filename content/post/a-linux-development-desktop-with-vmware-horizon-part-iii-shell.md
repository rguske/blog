---
title: "A Linux Development Desktop with VMware Horizon - Part III: Shell"
date: 2020-02-27T16:32:48+01:00
draft: false
image: /img/devdesk_part_iii_cover.jpg
thumbnail: /img/devdesk_part_iii_thumbnail.jpg
tags:
- February2020
- Linux
- Horizon
- VDI
- Tools
- Shell
---

**Previous articles:**

- <a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-i-horizon/">*A Linux Development Desktop with VMware Horizon - Part I: Horizon*</a>
- <a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-ii-applications/">*A Linux Development Desktop with VMware Horizon - Part II: Applications*</a>

# High :zap: Way to Shell

The operationalization of platforms such as e.g. Kubernetes, or the use of tools for building services or applications such as Docker, requires in both cases the use of the command-line. There are dozens of great plugins, themes and extensions out there to pimp your shell so that it´ll help you to increase velocity as well as useability. 

**Part III** of my series is focused on the Shell and which *"Highway"* you can take to have the described tool available at the end.

<center><a href="/img/posts/201912_development_desktop/CapturFiles-20200224_100043.jpg"><img src="/img/posts/201912_development_desktop/CapturFiles-20200224_100043.jpg"></img></a></center>

<center>*Figure I: Default Terminal appearance before tuning*</center>

## 1. ZSH aka Z shell - http://www.zsh.org/

ZSH is a extended shell with lots of features, support for plugins and themes.

<span style="color:#018914">**Ubuntu:**</span>

```shell
sudo apt install zsh
```

<span style="color:#6003B6">**CentOS:**</span>

Should be installed by default.

```shell
sudo yum install zsh
```

## 2. Oh My ZSH - https://ohmyz.sh/

A community-driven framework for managing your Z shell configuration.
<span style="color:#018914">**Ubuntu:**</span>

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

<span style="color:#6003B6">**CentOS:**</span>

```shell
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Verify if ZSH is set as your new default Shell:

```shell
echo $SHELL
cat /etc/passwd | grep "username"
```

## 3. Install Powerline Fonts - https://github.com/powerline/fonts

Needed fonts for the Powerlevel9k theme (step 4.).

<span style="color:#018914">**Ubuntu:**</span>

```shell
sudo apt install fonts-powerline
```

<span style="color:#6003B6">**CentOS:**</span>

```shell
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
cd ..
rm -rf fonts
```

## 4. Powerlevel-9k - https://github.com/Powerlevel9k/powerlevel9k

This theme will give your shell a new shine.

<span style="color:#6003B6">**CentOS**</span> & <span style="color:#018914">**Ubuntu:**</span>

```shell
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
vim ~/.zshrc
```

Replace the default ZSH_THEME with the new Powerlevel-9k theme:

```shell
ZSH_THEME="powerlevel9k/powerlevel9k"
```

And you should also paste the following variable to the top of your *.zshrc* file, otherwise we will get an error when we use `tmux` later.

```shell
export TERM="xterm-256color"
```

A logout or reboot of the system is necessary for the changes to take effect.

**The result:**

<center><a href="/img/posts/201912_development_desktop/CapturFiles-20200226_112728.jpg"><img src="/img/posts/201912_development_desktop/CapturFiles-20200226_112728.jpg"></img></a></center>

<center>*Figure II: ZSH with Powerlevel-9k Theme | l: Ubuntu r: CentOS*</center>

**I love it!** And it doesn´t only looks good, it also comes in very handy if you work e.g. with `Git` as you can see on *Figure II* for example. The right screenshot shows you that I am in a git branch called *featurebranch* and the color indicates me, that my workingtree is either clean and theres nothing to commit (green) or that new files were added and needs to be commited (orange).

---
**More! More! More!**

## Homebrew on Linux :beer: - https://docs.brew.sh/Homebrew-on-Linux

Homebrew is a package manager which simplifies the installation of software on your OS and should not be missing if you ask me.

<span style="color:#6003B6">**CentOS**</span> & <span style="color:#018914">**Ubuntu:**</span>

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"
```

The installation-script will throw a **Warning** at the end, because it points out that `/home/linuxbrew/.linuxbrew/bin` isn´t in your PATH.
Add the path to your .zshrc dot-file.

```shell
vim ~/.zshrc
export PATH=$PATH:"/home/linuxbrew/.linuxbrew/bin"
source ~/.zshrc
```

---

## CLI´s and tools I´m using and I´d like to recommend

**kubectl** - https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux

> A command line tool for controlling Kubernetes clusters.

```shell
brew install kubectl
```

**ZSH Autocompletion** for `kubectl` - https://v1-16.docs.kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete

```shell
source <(kubectl completion zsh)
echo "if [ $commands[kubectl] ]; then source <(kubectl completion zsh); fi" >> ~/.zshrc
```

**tmux** - https://github.com/tmux/tmux

> `tmux` is a terminal multiplexer.

**bat** - https://github.com/sharkdp/bat

> `bat` is a cat clone with syntax highlighting and Git integration.

**htop** - https://github.com/hishamhm/htop

> `htop` is an interactive process viewer.

**ctop** - https://github.com/bcicen/ctop

> `ctop` provides a concise and condensed overview of real-time metrics for multiple containers:

**prettyping** - https://github.com/denilsonsa/prettyping

> `prettyping` is a wrapper around the standard ping tool with the objective of making the output prettier, more colorful, more compact, and easier to read.

And all of them can be easily installed via `brew`:
```shell
brew install tmux bat htop ctop prettyping
```

Addtionally to `htop`, `ytop` (former gotop) is a pretty neat process and system monitor.

`ytop` - https://github.com/cjbassi/ytop

```shell
brew tap cjbassi/ytop && brew install ytop
```

<center><a href="/img/posts/201912_development_desktop/CapturFiles-20200217_050729.jpg"><img src="/img/posts/201912_development_desktop/CapturFiles-20200217_050729.jpg"></img></a></center>

<center>*Figure III: tmux, htop, ctop, prettyping & ytop*</center>

**figlet** - http://www.figlet.org/ & **lolcat** - https://github.com/busyloop/lolcat

```shell
brew install figlet lolcat

figlet cliFun | lolcat
```

```shell
      _ _ _____
  ___| (_)  ___|   _ _ __
 / __| | | |_ | | | | '_ \
| (__| | |  _|| |_| | | | |
 \___|_|_|_|   \__,_|_| |_|
```

---
**cowsay** -
You should have a look at this post regarding cowsay :smile:: <a href="https://medium.com/@jasonrigden/cowsay-is-the-most-important-unix-like-command-ever-35abdbc22b7f" target="_blank">"cowsay is the Most Important Unix-like Command Ever"</a>

```shell
brew install cowsay

cowsay -f vader I am your...LOL!

 __________________
< I am your...LOL! >
 ------------------
        \    ,-^-.
         \   !oYo!
          \ /./=\.\______
               ##        )\/\
                ||-----w||
                ||      ||

               Cowth Vader
```

---

***Last but not least!*** **Asciinema** - https://asciinema.org/

A perfect way how you can easily record your terminal sessions.

```shell
brew install asciinema

asciinema rec
```

<script id="asciicast-aVEG2MvECoVzqtYjkjHWDpw1L" src="https://asciinema.org/a/aVEG2MvECoVzqtYjkjHWDpw1L.js" async></script>

---

## Shell Aliases

Put the following aliases for the individual commands to the end of your `.zshrc` file

```shell
tmux
alias tmuxinit="tmux new-session -n init"
alias c=clear
alias k=kubectl
alias w='watch -n1'
alias ping=prettyping
alias cat=bat
```

# You´re good to go!

---

**Previous articles:**

<a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-i-horizon/">A Linux Development Desktop with VMware Horizon - Part I: Horizon</a>

<a href="https://rguske.github.io/post/a-linux-development-desktop-with-vmware-horizon-part-ii-applications/">A Linux Development Desktop with VMware Horizon - Part II: Applications</a>

---

## <center>**Thanks for reading.**</center>