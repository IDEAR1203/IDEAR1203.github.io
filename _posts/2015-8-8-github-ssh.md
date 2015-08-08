---
layout: post
title: github中使用ssh进行身份认证
---

这两天在实验室的台式机上写博客的时候，每次push时github都要求我输入密码，简直是烦透了。今天终于抽出一点时间来解决一下这个问题。

首先，我想弄清楚为什么github总是跟我要密码呢？就不能长点脑子记下来吗？所以我去读了下面这篇文章：[Why is Git always asking for my password?](https://help.github.com/articles/why-is-git-always-asking-for-my-password)。所以原因在于是我采用的是HTTPS方式来克隆仓库。如该文章中所提到的，这种方式的好处在于不需要任何繁琐的设置，开箱即用。同样导致了每次push远程仓库都要输入用户名密码的问题。该文章的最后提出了采用缓存机制使用户在一定时间内免于输入密码的方案。

然而，这对于我仍然是不能忍的。

最终我找到的解决方案是通过SSH方式来与远程仓库交互。

贴一张如何选择SSH克隆url的图。

![ssh clone]({{ site.baseurl }}/images/ssh/4.png)


为什么SSH就可以免于输入密码，而HTTPS就不行呢？这个问题搞清楚这些协议最基本的原理这个问题自然迎刃而解。本文给出一些相关资料。维基百科：[超文本传输安全协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)、[超文本传输协议](https://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)。读完仍然费解，又去找了一些资料，简单的说就是HTTP协议采用的是明文传输也不对通信双方进行身份认证，而HTTPS对传输的报文进行了加密。那么SSH为什么是安全的呢？[SSH原理与运用（一）：远程登录]——这篇文章清晰透彻地对SSH进行了介绍。检验之，SSH采取某种非对称加密算法从而完成了公钥登录。所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。RSA是一种常用的非对称加密算法。正好这学期跟着翟老师学习了应用密码学的课程。贴一下课程笔记。

首先是提出RSA的论文：[A Method for Obtaining Digital Signatures and Public-Key Cryptosystems](https://people.csail.mit.edu/rivest/Rsapaper.pdf)。

然后是我复习时整理的一些资料：

![crypto1]({{ site.baseurl }}/images/ssh/1.png)

![crypto2]({{ site.baseurl }}/images/ssh/2.png)

![crypto3]({{ site.baseurl }}/images/ssh/3.png)

我们再来看看ssh在github上的使用。github中有两种访问级别，用户级访问和项目级别访问。[公钥认证管理](http://www.worldhello.net/gotgithub/03-project-hosting/030-repo-authz.html)阐述的很清楚，不再赘述。

讨论完了是什么、为什么之后，接下来，讨论如何产生SSH秘钥并与Github通信。

Github的官方教程[Generating SSH keys](https://help.github.com/articles/generating-ssh-keys)介绍的十分清晰。

然而我在实际使用时，居然要求我输入passphrase！这和输入用户名密码有什么区别！当时我的内心简直是崩溃的。一定有解决办法！我又阅读了一遍官方教程，[Error: Agent admitted failure to sign](https://help.github.com/articles/error-agent-admitted-failure-to-sign)这篇文章帮助我解决了问题。

终于没有恼人的用户名密码了！

```
$ git push origin master
Counting objects: 18, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 157.48 KiB | 0 bytes/s, done.
Total 10 (delta 3), reused 0 (delta 0)
To git@github.com:idear1203/idear1203.github.io.git
   2472885..ccdf6c7  master -> master``
```
