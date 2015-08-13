---
layout: post
title: mac下vim补全插件YouCompleteMe安装与使用
---

今晚偶然打开自己的vimrc发现里面的YouCompleteMe配置段被我注释掉了。原因是因为我觉得YouCompleteMe安装过程需要编译llvm和clang，过于麻烦，并且时间很长。然后我去github上看了一下官方安装教程，发现可以用预编译的llvm二进制发行版。于是感觉可以在半小时内搞定，所以就在实验室的台式机ubuntu上试了一下。安装过程没有遇到任何困难。

人们都说YouCompleteMe是vim自动补全的杀手级软件。也许是我还用的不太习惯，感觉它并不能跟上编码的速度。有停下来找所需要的关键词的功夫已经输入完了。

下面总结一下在mac上的安装方法。

本文参考了[YouCompleteMe的github](https://github.com/Valloric/YouCompleteMe)。为方便起见，下称YouCompleteMe为YCM。

## 使用效果图

![effect]({{ site.baseurl }}/images/YCM/1.png )