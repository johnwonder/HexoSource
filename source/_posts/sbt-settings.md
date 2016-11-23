title: sbt-settings
date: 2016-10-22 21:32:02
tags: sbt
---

## sbt Settings 

Remember, a build definition creates a list of Setting, which is then used to transform sbt’s description of the build (which is a map of key-value pairs). A Setting is a transformation with sbt’s earlier map as input and a new map as output. The new map becomes sbt’s new state.

build定义会创建setting列表，用于转变build的描述，是key-value pairs.
一个setting是之前映射的会输出新的映射。新的映射是sbt新的状态。

Different settings transform the map in different ways. Earlier, you read about the := method.

不同的settings通过不同的方式改变映射。比如:=。

The Setting which := creates puts a fixed, constant value in the new, transformed map. For example, if you transform a map with the setting name := "hello" the new map has the string "hello" stored under the key name. 

:= 创建了新的常量，如果你设置了 name := "Hello" ,那么就存储了key 为name值为hello的map.
在build.sbt里面并非key := value, 而是key := expression. 文件里的每一行其实是一句scala语句

[More-About-Settings](http://www.scala-sbt.org/0.13/docs/More-About-Settings.html)

[sbt从入门到半熟](http://beike.iteye.com/blog/1575296)

### sbt +=

+=，将值添加进现有值里，适用于集合类型的key，比如libraryDependencies
如果是`version := "0.1"`,再用version += "0.2"，那么在交互模式里show version，会变成0.10.2

### sbt.Attributed

sbt.Attributed[java.io.File]

Retrieves the associated value of key from the metadata.

顾名思义，是获取java.io.File类型的Attributed[D]

```scala
def put[T](key: AttributeKey[T], value: T): Attributed[D]

Defines a mapping key -> value in the metadata.
```

[final case class Attributed[D](data: D)](http://www.scala-sbt.org/release/api/index.html#sbt.Attributed)



