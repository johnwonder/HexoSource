title: scala_collection
date: 2016-11-06 20:39:04
tags: scala
---

## 映射

映射（Map）是一种可迭代的键值对结构（也称映射或关联）。Scala的Predef类提供了隐式转换，允许使用另一种语法：key -> value，来代替(key, value)。如：Map("x" -> 24, "y" -> 25, "z" -> 26)等同于Map(("x", 24), ("y", 25), ("z", 26))，却更易于阅读。

[collections](http://docs.scala-lang.org/zh-cn/overviews/collections/maps)


## colon-equals

It's very likely you := in a SBT build file. See github.com/harrah/xsbt/wiki/Settings for all of SBT's assignmenty operators. 

IMHO : In My Humble Opinion 恕我直言
[What is the difference between = and := in Scala?](http://stackoverflow.com/questions/7749530/what-is-the-difference-between-and-in-scala)
