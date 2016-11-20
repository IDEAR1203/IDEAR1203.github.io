---
layout: post
title: Ubuntu16.04安装gcc4.8
---

笔者使用的系统是32位的Ubuntu16.04，默认的gcc版本是gcc 5.4。由于某工具要求的gcc版本是gcc 4.x，因此需要安装gcc 4.x。

尝试过从源码编译gcc，耗时巨长，并且容易出错，故而笔者最终抛弃了这种方式，而是采用添加软件源的方式来安装gcc 4.x。

```bash
 $ LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
 $ export LD_LIBRARY_PATH
 $ sudo apt-add-repository ppa:ubuntu-toolchain-r/test
 $ sudo apt-get update
 $ sudo apt-get install gcc-4.8 g++-4.8
 ```

 One more thing, **I HATE GCC!**

本文参考[How to fix Genymotion in linux with error `CXXABI_1.3.8′ not found](https://iamjagjeetubhi.wordpress.com/2016/06/30/how-to-fix-genymotion-in-linux-with-error-cxxabi_1-3-8-not-found/)
