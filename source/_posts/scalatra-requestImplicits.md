title: scalatra之隐式request
date: 2017-08-07 15:04:43
tags: scala
---

## scalatra隐式request

在之前[scala之隐式servletContext]()这篇文章中，我们提到了scalatra实现的隐式servletContext是如何实现的，那今天我们来看看在scalatra中是如何利用scala的特性来实现隐式HttpServletRequest的：

### ScalatraBase
在ScalatraBase这个伴生对象中，我们可以看到如下代码：

```java
private[this] val KeyPrefix: String = classOf[FutureSupport].getName
val Callbacks: String = s"$KeyPrefix.callbacks"

def addCallback(callback: Try[Any] => Unit)(implicit request: HttpServletRequest): Unit = {
  //http://johnwonder.github.io/2017/03/25/scala-implicit1/
  //放入AttributesMap中
  request(Callbacks) = callback :: callbacks
}
```

### ServletApiImplicits

这边的request(Callbacks)到底是啥呢？我们来结合scala的隐式调用来分析

ScalatraBase中引入了ServletApiImplicits

```java
  import org.scalatra.servlet.ServletApiImplicits._
```

```java
//隐式的ServletApi
trait ServletApiImplicits {

  implicit def enrichRequest(request: HttpServletRequest): RichRequest =
  RichRequest(request)
}
```

那说明实现是在RichRequest中

### RichRequest

```java
  case class RichRequest(r: HttpServletRequest) extends AttributesMap {
  }
```

我们看到它还实现了AttributesMap这个trait

### AttributesMap

```java

/**
*映射ServletRequest的属性到map
* Adapts attributes from servlet objects (e.g., ServletRequest, HttpSession,
* ServletContext) to a mutable map.
*/
trait AttributesMap extends Map[String, Any] with MutableMapWithIndifferentAccess[Any] {

}
```

我们看到他最终是继承自Map的，所以request()是Map调用。
