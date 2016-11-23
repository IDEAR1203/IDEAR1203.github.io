---
layout: post
title: 过时的Ubuntu发行版的软件源配置方法
---

Ubuntu9.04是一个古老的发行版，如今已不再被支持。

本文以Ubuntu 9.04为例，介绍了

1. 在不再支持的Ubuntu发行版上配置软件源
2. 配置SSH远程登录
3. 加载内核模块

<!--more-->

## 实验环境

* QEMU虚拟机版本：1.0
* Host操作系统：32位Ubuntu desktop 12.04.5
* Guest虚拟机操作系统：32位的Ubuntu 9.04 server

## 配置软件源

由于Ubuntu9.04是不再支持的版本，因此无法从默认的软件源中更新。`old-releases.ubuntu.com`为不再支持的Ubuntu发行版本提供软件源，将这个网站设置为软件源即可解决问题。
w
使用`vim`打开软件源信息文件（建议先备份），并作如下替换：

```
$ sudo vim /etc/apt/source.list
:%s/[a-z]\{2}.archive.ubuntu.com/old-releases.ubuntu.com/
:%s/security.ubuntu.com/old-releases.ubuntu.com/
```

然后就可以更新软件源了。

```
$ sudo apt-get update
```

Ubuntu 9.04 server版本居然不自带`gcc`和`make`。你没有天线，还怎么做朋友。

```
$ sudo apt-get install gcc make
```

## 配置SSH远程登录

我的Ubuntu9.04是作为QEMU的Guest操作系统，为了方便Host与Guest间的文件交互，我采取SSH方式从Host机器远程登录Guest。本节中GuestOS/Guest均指装载了Ubuntu9.04操作系统的虚拟机；Host是虚拟机的宿主机器，操作系统版本为32位Ubuntu 12.04。

在GuestOS上安装`openssh-server`。

```
$ sudo apt-get install openssh-server
```

启动QEMU虚拟机时添加端口重定向参数`hostfwd=tcp::2222-:22`(注意不要漏掉`-`号以及`:`的个数)，将Guest的22端口重定向到Host的2222端口。

例如，我启动虚拟机的命令如下：

```
$ qemu-system-i386 \
    -device e1000,netdev=user.0 \
    -netdev user,id=user.0,hostfwd=tcp::2222-:22 \
    -hda ~/images/ubuntu904-server.qcow2
```

另一种方式是采用过时的重定向`-redir`参数：

```
$ qemu-system-i386 \
    -redir tcp:2222::22 \
    -hda ~/images/ubuntu904-server.qcow2
```

在Host机器上查看端口`2222`是否处于监听状态：

```
$ netstat -apn | grep 2222
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      23874/qemu-system-i
unix  3      [ ]         STREAM     CONNECTED     12222    1796/unity-2d-shell
```

然后就可以通过SSH访问本机的2222端口来远程登录Guest机器了。`aaron`是登录Guest机器所使用的用户名。

```bash
$ ssh -p 2222 aaron@localhost
```

## 加载内核模块

本节介绍如何通过一个编写好的内核模块来获取操作系统的内核信息。

在Host机器上通过`scp`命令将源文件传递到宿主机器上。

```
$ scp -P 2222 procinfo-ubuntu-hardy.tar.gz aaron@localhost:~
```

在远程机器上解压并构建。

```bash
$ tar -zxvf procinfo-ubuntu-hardy.tar.gz
$ cd procinfo-ubuntu-hardy/
$ make
```

`make`抱怨找不到目录：

```
$ make
make -C /lib/modules/2.6.28-11-server/build M=/home/aaron/procinfo-ubuntu-hardy modules
make: *** /lib/modules/2.6.28-11-server/build: No such file or directory.  Stop.
make: *** [all] Error 2
```

问题发生的原因是没有安装linux内核头文件。

```bash
$ sudo apt-get install linux-headers-`uname -r`
```

重新执行`make`就可以了。

```
$ make
make -C /lib/modules/2.6.28-11-server/build M=/home/aaron/procinfo-ubuntu-hardy modules
make[1]: Entering directory `/usr/src/linux-headers-2.6.28-11-server'
  CC [M]  /home/aaron/procinfo-ubuntu-hardy/procinfo.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/aaron/procinfo-ubuntu-hardy/procinfo.mod.o
  LD [M]  /home/aaron/procinfo-ubuntu-hardy/procinfo.ko
make[1]: Leaving directory `/usr/src/linux-headers-2.6.28-11-server'
```

然后将该模块(`.ko`文件)加载到内核中。注意，必须以超级管理员root权限加载内核模块。

```
sudo insmod procinfo.ko
```
