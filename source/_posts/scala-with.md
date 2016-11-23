title: scala_with关键字
date: 2016-10-29 21:00:09
tags: scala
---

## with 关键字

with关键字可以用来实现包装器的功能,新建withSample.scala文件示例如下：

```scala
	trait Friendly {
	  def greet() = "Hi"
	}

	class Dog extends Friendly {
	  override def greet() = "Woof"
	}

	class HungryDog extends Dog {
	  override def greet() = "I'd like to eat my own dog food"
	}

	trait ExclamatoryGreeter extends Friendly {
	  override def greet() = super.greet() + "!"
	}

	var pet: Friendly = new Dog
	println(pet.greet())

	pet = new HungryDog
	println(pet.greet())

	pet = new Dog with ExclamatoryGreeter
	println(pet.greet())

	pet = new HungryDog with ExclamatoryGreeter
	println(pet.greet())
```

用scala命令调用(scala withSample.scala)，输出如下

```scala
	woof
	I'd like to eat my own dog food
	woof!
	I'd like to eat my own dog food!
```

张逸的[Scala中的Partial Function](http://www.tuicool.com/articles/reiMBzA)一文中提到PartialFunction的定义如下:

```scala
	trait PartialFunction[-A, +B] extends (A => B) { self =>
	  import PartialFunction._
	  def isDefinedAt(x: A): Boolean
	  def applyOrElse[A1 <: A, B1 >: B](x: A1, default: A1 => B1): B1 =
	    if (isDefinedAt(x)) apply(x) else default(x)
	}
```

追本溯源，是因为这里对偏函数值的调用，实则是调用了AbstractPartialFunction的apply()方法(case语句相当于是继承AbstractPartialFunction的子类)：

```scala
	abstract class AbstractPartialFunction[@specialized(scala.Int, scala.Long, scala.Float, scala.Double, scala.AnyRef) -T1, @specialized(scala.Unit, scala.Boolean, scala.Int, scala.Float, scala.Long, scala.Double, scala.AnyRef) +R] extends Function1[T1, R] with PartialFunction[T1, R] { self =>
    def apply(x: T1): R = applyOrElse(x, PartialFunction.empty)
}
```

看到里面用到了with PartialFunction[T1,R]。

[Scala 学习笔记（一）](http://www.cnblogs.com/elaron/archive/2013/01/18/2866142.html)