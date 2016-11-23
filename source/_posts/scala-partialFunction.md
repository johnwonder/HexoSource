title: scala_partialFunction
date: 2016-10-24 21:59:42
tags: scala
---

## 偏函数

### 语法

```scala
	val pf: PartialFunction[Int, String] = {
   		case i if i%2 == 0 => "even"
 	}
```

也可以由orElse组成：
```scala
	val tf: (Int => String) = pf orElse { case _ => "odd"}

	tf(1) == "odd"
	tf(2) == "even"
```

```scala
	trait Publisher[T] {
   		def subscribe(f: PartialFunction[T, Unit])
	}

	 val publisher: Publisher[Int] = ..
	 //定义subscribe函数
	 publisher.subscribe {
	   case i if isPrime(i) => println("found prime", i)
	   case i if i%2 == 0 => count += 2
	   
	 }
```

## 偏应用函数

偏应用函数（Partial Applied Function)是缺少部分参数的函数，是一个逻辑上概念

### 语法和示例

Scala里，当你调用函数，传入任何需要的参数，你就是在把函数应用到参数上。如，给定下列函数：

```scala
    scala> def sum(a: Int, b: Int, c: Int) = a + b + c
    sum: (Int,Int,Int)Int
```

偏应用函数是一种表达式，你不需要提供函数需要的所有参数。代之以仅提供部分，或不提供所需参数

```scala
    scala> val a = sum _
    a: (Int, Int, Int) => Int = < function>
```

[Scala: 偏函数(PartialFunction) && 偏应用函数(Partial Applied Function](http://www.aiuxian.com/article/p-1741977.html)
