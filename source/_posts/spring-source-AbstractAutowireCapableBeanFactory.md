title: spring5.1.1源码剖析之AbstractAutowireCapableBeanFactory
date: 2019-05-29 13:44:40
tags: Spring
---

## 关于AbstractAutowireCapableBeanFactory

源码文档中是这么说的：

Abstract bean factory superclass that implements default bean creation,
with the full capabilities specified by the {@link RootBeanDefinition} class.
Implements the {@link org.springframework.beans.factory.config.AutowireCapableBeanFactory}
interface in addition to AbstractBeanFactory's {@link #createBean} method.

org.springframework.beans.factory.support.DefaultListableBeanFactory 这个很重要的类继承自
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

简单点说这个beanfactory做了三样事情
1. 实现默认bean创建，bean拥有RootBeanDefinition类指定的所有能力
2. 实现了AutowireCapableBeanFactory接口。
3. 实现了AbstractBeanFactory的createBean方法。
