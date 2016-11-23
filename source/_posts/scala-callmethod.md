title: scala之空格调用函数
date: 2016-10-28 22:08:11
tags: scala
---

## java和scala调用函数的区别

```java
	object.method();
```
在scala中，这样调用,don't need the semi-colon:

```scala
	object.method()
```

甚至都不用括号(and then you can omit the parentheses):

```scala
	object.method
```

如果方法只有一个参数,

```scala
object.method(param)
```

你可以把点换成空格，而且可以把括号去掉(you can change the Scala syntax to use a space instead of a dot, while also dropping the parentheses):

```scala
	object method param
```


Scala methods that take a single parameter can be invoked without dots or parentheses.

[Scala methods: dots, spaces, and parentheses](http://alvinalexander.com/scala/scala-methods-dots-spaces-single-one-parameter)
