title: scala_map
date: 2017-03-13 16:51:09
tags: scala
---

## scala map 学习

在看scalatra的源码时，发现其中定义了一个MultiMap类，如下：

```java
  //MultiMap的伴生对象
  object MultiMap {

  def apply(): MultiMap = new MultiMap

  def apply[SeqType <: Seq[String]](wrapped: Map[String, SeqType]): MultiMap = new MultiMap(wrapped)

  def empty: MultiMap = apply()

  //隐式函数
  implicit def map2MultiMap(map: Map[String, Seq[String]]): MultiMap = new MultiMap(map)

  }

  class MultiMap(wrapped: Map[String, Seq[String]] = Map.empty)
  extends Map[String, Seq[String]] {

  def get(key: String): Option[Seq[String]] = {
  (wrapped.get(key) orElse wrapped.get(key + "[]"))
  }

  /*This class provides a simple way to get unique objects for equal strings.
   *  Since symbols are interned, they can be compared using reference equality.
   *  Instances of `Symbol` can be created easily with Scala's built-in quote
   *  mechanism.*/
   //相同的字符串获得同一个对象
  def get(key: Symbol): Option[Seq[String]] = get(key.name)

  def +[B1 >: Seq[String]](kv: (String, B1)): MultiMap = {
  new MultiMap(wrapped + kv.asInstanceOf[(String, Seq[String])])
  }

  def -(key: String): MultiMap = new MultiMap(wrapped - key)

  def iterator: Iterator[(String, Seq[String])] = wrapped.iterator

  override def default(a: String): Seq[String] = wrapped.default(a)

  }
```

### map用法：

```java

  val mapTest  = Map[String,String] ("key1" -> "faeff")
  println(mapTest.get("key1").get)

  val map2 =mapTest - "key1"
```
