---
layout: post
title: 在Ubuntu12.04上源码编译安装Boost
---

Boost库是一个可移植、提供源代码的C++库，由Boost社区组织开发维护，其目的是为C++程序员提供免费、同行审查的、可移植的程序库。

Aaron在Ubuntu12.04上编译某工具时，遇到了Boost版本过低的问题。

```
CMake Error at /usr/share/cmake-2.8/Modules/FindBoost.cmake:1202 (message):
  Unable to find the requested Boost libraries.

  Boost version: 1.46.1

  Boost include path: /usr/include

  Detected version of Boost is too old.  Requested version was 1.55 (or
  newer).
```

最终从一篇博文找到了从源码编译Boost的方法，解决了问题。从这篇博文中除了学习到了如何源码编译安装Boost外，还学习了查看机器内核数目的方法。

我将这篇博文进行翻译，记在这里，以备后用。

<!--more-->

首先，获取所需版本的Boost程序库。这里使用的是1.55版本的Boost库，你可以改为你所需要的版本。

```
wget -O boost_1_55_0.tar.gz http://sourceforge.net/projects/boost/files/boost/1.55.0/boost_1_55_0.tar.gz/download
tar xzvf boost_1_55_0.tar.gz
cd boost_1_55_0/
```

更新软件源，并安装必要的软件包。

```
sudo apt-get update
sudo apt-get install build-essential g++ python-dev autotools-dev libicu-dev build-essential libbz2-dev
```

对Boost编译进行必要的配置。

```
./bootstrap.sh --prefix=/usr/local
```

更改`user-config.jam`，添加MPI。

```
user_configFile=`find $PWD -name user-config.jam`
echo "using mpi ;" >> $user_configFile
```

找到机器最大物理核数。

```
n=`cat /proc/cpuinfo | grep "cpu cores" | uniq | awk '{print $NF}'`
```

使用多核并行编译安装Boost。

```
sudo ./b2 --with=all -j $n install 
```

至此，安装完成。

## 参考资料

1. [Installing Boost 1.55 from source on Ubuntu 12.04](https://coderwall.com/p/0atfug/installing-boost-1-55-from-source-on-ubuntu-12-04)
