title: slf4j_MDC
date: 2016-12-29 16:50:56
tags: slf4j
---

# slf4j MDC

A lighter technique consists of uniquely stamping each log request servicing a given client. Neil Harrison described this method in the book Patterns for Logging Diagnostic Messages in Pattern Languages of Program Design 3, edited by R. Martin, D. Riehle, and F. Buschmann (Addison-Wesley, 1997).

Logback leverages a variant of this technique included in the SLF4J API: Mapped Diagnostic Contexts (MDC).

在分布式系统中，各种无关日志穿行其中，导致我们可能无法直接定位整个操作流程。因此，我们可能需要对一个用户的操作流程进行归类标记，比如使用线程+时间戳，或者用户身份标识等；如此，我们可以从大量日志信息中grep出某个用户的操作流程，或者某个时间的流转记录。

因此，这就有了 Slf4j MDC 方法

转载自[Slf4j MDC 使用和 基于 Logback 的实现分析](http://www.ithao123.cn/content-8291525.html)

[LogBack sl4j 通过MDC实现日志记录区分用户Session](http://www.cnblogs.com/dumuqiao/p/3654702.html?utm_source=tuicool&utm_medium=referral)

## slf4j-api MDC类

```java

/**
   *内部静态类
   * An adapter to remove the key when done.
   */
  public static class MDCCloseable implements Closeable {
      private final String key;

      private MDCCloseable(String key) {
          this.key = key;
      }

      public void close() {
          MDC.remove(this.key);
      }
  }

```

Java中的类可以是static吗？答案是可以。在java中我们可以有静态实例变量、静态方法、静态块。类也可以是静态的

静态内部类只能访问外部类的静态成员

静态内部类和非静态内部类之间到底有什么不同呢？下面是两者间主要的不同。

> 内部静态类不需要有指向外部类的引用。但非静态内部类需要持有对外部类的引用。

> 非静态内部类能够访问外部类的静态和非静态成员。静态类不能访问外部类的非静态成员。他只能访问外部类的静态成员。

> 一个非静态内部类不能脱离外部类实体被创建，一个非静态内部类可以访问外部类的数据和方法，因为他就在外部类里面。

参考资料:
[java中的Static class](http://www.cnblogs.com/kissazi2/p/3971065.html)
