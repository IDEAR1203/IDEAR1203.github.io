---
layout: post
title: "论文阅读笔记： FlowDroid: Precise Context, Flow, Field, Object-sensitive and Lifecycle-aware Tainted Analysis for Android Apps"
---

## 摘要

这篇论文介绍FLOWDROID——一种新型的高精度的用于安卓程序污点分析的静态分析工具。该工具对安卓生命周期进行了精确的建模，由此可以正确地处理了回调函数。

该工具具有以下特点：

- context sensitive
- flow sensitive
- field sensitive
- object sensitive

除了FlowDroid分析工具外，论文作者还提出了droidBench，一个用来检测针对安卓程序的污点分析工具的效率和精度。

实验结果表明，他们的方法在droidBench上的准确率达到了86%，回召率达到了93%，远胜于商业软件AppScan和Fortify SCA。

## 1. 引言

安卓系统在手机市场的占有率已经达到了75%，2013-2014年的增长率为91.5%。随着安卓手机的占有率的提升，安卓系统逐渐成为了个人隐私和信息安全攻击的目标。

### 动态分析：

Google's Bouncer在app上传到Play Store后对app测试5分钟。

不足：app可以绕过这种方法。1.实现设定一个时间，在这个时间段内关闭恶意activity。2.通过ip地址或其他信息来识别检测环境。

### 静态分析

静态分析的难点主要在于安卓程序不是独立的封闭程序，而是在安卓framework中运行的。安卓的回调函数机制增加了app生命周期的复杂度。

**回调函数、生命周期是什么**

### 论文贡献

- FlowDroid，学术界第一个context/object/flow sensitive的针对安卓全生命周期的污点分析方法。
- 基于上述方法的开源工具的实现。
- DroidBench，一个新的、开源的、全面的测试程序集。
- 将FlowDroid与商业工具AppScan和Fortify进行对比，比较这些工具在精确度precision和召回率recall。

`P = TP/(TP+FP)`:  反映了被分类器判定的正例中真正的正例样本的比重

`R = TP/(TP+FN) = 1 - FN/T`: 反映了被正确判定的正例占总的正例的比重

## 2. 背景与示例

- field sensitity: User有两个域，username和password。我们不关心username，我们只关心password。

- object sensitity: 这个例子没有体现，但是显然有必要区分同一代码片段产生的存储位置不同的对象。

### IFDS框架

Resps提出的IFDS框架可以将函数间的数据流分析问题转化为图可达性问题。

**解释IFDS的边集点集**

IFDS可以解决函数间的状态有限的满足分配率的数据流分析的子集问题。分析过程依靠的模型是exploded supergraph。例如，对于`x = y`的语句，如果y属于污点，则在语句之前fact set是`{y}`，在语句执行之后，fact set是`{x, y}`。模型构建完成之后，通过判定结点`(s, x)`是否可达来决定`s`语句中的变量`x`是否可达。

### 攻击模型

假定：

1. 攻击者可以向用户提供含有恶意字节码的应用程序。
2. 攻击者可以改变安装环境和应用程序的输入
3. 攻击者没有办法突破安卓平台的安全防护措施


## 3. 对生命周期的精准建模

### 多入口

与java程序不同，安卓程序没有main方法。

安卓有4大组件：

- activity: 当前与用户交互的界面
- service: 后台任务
- content provider: 持久化存储相关
- broadcast receiver: 用来监听全局事件

activity生命周期图：

![activity lifecycle]( {{site.baseurl}}/images/flowdroid/1.png )

4种状态：

**解释4种状态**

由此导致的结果是，分析安卓程序构建调用图时不能简单的从寻找main方法着手。必须对安卓生命周期中所有可能的传递关系进行建模。FlowDroid采用*dummy main*的方法来对生命周期进行仿真。然而生命周期并非这么简单。存在某些函数来保存和恢复状态。回调函数也可以造成额外的状态变化。

### 组件的异步执行

一个应用程序可能包含多个组件，如3个activities和1个service。actitity虽然是顺序执行的，但是在app实际运行之前事先其执行顺序是无法确定的。例如，1个activity作为初始化界面，然后根据用户不同的输入切换到不同的actitity。service可以并行执行。

因此，flowdroid假定所有组件可能按照任意顺序执行。

## 4. Context/object/Field/Flow Sensitive的分析方法

解决的问题：别名分析

解决方法：正向污点分析 + on demand的逆向别名分析

采用access path来实现object sensitivity:

`x.f.g.h`： access path长度为3

### 4.1 污点分析

转换函数：

