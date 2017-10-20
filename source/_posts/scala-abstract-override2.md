title: scala_abstract_override2
date: 2017-03-31 10:21:25
tags: scala
---

## scala abstract override

看到了[Scala's Stackable Trait Pattern](http://www.artima.com/scalazine/articles/stackable_trait_pattern.html)这篇文章， 继续研究scala abstract override：

```java
  trait Base {
  def foo
  }

  trait StackingTrait extends Base {
  abstract override def foo { println ("aa")}
  }

  class ImplHelper extends Base {
  def foo {}
  }

  class Impl   extends Base{
    def foo {
          println("ss")
    }
  }

  val implInstance = new Impl with StackingTrait

  implInstance.foo
```  

打印出来了aa  

This arrangement is frequently needed with traits that implement stackable modifications. To tell the compiler you are doing this on purpose, you must mark such methods as abstract override. This combination of modifiers is only allowed for members of traits, not classes, and it means that the trait must be mixed into some class that has a concrete definition of the method in question. 
