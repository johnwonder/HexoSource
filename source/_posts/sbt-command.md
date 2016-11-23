title: sbt_command
date: 2016-10-22 14:17:42
tags: sbt
---

## sbt 命令解释

### run command

[run <argument>*](http://www.scala-sbt.org/0.13/docs/Command-Line-Reference.html)Runs a main class, passing along arguments provided on the command line.

运行一个主类，如果在文件内部定义变量或者直接调用函数，那么会提示如下：

```expected class or object definition```

因为会先编译这个文件。

文件内部必须包含main方法的类：

```scala
	object HelloWorld{
	  def main(args: Array[String]) {
	   println(ChecksumAccumulator.calculate("We"))
	  }
	}
```

### package command

在sbt交互模式下输入package，会把当前目录下的project打包，然后运行run命令，如果当前能找到object对象并且有main函数，那么就会执行它。

### exit command

输入exit命令退出sbt交互模式。
