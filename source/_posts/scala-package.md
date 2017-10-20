title: scala_package
date: 2017-03-31 15:29:23
tags: scala
---

## scala 打包子项目

在idea中有多个项目时，我想单独打包子项目，搜了几篇文章受到了启发，  

只要在命令行中执行`sbt module-a/clean module-a/test`命令即可.  

其中 module-a 为定义在根项目中的变量.  
比如：
lazy val `api` = (project in file("api")).settings(javadocSettings: _*).dependsOn(`common`, `bigdata`).aggregate(`common`, `bigdata`)


[How to execute package for one submodule only on Jenkins?](http://stackoverflow.com/questions/18075820/how-to-execute-package-for-one-submodule-only-on-jenkins)
