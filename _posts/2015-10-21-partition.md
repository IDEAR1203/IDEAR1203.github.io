---
layout: post
title: Virtualbox虚拟硬盘根分区大小调整
---

参考链接：<http://www.cnblogs.com/anjingshen/p/4887426.html>

### 第一步：扩容虚拟硬盘容量 

通过`VBoxManage list hdds`找到所需扩容的虚拟硬盘的UUID

```
$ VBoxManage list hdds
UUID:           5562465c-357e-4cb8-84c4-1e6e25d6943e
Parent UUID:    base
State:          locked write
Type:           normal (base)
Location:       /home/hadoop/VirtualBox VMs/slave1/slave1.vdi
Storage format: VDI
Capacity:       30000 MBytes
Encryption:     disabled

UUID:           8a198e2d-6db7-4a12-93f9-c57286d70cca
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /home/hadoop/VirtualBox VMs/slave1/NewVirtualDisk1.vdi
Storage format: VDI
Capacity:       23142 MBytes
Encryption:     disabled

UUID:           0d94b868-0b95-405b-9015-eac85ac65c63
Parent UUID:    base
State:          locked write
Type:           normal (base)
Location:       /home/hadoop/VirtualBox VMs/slave2/slave2.vdi
Storage format: VDI
Capacity:       30000 MBytes
Encryption:     disabled
```

这里以`slave2.vdi`为例。得到其UUID为`0d94b868-0b95-405b-9015-eac85ac65c63`。

然后`VBoxManage modifyhd uuid --resize size`命令将其扩容，`uuid`是上面获得的UUID，`size`是扩容后的大小。以兆为单位。

例如`VBoxManage modifyhd 0d94b868-0b95-405b-9015-eac85ac65c63 --resize 30000`
## 第二步： 虚拟机光驱启动

在虚拟机设置界面，选择存储选项卡，点击“添加虚拟光驱”，导入iso文件。然后确定。结果如下图

![iso]( {{site.baseurl}}/images/partition/1.png)

在系统选项卡中设置优先光驱启动。把光驱调整至最前。

![device]({{site.baseurl}}/images/partition/2.png)

启动虚拟机，选择LiveCD模式，即点击(try ubuntu)

## 第三步：使用`gparted`合并分区

终端下输入`sudo gparted`，使用`gparted`图形工具进行分区合并。

此时硬盘结构情况如下图：

![sda]({{site.baseurl}}/images/partition/3.png)

我现在我要将`unallocated`区域合并到`sda1`中。

1. 在sd5右键，选择swapoff取消虚拟内存功能。
  ![swapoff]({{site.baseurl}}/images/partition/4.png)
2. 在sd5上右键，选择delete，删除该分区。
  ![delete]({{site.baseurl}}/images/partition/5.png)
3. 同样的方法，删除sda2分区。删除后只剩下sda1和未分配区域。
  ![result]({{site.baseurl}}/images/partition/6.png)
4. 在sda1上右键，选择`Resize/Move`，拖动滑条或者在下面填入数字来调整分区大小。**注意留出一部分作为swap分区**。确定。
  ![resize]({{site.baseurl}}/images/partition/7.png)
5. 接下来在上一步预留的区域里新建扩展分区。未分配的区域右键，选择`New`。在弹出的窗口中`Create as`中选择`Extended Partition`。
  ![new]({{site.baseurl}}/images/partition/8.png)
  ![extended]({{site.baseurl}}/images/partition/9.png)
6. 在`unallocated`区域新建swap分区。跟上一步同样的方法，右键选择`New`，在弹出的窗口中`File system`的下拉菜单中选择`linux-swap`。点击`Add`按钮。
  ![linux-swap]({{site.baseurl}}/images/partition/10.png)
7. 至此，根目录调整工作完毕。最终的硬盘结构如下图。点击工具栏最后的绿色对勾(应用)按钮使之生效。
  ![sdaresult]({{site.baseurl}}/images/partition/11.png)

重新启动虚拟机。大功告成！



