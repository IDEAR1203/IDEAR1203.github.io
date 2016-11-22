---
layout: post
title: 过时的Ubuntu发行版的软件源配置方法
---

Ubuntu9.04是一个古老的发行版，如今已不再被支持。

本文以Ubuntu 9.04为例，介绍了

1. 在不再支持的Ubuntu发行版上配置软件源
2. 配置SSH远程登录
3. 加载内核模块
4. 使用TCP协议传输文件

<!--more-->

## 实验环境

* QEMU虚拟机版本：1.0
* Host操作系统：32位Ubuntu desktop 12.04.5
* Guest虚拟机操作系统：32位的Ubuntu 9.04 server

## 配置软件源

由于Ubuntu9.04是不再支持的版本，因此无法从默认的软件源中更新。`old-releases.ubuntu.com`为不再支持的Ubuntu发行版本提供软件源，将这个网站设置为软件源即可解决问题。

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

## 使用TCP协议传输文件

QEMU模式的网络通信协议是NAT协议。

NAT的基本思想是，为每个公司分配一个IP地址，用于传输Internet流量。在每个公司内部，每台计算机有唯一的IP地址，它使用该地址来传输内部流量。然而，当一个分组离开公司的网络，发向ISP时，它需要执行一个地址的转换。为了保证这种方案的可行性，有三段IP地址范围已经被声明为私有地址。这三段保留的地址范围为

| 基地址        | 子网掩码                |主机地址个数 |
|--------------|------------------------|---------  |
| 10.0.0.0     | -10.255.255.255/8      |  16777216 |
| 172.16.0.0   |  -172.31.255.255/12    |  1048576  |
| 192.168.0.0  |  -192.168.255.255/16   |  65536    |

当一个进程希望与另一个远程进程建立TCP连接时，它绑定到一个本地机器尚未使用的TCP端口上，该端口称为**源端口**，它告诉TCP代码，凡是属于该连接进来的分组都应该发送给这个进程。这个进程也要提供一个**目标端口**，已执行分组被送到远程机器上应该给谁。从`0`到`1023`之间的端口都是保留端口，用于一些知名的服务。例如`80`端口被用于Web服务器，所以，远程客户可以找到Web服务。每个向外发送的TCP消息都包含一个源端口和一个目标端口。这两个端口合起来标识出了客户端和服务器端正在使用该连接的进程。

利用`source port`（源端口）域，我们可以解决前面的映射问题。任何时候，当一个向外发送的分组进入到NAT盒的时候，源地址`10.x.y.z`被公司的真实IP地址所取代，而且，TCP的`source port`域被一个索引值所取代，该索引值指向NAT盒的地址转换表中65535个表项之一。该表项包含了原来的IP地址和原来的源端口。最后NAT盒重新计算IP头和TCP头的校验和，并将校验和插入到分组中。这里之所以要替换`source port`域，是因为`10.0.0.1`和`10.0.0.2`出发的连接可能碰巧使用了同一个端口，比如`5000`，所以，仅仅使用`source port`域还不足以唯一标识发送进程。

当一个分组从ISP到达NAT盒的时候，NAT盒从TCP头中提取中源端口，把他作为索引值从NAT盒的映射表中找到对应的表项，然后从该表项中提取出内部IP地址和原来的TCP `source port`，并将它们插入到分组中。然后重新计算IP和TCP的校验和，并插入分组中。最后将该分组传递给公司内部的路由器，它使用`10.x.y.z`地址进行正常的路由。

Aaron所使用的宿主机器所在的网段是`192.168.1.0/24`。

QEMU默认给虚拟机分配的IP地址是`10.0.2.15`，这是虚拟的DHCP服务器可以分配的第一个IP地址。虚拟的DHCP服务器所在的地址是`10.0.2.2`。虚拟的DNS服务器的地址是`10.0.2.3`。

查看虚拟机的网络信息：

```
aaron@ubuntu904-i386:~$ ifconfig -a
eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe12:3456/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:345 errors:0 dropped:0 overruns:0 frame:0
          TX packets:308 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:45014 (45.0 KB)  TX bytes:46252 (46.2 KB)
          Interrupt:11 Base address:0xc100

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:760 (760.0 B)  TX bytes:760 (760.0 B)
```

查看虚拟机的路由表：

```
aaron@ubuntu904-i386:~$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
0.0.0.0         10.0.2.2        0.0.0.0         UG        0 0          0 eth0
```

从路由表得到内部网络网关IP是`10.0.2.2`。我们将使用这个IP与宿主机器进行通信。

我们使用的通信工具是`NetCat`，命令简称是`nc`。终端下输入`man nc`可以查看这个工具的手册，可以知道这是一个用于TCP/UDP链接和监听的工具。

```
NC(1)                     BSD General Commands Manual                    NC(1)

NAME
     nc -- arbitrary TCP and UDP connections and listens

...

DESCRIPTION
     The nc (or netcat) utility is used for just about anything under the sun involving TCP or UDP.  It can open TCP connections, send UDP packets, listen on
     arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6.  Unlike telnet(1), nc scripts nicely, and separates error messages onto
     standard error instead of sending them to standard output, as telnet(1) does with some.
```


首先，让宿主机器在某个端口监听TCP连接，这里选取端口`12345`：

```
$ nc -l 12345
```

然后，让虚拟机创建连接，目标地址为网关IP，目标端口`12345`:

```
$ nc 10.0.2.15 12345
```

这时就建立好了TCP连接，宿主机器可以与虚拟机器通信了。

在宿主机器终端输入的消息：

```
aaron@aaron-u1204:~$ nc -l 12345
hello
world
```

消息将实时显示在虚拟机器上。

```
aaron@ubuntu904-i386:~$ nc 10.0.2.2 12345
hello
world
```

注意通信是双向的，虚拟机发送的消息也将实时的显示在宿主机上。按`ctrl` + `c`结束通信。

这种方法可以用于宿主机器和虚拟机器之间的文件传输。

例如，假设宿主机器上有一个文件`input.txt`，文件内容如下：

```
aaron@aaron-u1204:~/Desktop$ cat input.txt
this is
a
input
file
.
```

下面通过`nc`命令将其传输到虚拟机上。

宿主机终端执行命令：

```
aaron@aaron-u1204:~/Desktop$ nc -l 12345 < input.txt
```

虚拟机上采用如下命令接收文件：

```
aaron@ubuntu904-i386:~/test$ nc 10.0.2.2 12345 > input.txt
```

在虚拟机上查看文件内容：

```
aaron@ubuntu904-i386:~/test$ ls
input.txt
aaron@ubuntu904-i386:~/test$ cat input.txt
this is
a
input
file
.
```

由此可见，文件已经成功的传输到虚拟机器上了。

## 参考资料

1. 计算机网络. 潘爱民译. 清华大学出版社
2. [QEMU/Networking](https://en.wikibooks.org/wiki/QEMU/Networking)
