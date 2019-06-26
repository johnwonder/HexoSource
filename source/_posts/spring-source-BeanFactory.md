title: spring5.1.1源码剖析之BeanFactory接口
date: 2019-06-26 13:42:54
tags: Spring
---

## 关于BeanFactory接口

The org.springframework.beans and org.springframework.context packages are the basis for Spring Framework’s IoC container. The BeanFactory interface provides an advanced configuration mechanism capable of managing any type of object. ApplicationContext is a sub-interface of BeanFactory。

`org.springframework.beans` 和 `org.springframework.context` 两个包是Spring框架IoC
容器的基础。BeanFactory接口为任何类型提供了高级配置能力。ApplicationContext是BeanFactory的子接口,它提供了以下功能：

1. Easier integration with Spring’s AOP features
(与Spring AOP特性的简单集成)

2. Message resource handling (for use in internationalization) (处理Message resource,用于国际化)

3. Event publication (事件发布)

4. Application-layer specific contexts such as the WebApplicationContext for use in web applications. (应用层指定上下文,比如Web应用的WebApplicationContext)

为啥这么说呢？我们来看AppplicationContext接口：

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
  MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
  }
```
1. ApplicationEventPublisher跟Event publication相关
2. MessageSource跟Message resource handling相关


In short, the BeanFactory provides the configuration framework and basic functionality, and the ApplicationContext adds more enterprise-specific functionality. The ApplicationContext is a complete superset of the BeanFactory and is used exclusively in this chapter in descriptions of Spring’s IoC container.

大致意思如下：

简单的说，BeanFactory提供配置框架和基础的函数，子接口ApplicationContext加入了针对商业特定的方法。
ApplicationContext是BeanFactory完整的超集，在这个Spring IoC容器章节描述中会专门使用。

In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.

在Spring世界里，构成应用程序主干和被IoC容器管理的对象称之为beans。一个bean要么实例化，组装，要么被容器管理。除此以外，一个bean就是你应用里许多对象中的一个。
Beans和他们的依赖是在容器使用的配置元数据里反射的。

参考资料：

1. [ Introduction to the Spring IoC Container and Beans](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html)
