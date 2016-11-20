---
layout: post
title: Mac下使用ShadowSocks科学上网
---
## 所需工具

1. [GoAgentX](https://github.com/ohdarling/GoAgentX/releases)

2. [ShadowSocks](https://github.com/shadowsocks/shadowsocks)

3. [SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega/releases)

4. [Chrome](http://www.google.com/chrome/)

## 方法步骤

1. 下载[GoAgentX](https://github.com/ohdarling/GoAgentX/releases)并安装。

2. 在GoAgentX中配置ShadowSocks客户端。

    第1步，点击左下角的加号，Service Type选择ShadowSocks，Profile Name随便填个名字。

    ![new profile]({{ site.baseurl }}/images/goagent/1.png)

    第2-6步如下图所示。
    * Local Port是本地服务端的端口号，最好填入一个非知名端口号（大于1023的端口号）
    * 使用ShawSocks科学上网的方法成功的关键是要找到一台位于海外的远程服务器。3、4步分别填入远程服务器域名或ip地址以及ShadowSocks服务的端口号。
    * 第5步填入与远程服务器约定的密码
    * 第6步选择加密方式，建议选取AES加密方式。

    ![configure]({{ site.baseurl }}/images/goagent/2.png)

3. 下载并安装[SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega/releases)。

4. 配置SwitchyOmega。

    1. 点击Options。

        ![options]({{ site.baseurl }}/images/goagent/3.png)

    2. 按照下图1-4步新建profile。

        ![new profile]({{ site.baseurl }}/images/goagent/4.png)

    3. 按照下图中5-8步设置profile。

        ![configure profile]({{ site.baseurl }}/images/goagent/5.png)

5. 配置完成！使用方法如下：

    ![howtouse]({{ site.baseurl }}/images/goagent/6.png)

## ShadowSocks科学上网原理

参见[ShadowSocks的翻墙原理](https://tumutanzi.com/archives/13005)
