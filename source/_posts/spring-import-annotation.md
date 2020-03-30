title: spring源码分析之@Import注解
date: 2019-07-06 17:42:36
tags: Spring
---


## @Import定义

Indicates one or more @Configuration classes to import.

表示一个或多个类会被导入，参考@Configuration.

Provides functionality equivalent to the <import/> element in Spring XML.

提供在xml中<import>标签相同的功能.

Allows for importing   @Configuration classes, ImportSelector and  ImportBeanDefinitionRegistrar implementations,as well as regular component classes (as of 4.2; analogous to  AnnotationConfigApplicationContext#register).

允许导入@Configuration类，ImportSelector和ImportBeanDefinitionRegistrar的实现类,等同于正常的组件类(4.2开始;类似于AnnotationConfigApplicationContext.register).

不过我从spring文档中看到AnnotationConfigRegistry接口是从4.1版本开始的

@Bean definitions declared in imported @Configuration classes should be accessed by using org.springframework.beans.factory.annotation.Autowired @Autowired injection. Either the bean itself can be autowired, or the configuration class instance declaring the bean can be autowired. The latter approach allows for explicit, IDE-friendly navigation between @Configuration class methods.

在导入类里定义的Bean应该可以被Autowired注入访问。bean自身或者定义bean的配置类都能被autowired。后者能在类方法之间被显式的导航. 这个我们在之后使用示例中会展示。

May be declared at the class level or as a meta-annotation.

在class层级定义或者当做一个meta-annotation。

If XML or other non-@Configuration bean definition resources need to be imported, use the @ImportResource annotation instead.

如果xml或者其他没有@Configuration的bean资源需要被导入，那就使用@ImportResource注解来替代。


参考资料：

1. [Spring Import 三种用法与源码解读](https://www.jianshu.com/p/7eb0c2b214a7)
2. [Spring @Import注解 —— 导入资源](https://blog.csdn.net/pange1991/article/details/81356594)
3. [@Autowired用法详解](https://blog.csdn.net/horacehe16/article/details/79811763)
