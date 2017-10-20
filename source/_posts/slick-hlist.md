title: slick_hlist
date: 2017-05-27 15:03:44
tags: scala
---


## slick 使用hlist 用于超过22列

```java
 slick.collection.heterogeneous.{HNil}
 slick.collection.heterogeneous.syntax._

 class Invoice(tag:Tag) extends Table[String :: String :: String :: HNil](tag,"INVOICELOG"){
     def logId = column[String]("LOGID")
   def createDate = column[String]("CREATEDATE")
   def invoiceId = column[String]("INVOICEID")

   def * = logId :: createDate :: invoiceId :: HNil

 }

val i  = db.run(invoices.filter((i) => i.logId === "0af9de5ab5a9457a950ecd9a97ddbe80").result)

val t = Await.result(i, Duration.Inf)

type MyRow  = String::String :: String :: HNil

val  ssss  = t.asInstanceOf[Seq[MyRow]]

   println(ssss.head(0))

   t.foreach((i) =>  {

     i match  {
       case a::b::c::rest =>
           println(a)
     }

  })
```

参考资料：
[How to read an element from a Scala HList?](https://stackoverflow.com/questions/41637704/how-to-read-an-element-from-a-scala-hlist)
[Using shapeless HLists with Slick 3](http://underscore.io/blog/posts/2015/08/08/slickless.html)
[细谈Slick（6）－ Projection：ProvenShape，强类型的Query结果类型](http://www.cnblogs.com/tiger-xc/p/6178065.html)
