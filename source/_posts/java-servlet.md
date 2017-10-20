title: java_servlet
date: 2017-06-16 22:24:15
tags: scala
---

## scalatraServlet实现

前几天看了别人的面试题，提到了HttpServlet实现了哪几个接口，

今天就抽空看了下scalatraServlet中的源码如下：

```java
public abstract class HttpServlet extends GenericServlet{
  ...
}
public abstract class GenericServlet
  implements Servlet, ServletConfig, java.io.Serializable
{
  ...
}
```

### Servlet
Defines methods that all servlets must implement.
就是说定义了所有servlets必须实现的方法。

### ServletConfig


* A servlet configuration object used by a servlet container
* to pass information to a servlet during initialization.

一个被servlet容器用来传递信息给servlet初始化的配置对象

### java.io.Serializable

java的序列化接口

Serializability of a class is enabled by the class implementing the
java.io.Serializable interface. Classes that do not implement this
 interface will not have any of their state serialized or
deserialized.  
