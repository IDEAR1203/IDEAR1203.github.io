---
layout: post
title: 使用QEMU创建Ubuntu12.04虚拟机镜像
---

## 实验环境

* 操作系统：Ubuntu 12.04.5 LTS
* 操作系统类型：32位操作系统
* QEMU版本：1.0

## 操作步骤

### 创建一个空的镜像文件

与普通的raw格式的镜像相比，qcow2格式的镜像文件有以下特性：
1. 更小的空间占用，即使文件系统不支持空洞(holes)；
2. 支持写时拷贝（COW, copy-on-write），镜像文件只反映底层磁盘的变化；
3. 支持快照（snapshot），镜像文件能够包含多个快照的历史；
4. 可选择基于 zlib 的压缩方式
5. 可以选择 AES 加密

下面的命令创建了一个大小为15G的ubuntu1204-server的raw类型的镜像文件。`-f`参数指定镜像文件类型。

```bash
$ qemu-img create -f qcow2 ubuntu1204-server.qcow2 15G
```

### 加载已有的iso镜像文件

```bash
$ qemu-system-i386 -hda ubuntu1204-server.qcow2 -cdrom ~/images/ubuntu-12.04.5-server-i386.iso -boot d -m 1536
```

* `-hda file`，把file作为镜像挂载在到硬盘的第0分区
* `-cdrom file`，把file作为镜像挂载到光驱
* `-boot d`，设置从光驱启动
* `-m megs`，将虚拟机内存大小设置为megs，这里是设置为1.5G（qemu1.0虚拟机内存最大不超过2G）


## 参考资料

1. [QEMU 使用的镜像文件：qcow2 与 raw](http://www.ibm.com/developerworks/cn/linux/1409_qiaoly_qemuimgages/index.html)
