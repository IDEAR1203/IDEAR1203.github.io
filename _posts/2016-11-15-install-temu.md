---
layout: post
title: Ubuntu12.04下的TEMU安装与使用
---

本文主要参考[TEMU/Tracecap, 64-bit Ubuntu 12.04 build walk-through](https://groups.google.com/forum/#!searchin/bitblaze-users/temu%7Csort:relevance/bitblaze-users/QdoY9l8D-ho/GeX8aY2NHOwJ)

TEMU是berkeley的二进制分析平台Bitblaze动态分析组件，详见[BitBlaze: Binary Analysis for Computer Security](http://bitblaze.cs.berkeley.edu/)。

安装过程中遇到了好多问题，本文对TEMU安装和使用进行总结，以飨来者。

## 实验环境

1. 32位Ubuntu 12.04.5 LTS
2. gcc 4.6.3

## 安装

### 系统选择

我尝试了Ubuntu16.04/14.04/12.04/9.04这几个版本，最后在12.04上取得了成功。

* 16.04和14.04的问题在于兼容性，尝试了许多办法，如选取一些低版本的包、使用低版本的gcc等等，最后编译通过，但使用时遇到问题，最终放弃。

### 安装步骤

1. 安装必要软件

    ```
    sudo apt-get install g++
    sudo apt-get install libsdl1.2-dev
    sudo apt-get install libssl-dev
    sudo apt-get install qemu
    sudo apt-get build-dep qemu
    sudo apt-get install binutils-dev
    ```

2. 下载、编译、构建temu

    补丁文件`temu-release2009-gcc4.patch`可从[TEMU/Tracecap, 64-bit Ubuntu 12.04 build walk-through](https://groups.google.com/forum/#!searchin/bitblaze-users/temu%7Csort:relevance/bitblaze-users/QdoY9l8D-ho/GeX8aY2NHOwJ)找到，下同。这里构建了一个样本工程sample_plugin，来测试工程是否能够通过。

    ```
    mkdir bitblaze
    cd bitblaze/
    mkdir temu
    cd temu/
    wget http://bitblaze.cs.berkeley.edu/release/temu-1.0/temu-1.0.tar.gz
    tar zxvf temu-1.0.tar.gz
    cd temu-1.0/
    patch -p0 < ../temu-release2009-gcc4.patch
    ./configure --target-list=i386-softmmu --proj-name=sample_plugin --prefix=$(pwd)/install --disable-gcc-check
    sudo make
    sudo make install
    ```

3. 下载额外的源码，并将其整合到工程中

    ```
    cd ..
    wget http://bitblaze.cs.berkeley.edu/release/additional/bitblaze-additional-2010-06.tar.gz
    tar xvzf bitblaze-additional-2010-06.tar.gz
    mv bitblaze bitblaze-additional-2010-06
    cd temu-1.0/
    rsync -rav ../bitblaze-additional-2010-06/temu/ .
    ```

4. 下载两个开源的组件

    Tracecap依赖于两个开源组件，Sleuthkit和llconf。

    下载、编译、构建Sleuthkit。

    ```
    cd shared/
    mv sleuthkit/sleuthkit-2.04.patch .
    rmdir sleuthkit
    wget  http://sourceforge.net/projects/sleuthkit/files/sleuthkit/2.04/sleuthkit-2.04.tar.gz
    tar zxvf sleuthkit-2.04.tar.gz
    cd sleuthkit-2.04/
    patch -p1 <../sleuthkit-2.04.patch
    patch -p0 <../../../sleuthkit-linux3.patch
    sudo make
    cd ..
    ln -s sleuthkit-2.04 sleuthkit
    ```

    下载、编译、构建llconf：

    ```
    wget https://github.com/lipnitsk/llconf/archive/v0.4.6.tar.gz -O llconf-0.4.6.tar.gz
    tar xvzf llconf-0.4.6.tar.gz
    cd llconf-0.4.6/
    CFLAGS="-fPIC" ./configure --prefix=$(pwd)/install
    sudo make
    sudo make install
    cd ..
    ln -s llconf-0.4.6 llconf
    cd ..
    ```

5. 编译并构建temu，产生tracecap插件。

    ```
    ./configure --target-list=i386-softmmu --proj-name=tracecap --prefix=$(pwd)/install --disable-gcc-check
    make clean
    sudo make
    sudo make install
    ```