- 赋值语句：`lhs = rhs`， 如果`rhs`tainted，则`lhs`taints.
- 涉及数组的赋值语句：某污点变量对数组元素赋值，则整个数组被污染。
- 带`new`的赋值语句：`x = new ...`，所有根植于x的access path中的变量从污染集合除去
- 函数调用语句：从调用函数的context转换到被调函数的context
- 函数返回语句：从返回函数的context转换到被调函数的context

### 4.2 按需的别名分析

现有的别名分析技术存在以下问题：

- 开销太大：对程序中出现的所有程序变量进行分析
- 不够精确：不能满足污点分析对context-sensitity的精度要求

别名分析是逆向分析，发生时机：对堆中的变量赋值。`x.f = v`。

## 5 实现

Soot: 提供分析的一些基本信息，例如精确的函数调用图。基于Dexpler插件Soot可以将JAVA源代码以及安卓的dex文件转换为Jimple的IR表示。这使得FlowDroid可以既可以分析Java源码，又可以发分析安卓的dex文件。

### 系统架构

工具的系统框图如下：

![architecture]( {{site.baseurl}}/images/flowdroid/2.png )

Manifest.xml: 应用全局配置文件，

*.dex: Dalvik虚拟机字节码（应用程序）

layout xml 布局配置文件

Android在运行程序时首先需要解压apk文件，然后获取编译后的androidmanifest.xml文件中配置信息，执行dex程序。

解压apk文件后，flowdroid生命周期相关的函数、回调函数，以及作为source和sink的函数。

然后为生命周期和回调函数产生main方法。main方法用来产生调用图以及inter-procedural control-flow graph(ICFG)。

### UI界面交互

读取layout XML文件，找到文本输入框，与源代码中的读取文本的语句相关联。


### 回调函数

1. 首先构建一张没有回调函数的函数调用图
2. 为每个活动构建一个含有调用图的main方法

### 待改进之处

1. flowdroid没有考虑反射调用
2. 不明确的控制依赖产生
3. flowdroid没有考虑多线程导致的可能出现的信息泄漏

## 6. 实验评估

**RQ1** FlowDroid和商业的安卓污点分析工具在精度和召回率的比较情况？

**RQ2** FlowDroid在Insecure Bank测试集上的效果如何？Insecure Bank是一个用来测试安卓分析工具的能力和性能的测试集。

**RQ3** FlowDroid可以用于实际应用程序的泄漏检测吗？速度如何？

**RQ4** FlowDroid在纯java程序的污点分析问题的效果如何？

### 6.1 与商业污点测试工具的比较

由于没有专门针对安卓的测试集，因此，论文作者开发了DroidBench。该测试集包含了39个小的安卓应用程序，可用于安卓污点静态、动态分析的评估。

比较对象：

IBM AppScan Source v8.7

HP Fortify SCA

结论：

AppScan和Fortify为了降低误报，牺牲了回召率，因此漏报了一些实际存在的隐私泄漏错误。而FlowDroid回召率比上述两个工具有显著提高，精度上也有小幅优势。

### 6.2 RQ2: Performance on InsecureBank

InsecureBank是Paladion Inc.制作的用于评估分析工具水平的安卓程序。它包含了许多种与实际应用程序类似的漏洞和数据泄漏。

测试环境：

- Intel Core 2 Centrino CPU
- 4 GB 物理内存
- Windows 7
- Oracle JRE V1.7 64 bit

笔记本电脑

测试结果：

FlowDroid找到了所有的数据泄漏缺陷，既没有误报，也没有漏报。用时31s。

### 6.3 在实际应用程序中的测试情况

#### Google Play

论文作者将FlowDroid应用于超过500个Google Play中的应用程序。

精度：

没有发现恶意软件。检测到的主要问题是敏感数据例如位置信息泄漏在logs或者是preference文件中。

效率：

绝大多数检测在1分钟内完成。用时最长的app是Sumsang's Push Service，用时4.5分钟。

#### VirusShare

在VirusShare项目中测试。该项目包含了约1000个已知的恶意程序样本。平均运行时间为16s，最短用时5s，最长用时71s。大部分app含有2个数据泄漏问题（平均每个app含1.85个）。

**IMEI是什么？**

主要是将IMEI发送给远端服务器。某些恶意软件通过broadcast receiver来接收数据并通过短信发送，使得其他应用程序可以不经过用户许可就发送短信。

### 6.4 SecuriBenchMicro

纯java程序测试。SecuriBenchMicro v1.08。

### 6.5 与其他工具比较

TrustDroid, LeakMiner, CHEX










