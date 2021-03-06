title: scala中的合成函数(synthetic function)
date: 2017-11-20 14:42:18
tags: scala
---

今天看到akka http库中有这么一段函数：

```Java
//R <: Rendering 意思是 renderQuery 可以设置任何Rendering类的子类
def renderQuery[R <: Rendering](r: R, query: Query, charset: Charset,
                keep: CharPredicate = `strict-query-char-np`): r.type = {
    //函数内部定义的函数
    def enc(s: String): Unit = encode(r, s, charset, keep, replaceSpaces = true)
    @tailrec def append(q: Query): r.type =
      q match {
        case Query.Empty ⇒ r
        case Query.Cons(key, value, tail) ⇒
          if (q ne query) r ~~ '&'
          enc(key)
          if (value ne Query.EmptyValue) r ~~ '='
          enc(value)
          append(tail)
      }
    append(query)
  }

```

我查了下ne 就是和eq 类似的，相当于not equal 和equal的关系。这个函数字面意思就是显示
查询参数的，看到里面的```q ne query``` 这一句来了兴趣，查看stackoverflow上是这么写的：

A synthetic field, instead, is a field generated by the compiler to work around
the underlying JVM limitations, especially when dealing with inner anonymous
classes, a concept extraneous to the JVM

参考资料：
[What's the difference between A<:B and +B in Scala?](https://stackoverflow.com/questions/4531455/whats-the-difference-between-ab-and-b-in-scala)
[Scala的eq,ne,equals,==方法与Java异同](http://blog.csdn.net/dax1n/article/details/70599491)
[Synthetic Function “##” in scala](https://stackoverflow.com/questions/32054647/synthetic-function-in-scala/32054844)
