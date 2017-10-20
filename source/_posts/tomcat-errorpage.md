title: tomcat_errorpage
date: 2017-05-12 10:02:07
tags: tomcat
---


## scalatra 自定义404文件

在scalatra中，如果访问servlet不存在的路由时，默认为返回
`Requesting "%s %s" on servlet "%s" but only have: %s`

那么我就去源码看了看，发现如下代码：

```java
protected var doNotFound: Action = () => {
    serveStaticResource() getOrElse resourceNotFound()
  }

  /**
   * Attempts to find a static resource matching the request path.  Override
   * to return None to stop this.
   */
  protected def serveStaticResource(): Option[Any] =
    servletContext.resource(request) map { _ =>
      servletContext.getNamedDispatcher("default").forward(request, response)
    }

  /**
   * Called by default notFound if no routes matched and no static resource
   * could be found.
   */
  protected def resourceNotFound(): Any = {
    response.setStatus(404)
    if (isDevelopmentMode) {
      val error = "Requesting \"%s %s\" on servlet \"%s\" but only have: %s"
      response.getWriter println error.format(
        request.getMethod,
        Option(request.getPathInfo) getOrElse "/",
        request.getServletPath,
        routes.entryPoints.mkString("<ul><li>", "</li><li>", "</li></ul>"))
    }
  }protected var doNotFound: Action = () => {
    serveStaticResource() getOrElse resourceNotFound()
  }

  /**
   * Attempts to find a static resource matching the request path.  Override
   * to return None to stop this.
   */
  protected def serveStaticResource(): Option[Any] =
    servletContext.resource(request) map { _ =>
      servletContext.getNamedDispatcher("default").forward(request, response)
    }

  /**
   * Called by default notFound if no routes matched and no static resource
   * could be found.
   */
  protected def resourceNotFound(): Any = {
    response.setStatus(404)
    if (isDevelopmentMode) {
      val error = "Requesting \"%s %s\" on servlet \"%s\" but only have: %s"
      response.getWriter println error.format(
        request.getMethod,
        Option(request.getPathInfo) getOrElse "/",
        request.getServletPath,
        routes.entryPoints.mkString("<ul><li>", "</li><li>", "</li></ul>"))
    }
  }
```
发现它如果没找到路由时，会尝试通过defaultServlet找静态资源，如果还找不到那么就会进入resourceNotFound方法，

如果是开发模式下，那么就会返回一开始提到的内容。。

如果我们通过在web.xml中配置：
```xml
<context-param>
        <param-name>org.scalatra.environment</param-name>
        <param-value>production</param-value>
    </context-param>
```

就不会显示配置的404页面，那是怎么回事呢？

我们来看看response.setStatus(404)

```java
/**
     * Sets the status code for this response.
     *
     * <p>This method is used to set the return status code when there is
     * no error (for example, for the SC_OK or SC_MOVED_TEMPORARILY status
     * codes).
     *
     * <p>If this method is used to set an error code, then the container's
     * error page mechanism will not be triggered. If there is an error and
     * the caller wishes to invoke an error page defined in the web
     * application, then {@link #sendError} must be used instead.
     *
     * <p>This method preserves any cookies and other response headers.
     *
     * <p>Valid status codes are those in the 2XX, 3XX, 4XX, and 5XX ranges.
     * Other status codes are treated as container specific.
     *
     * @param	sc	the status code
     *
     * @see #sendError
     */
    public void setStatus(int sc);
```

有这么段话：
If this method is used to set an error code, then the container's
* error page mechanism（机制） will not be triggered.

参考资料：
[Default Servlet](http://tomcat.apache.org/tomcat-4.1-doc/catalina/funcspecs/fs-default.html)
[Tomcat use DefaultServlet for static content in external directory](http://stackoverflow.com/questions/32337999/tomcat-use-defaultservlet-for-static-content-in-external-directory)
[Tomcat处理静态文件DefaultServlet分析 ](http://blog.csdn.net/husan_3/article/details/23792517)
[Configuration](http://scalatra.org/guides/2.3/deployment/configuration.html)
[Configuring Web Application Components](https://docs.oracle.com/cd/E13222_01/wls/docs81/webapp/components.html)
