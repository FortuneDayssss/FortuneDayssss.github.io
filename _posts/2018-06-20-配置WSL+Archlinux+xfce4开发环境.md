---
layout: post
title:  "配置WSL + Archlinux + xfce4开发环境"
date:   2018-06-20 05:14:46 -0400
categories: 
---

# 背景：
由于本人开发的某项目IDE使用Windows下的VSCode，而调试在Linux环境下并且需要使用Linux下的图形界面。此时使用虚拟机用来编译/调试开销太大，于是打算配置WSL来解决这个问题。
为了方便使用新版本软件等考虑(本着生命在于折腾的原则)，将WSL中的Ubuntu替换Archlinux。

![image1](/assets/img/2018-6-20/9.png){:height="100" width="100"}

不得不说，pacman管理器与aur用起来真是爽。

> ## 最终运行情况

![image01](/assets/img/2018-6-20/1.png)

# 配置步骤

> ## 1. 安装WSL (Windows Subsystem for Linux)
> 参考  https://wiki.archlinux.org/index.php/ArchWiki:Archive

### 开启Windows开发者模式
![image02](/assets/img/2018-6-20/2.png)
### 从控制面板开启WSL功能
![image03](/assets/img/2018-6-20/3.png)
### 重启后从Windows商店搜索安装Ubuntu,并等待其安装完成
![image04](/assets/img/2018-6-20/4.png)
### 设置ubuntu默认账户为root (为了更改至Archlinux)
![image05](/assets/img/2018-6-20/5.png)
Ubuntu安装完成后无需创建用户，到powershell中将默认账户改为root
```
ubuntu config --default-user root
```
> ## 2. 下载Archlinux替换Ubuntu
### 下载Arch bootstrap.tar.gz至WSL文件系统的/root路径下

Archlinux Download: https://www.archlinux.org/download/

/root path: C:\Users\\[USER_NAME]\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_xxxx\LocalState\rootfs\root

### 更改完默认账户后重新打开bash，以root身份解压Arch Bootstrap.tar.gz

```
$ tar -zxvf archlinux-bootstrap-xxxx.xx.xx-x86_64.tar.gz
```

### 在解压出的root.x86_64文件夹中的etc/pacman.d/mirrorlist中选择适合的服务器取消注释

### 配置/etc/resolv.conf

```
$ vim ~/root.x86_64/etc/resolv.conf
```

```
# Resolver configuration file.
# See resolv.conf(5) for details.
nameserver 192.168.1.1
nameserver 8.8.8.8
nameserver 8.8.4.4
```

关闭bash，使用Windows资源管理器在WSL根路径下删除bin, etc, lib, lib64, sbin, usr, var文件夹，并将/root/root.x86_64中的对应文件移动至根路径(是移动不是复制!)

重新打开bash，安装Archlinux
```
$ pacman-key --init
$ pacman-key --populate archlinux
$ pacman -Syyu base base-devel
```

创建新用户并设置密码
```
$ useradd -m -s /bin/bash username
$ passwd root
$ passwd username
```

在/etc/sudoers中设置sudo权限，如
```
username ALL=(ALL)
```

关闭bash，在powershell中设置默认账户为刚刚创建的账户
```
$ ubuntu config --default-user username
```

> ## 3. 配置GUI

为了轻量化与占资源少，选用了xfce4
```
$ pacman -S xfce4
$ pacman -S xfce4-goodies # optional
```

安装中文适配```(optional)```
```
$ vim /etc/locale.gen
```

```
en_US.UTF-8 UTF-8 # 取消注释
zh_CN.UTF-8 UTF-8 # 取消注释
```

```
$ locale-gen 
$ echo LANG=en_US.UTF-8 > /etc/locale.conf 
$ sudo pacman -S wqy-microhei
```

Windows端架设Xserver，可使用Xming，MobaXterm等软件，选用MobaXterm

https://mobaxterm.mobatek.net

启动MobaXterm，启动Xserver，在要使用GUI的用户的~/.bashrc中配置显示选项
```
export DISPLAY=:0
```
启动需要GUI的程序，效果如下
![image06](/assets/img/2018-6-20/6.png)

> ## 4. 优化

### 在启动文件夹创建使MobaXterm快捷方式，配置开机启动


启动文件夹位于C:\Users\username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

加入-hideterm启动项可以使其在托盘启动
![image07](/assets/img/2018-6-20/7.png)


### 使用xfce4的GUI终端
创建terminal-xfce4.bat
```
C:\Windows\System32\bash.exe -c "export DISPLAY=:0 && xfce4-terminal"
```

使用vbs启动bat消除Windows命令行界面，创建一个vbs文件，内容如下
```
Set shell = Wscript.createobject("wscript.shell")
a = shell.run("C:\Users\yisic\Software\script\terminal-xfce4.bat", 0)
```

vbs启动终端效果如下
![image08](/assets/img/2018-6-20/8.png)

项目运行效果
![image01](/assets/img/2018-6-20/1.png)

### 配置VSCode默认Console为WSL-bash ```(optional)```

在文件>首选项>设置中加入
```
"terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\bash.exe"
```

# 参考资料
Install on WSL (简体中文) - ArchWiki

https://wiki.archlinux.org/index.php/Install_on_WSL_(简体中文)

[HOW-TO] Installing Arch on WSL Manually : bashonubuntuonwindows

https://www.reddit.com/r/bashonubuntuonwindows/comments/5vnne8/howto_installing_arch_on_wsl_manually/
