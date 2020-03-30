title: Spring源码分析之ClassPathXmlApplicationContext
date: 2020-03-20 13:45:00
tags: Spring
---

我们在分析Spring源码时，通常会自己创建一个独立的应用上下文，比如本文说的ClassPathXmlApplicationContext。我们在实际开发过程中一般框架内部会初始化一个应用程序上下文，比如springmvc中的XmlWebApplicationContext还有springboot中的AnnotationConfigServletWebServerApplicationContext。

## ClassPathXmlApplicationContext 简介

官方文档是这么说的：

Standalone XML application context, taking the context definition files from the class path,
interpreting plain paths as class path resource names that include the package path
(e.g. "mypackage/myresource.txt"). Useful for test harnesses(测试框架) as well as
for application contexts embedded within JARs.

也就是说它可以创建独立应用程序上下文，获取上下文的定义文件等等。它的主要功能如下：

1. 定义了很多构造函数，只是参数不同，不同的参数具体会影响配置资源的定义和spring应用上下文的刷新(refresh函数)。

2. 配置资源路径的不同，获取方式的不同。如果设置configLocations参数，那么会通过该类的父类也就是抽象类AbstractRefreshableConfigApplicationContext的resolvePath函数来解析。如果使用带有class参数的构造函数，那么会使用ClassPathResource来获取资源。

### AbstractEnvironment

resolvePath函数内部其实是调用AbstractEnvironment的resolveRequiredPlaceholders方法，方法内部会调用propertyResolver它的父类AbstractPropertyResolver的resolveRequiredPlaceholders方法。

```java

private final ConfigurablePropertyResolver propertyResolver =
			new PropertySourcesPropertyResolver(this.propertySources);

  @Override
	public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
		return this.propertyResolver.resolveRequiredPlaceholders(text);
	}
```


参考资料：

[Spring 中 ClassPathXmlApplicationContext 类的简单使用](https://blog.csdn.net/qq_37960603/article/details/82709660)
[彻底搞懂Class.getResource和ClassLoader.getResource的区别和底层原理](https://blog.csdn.net/zhangshk_/article/details/82704010)
[SpringMVC加载流程](https://baijiahao.baidu.com/s?id=1655259813981508769&wfr=spider&for=pc)
