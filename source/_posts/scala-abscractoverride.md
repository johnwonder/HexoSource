title: scala_abscractoverride
date: 2017-03-31 09:42:43
tags: scala
---

## abstract override

前几天在看scalatra源码时，看到FileUploadExample这个实例类时，竟然提示我如下错误：  

` error: class FileUploadExample needs to be a mixin, since method initialize in trait HasMultipartConfig of type => Unit is marked abstract and override`  

我在HasMultipartConfig中可以看到如下定义：

```java
abstract override def initialize(config: ConfigT) {
 super.initialize(config)

 providedConfig foreach { _ apply config.context }
}
```  

这个initialize其实是定义在Initializable这个trait中的：  

```java
  trait Initializable {

    /**
     * A hook to initialize the class with some configuration after it has
     * been constructed.
     *
     * Not called init because GenericServlet doesn't override it, and then
     * we get into https://lampsvn.epfl.ch/trac/scala/ticket/2497.
     */
      def initialize(config: ConfigT)
  }
```

也就是说HasMultipartConfig 中abstract override了它，那么为什么会报错呢？搜索了相关资料，有如下答案：  


My understanding is that while the error message may be confusing, the behaviour is correct. foo is declared as abstract override in StackingTrait, and thus in any concrete class that mixes StackingTrait there must be a concrete (not marked as abstract) implementation of foo before StackingTrait (relative to the linearization order). This is because super refers to the trait just before in the linearization order, so there definitely needs to be a concrete implementation of foo before StackingTrait is mixed in, or super.foo would be nonsensical.

When you do this:  

```java
class Impl extends Base with StackingTrait {
  def foo {}
}
```

the linearization order is Base <- StackingTrait <- Impl. The only trait before StackingTrait is Base and Base does not define a concrete implementation of foo.

But when you do this:    

```java
traitImplHelper extends Base {
  def foo {}
}
class Impl extends ImplHelper with StackingTrait
```

参考资料：  
[Scala: Trait Mixin with Abstract Base Class](http://stackoverflow.com/questions/14169542/scala-trait-mixin-with-abstract-base-class)
