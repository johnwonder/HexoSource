title: scalatra_mount
date: 2017-03-25 16:13:39
tags: scala
---

## scalatra是如何加入servlet的

```java
private def mountServlet(
  servlet: HttpServlet,
  urlPattern: String,
  name: String,
  loadOnStartup: Int): Unit = {
  val reg = Option(sc.getServletRegistration(name)) getOrElse {
    val r = sc.addServlet(name, servlet)
    servlet match {
      case s: HasMultipartConfig => //如果有HasMultipartConfig 特质
        r.setMultipartConfig(s.multipartConfig.toMultipartConfigElement)
      case _ =>
    }
    if (servlet.isInstanceOf[ScalatraAsyncSupport])
      r.setAsyncSupported(true)
    r.setLoadOnStartup(loadOnStartup)
    r
  }

  reg.addMapping(urlPattern)
}

private def mountServlet(
  servletClass: Class[HttpServlet],
  urlPattern: String,
  name: String,
  loadOnStartup: Int): Unit = {
  val reg = Option(sc.getServletRegistration(name)) getOrElse {
    val r = sc.addServlet(name, servletClass)
    // since we only have a Class[_] here, we can't access the MultipartConfig value
    // if (classOf[HasMultipartConfig].isAssignableFrom(servletClass))
    if (classOf[ScalatraAsyncSupport].isAssignableFrom(servletClass)) {
      r.setAsyncSupported(true)
    }
    r.setLoadOnStartup(loadOnStartup)
    r
  }
  reg.addMapping(urlPattern)
}
```
