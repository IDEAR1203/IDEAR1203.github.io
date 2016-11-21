---
layout: post
title: maven打jar包指定依赖包的相对路径
---

## 问题描述

用maven package对项目打jar包，默认会在项目根目录/target下面生成jar包，然后目标jar所依赖的其他jar会放在target/lib下面。直接执行`java -jar 目标.jar`，会抛出`ClassNotFoundException`异常。

```
Exception in thread "main" java.lang.NoClassDefFoundError: org/springframework/context/support/ClassPathXmlApplicationContext
    at cn.net.xxxx.xxxx.main(xxxx.java:39)
Caused by: java.lang.ClassNotFoundException: org.springframework.context.support.ClassPathXmlApplicationContext
    at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
    ... 1 more
```

<!--more-->

## 原因分析

一定有什么地方可以指定依赖包所在的目录！

下称我们项目所生成的目标jar包为project.jar。通过`unzip project.jar -d project`对产生的project.jar进行解压（`-d project`将所有解压的文件放在project目录下）。进入解压后的目录，打开`META-INF/MANIFEST.MF`。文件内容如下：

```
Manifest-Version: 1.0
Built-By: doze
Build-Jdk: 1.7.0_75
Class-Path: dependency1.jar dependency2.jar dependency3.jar
```

就是它了！那么如何在每个所依赖的jar前加上lib/的前缀呢？

## 问题解决

maven提供了这个功能。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.5</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <useUniqueVersions>false</useUniqueVersions>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>主类的位置</mainClass>
            </manifest>
        </archive>

    </configuration>
</plugin>
```

解决问题的主要是这一句：

```xml
<classpathPrefix>lib/</classpathPrefix>
```
