title: scala命令行引入scala文件
date: 2016-10-22 13:26:13
tags: scala
---

## scala命令行引入scala文件

### scala REPL

The Scala Interpreter (often called a REPL for Read-Evaluate-Print Loop).

Scala解释器读到一个表达式，对它进行求值，将它打印出来，接着再继续读下一个表达式。这个过程被称做读取--求值--打印--循环，即：REPL。
从技术上讲，scala程序并不是一个解释器。实际发生的是，你输入的内容被快速地编译成字节码，然后这段字节码交由Java虚拟机执行。正因为如此，大多数scala程序员更倾向于将它称做“REPL”
[scala入门之REPL](https://my.oschina.net/fhd/blog/273965)

### scala -i

```scala
	scala -i extractors.scala
```

运行 scala -help 可以看到如下提示：

```-i <file>    preload <file> before starting the repl```

在启动repl之前预先加载file。

按照这种方式引入的话，如果碰到companion对象,那么会报如下警告：

warning: previously defined class FreeUser is not a companion to object FreeUser
.
Companions must be defined together; you may wish to use :paste mode for this.
defined object PreminumUser

希望你用:paste模式。

### :paste file

先进入REPL，然后运行如下命令:

```:paste file```



