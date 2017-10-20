title: scala之隐式servletContext
date: 2017-03-25 15:20:48
tags: scala
---

## scala 转换被方法调用的对象

在scalatra启动时，会mount多个servlet ,但是servletContext并没有定义mount方法，那为什么servletContext能调用mount方法呢？

这就是scala隐式转换的魅力。

在看scalatra源码时，看到在trait ServletApiImplicits定义了如下代码：
```java
// 用来调用mount
implicit def enrichServletContext(servletContext: ServletContext): RichServletContext =
 RichServletContext(servletContext)
```  

在RichServletContext中定义了mount方法：
```java
/**
* Mounts a handler to the servlet context.  Must be an HttpServlet or a
* Filter.
*
* @param handler the handler to mount
*
* @param urlPattern the URL pattern to mount.  Will be appended with `\/\*` if
* not already, as path-mapping is the most natural fit for Scalatra.
* If you don't want path mapping, use the native Servlet API.
*
* @param name the name of the handler
*/
def mount(handler: Handler, urlPattern: String, name: String): Unit = {
 mount(handler, urlPattern, name, 1)
}
```

看如下代码：

```java
implicit def enrichServletContext(sc: String): RichServletContext =
  RichServletContext(sc)

case class RichServletContext(sc:String){

def mount():Unit = {
  println(sc)
}
}

val ss = "sssss"
ss.mount
```  

在这篇文章中就提到了[转换被方法调用的对象](http://www.guidebee.info/wordpress/archives/5076)
