title: scala_def定义函数
date: 2016-11-03 21:21:47
tags: scala
---

## scala def 定义函数的区别

```scala
	 def foo = {1}
```

A function which should return a non-Unit value must be declared with the = notation (although of course the compiler can infer the return-type from the expression's type).

```scala
	def foo {1}
```

返回类型为Unit
 In Scala if a method declaration does not have an equal sign before its body, the compiler infers that the result type will be Unit

 参考
 [scala: 'def foo = {1}' vs 'def foo {1}'](http://stackoverflow.com/questions/1661817/scala-def-foo-1-vs-def-foo-1)

