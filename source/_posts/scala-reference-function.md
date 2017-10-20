title: scala_reference_function
date: 2017-05-27 15:34:19
tags: scala
---

## scala 函数引用

```java
def f(x:Int): Int => Int =  { n:Int => x + n }

val ff = f(2)

println(ff(3))//5
println(ff(5))//7
```

参考资料：
[Scala: return reference to a function](https://stackoverflow.com/questions/3363459/scala-return-reference-to-a-function)
[小码农的碎碎念之Scala](http://www.jianshu.com/p/a52f1cb01a51?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
