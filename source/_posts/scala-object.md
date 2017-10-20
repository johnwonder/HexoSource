title: scala_object
date: 2017-03-24 09:26:06
tags: scala
---

## scala object 线程安全

这篇[Scala thread-safety question](http://www.scala-lang.org/old/node/6610)回复

提到如下代码：

```java
  object Class {
  val students = new mutable.List()
  }
```

"object" initialization is thread safe. "mutable.List" is not thread
safe and needs to be synchronized.

然后我们自己定义一个object用于测试：

```java
  object  TestObject {

    val prop1 = "sad"

    lazy  val prop2 = "testLazy"
    println("propinitialize")
    private val props: java.util.Map[String, Object] = new java.util.HashMap[String, Object]

    props.put("prop2",prop2)//访问prop2的时候会给prop2赋值

    def initialize() = {
      println("initialize")
    }

    def apply() ={

      println(props.get("prop2").toString)
      println("initializing")
    }
}
```
用以下代码测试：

```java
object  TestObjectInitializer extends  App {

  val exce = Executors.newFixedThreadPool(6)

  for (i <- 0.to(100)){
    val runner = new Runnable {
      def run(): Unit = {

        TestObject()
      }}
    exce.submit(runner)
  }

  println("initialized")
}
```

打印如下代码：

```
propinitialize
initialized
testLazy
testLazy
initializing
testLazy
testLazy
initializing
testLazy
testLazy
initializing
initializing
```

propinitialize只执行一次。

参考资料：
[Scala Singleton Object with Multi-threading](http://stackoverflow.com/questions/19934176/scala-singleton-object-with-multi-threading)
[How to make object (a mutable stack) thread-safe](http://stackoverflow.com/questions/8318461/how-to-make-object-a-mutable-stack-thread-safe)
