title: gradle知识点整理
date: 2019-06-14 17:52:34
tags: gradle
---

## buildscript

buildscript中的声明是gradle脚本自身需要使用的资源。可以声明的资源包括依赖项、第三方插件、maven仓库地址等。
而在build.gradle文件中直接声明的依赖项、仓库地址等信息是项目自身需要的资源。

1. [Gradle中的buildScript代码块](https://www.cnblogs.com/qiangxia/p/4826532.html#top)

## subprojects

allprojects是对所有project的配置，包括Root Project。而subprojects是对所有Child Project的配置

1.[Gradle配置中subprojects 和 allprojects 的区别](https://www.jianshu.com/p/84ac62747e59)

参考资料：

1. [Gradle学习系列之八——构建多个Project](https://www.cnblogs.com/davenkin/p/gradle-learning-8.html)
