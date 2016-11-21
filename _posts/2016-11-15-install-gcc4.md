---
layout: post
title: Ubuntu16.04安装gcc4.8
---

32位的Ubuntu16.04默认的gcc版本是gcc 5.4。由于某工具要求的gcc版本是gcc 4.x，因此Aaron需要安装gcc 4.x。

本文总结了在Ubuntu16.04安装低版本的gcc的方法。

```
$ LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
$ export LD_LIBRARY_PATH
$ sudo apt-add-repository ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
$ sudo apt-get install gcc-4.8 g++-4.8
```

One more thing, **I HATE GCC!**

## 参考资料

1. [How to fix Genymotion in linux with error `CXXABI_1.3.8′ not found](https://iamjagjeetubhi.wordpress.com/2016/06/30/how-to-fix-genymotion-in-linux-with-error-cxxabi_1-3-8-not-found/)
