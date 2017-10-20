title: scala_implicit
date: 2016-11-19 21:06:25
tags: scala
---

## implicit

### 转换成预期的数据类型

比如你有一个方法参数类型是IndexedSeq[Char]，在你传入String时，编译器发现类型不匹配，就检查当前作用域是否有从String到IndexedSeq隐式转换。

Scala在需要時會自動把整數轉換成雙精度實數，這是因為在Scala.Predef對象中定義了一個

implicit def int2double(x:Int) :Double = x.toDouble

而Scala.Predef是自動引入到當前作用域的，因此編譯器在需要時會自動把整數轉換成Double類型。

### 转换selection的receiver

转换selection的receiver允许你适应某些方法调用，比如 “abc”.exist ，”abc”类型为String，本身没有定义exist方法，这时编辑器就检查当前作用域内String的隐式转换后的类型是否有exist方法，发现stringWrapper转换后成IndexedSeq类型后，可以有exist方法，这个和C# 静态扩展方法功能类似。

### 隐含参数

隐含参数有点类似是缺省参数，如果在调用方法时没有提供某个参数，编译器会查找当前作用域是否有符合条件的implicit对象作为参数传入（有点类似dependency injection)

### 隐式变换也可以转换调用方法的对象

编译器看到X.method,而类型X没有定义method（包括基类)方法，那么编译器就查找作用域内定义的从X到其它对象的类型转换，比如Y，而类型Y定义了method方法，编译器就首先使用隐含类型转换把X转换成Y，然后调用Y的method。


隐式转换可以用来扩展Scala语言，定义新的语法结构，比如我们在定义一个Map对象时可以使用如下语法：

```
Map(1 -> "One", 2->"Two",3->"Three")
```

你有没有想过->内部是如何实现的，->不是scala本身的语法，而是类型ArrowAssoc的一个方法。这个类型定义在包Scala.Predef对象中。Scala.Predef自动引入到当前作用域，在这个对象中，同时定义了一个从类型Any到ArrowAssoc的隐含转换。因此当使用1 -> “One”时，编译器自动插入从1转换到ArrowAssoc转换。
