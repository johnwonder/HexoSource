title: slick_plainSQL
date: 2017-05-27 14:54:11
tags: scala
---

## slick3.1.1 使用sql语句查询

```java
val db  = Database.forURL("jdbc:mysql://ip:port/db?useUnicode=true",
    driver = "com.mysql.jdbc.Driver",
    user="root",
    password="root")

 def insertInvoice(): DBIO[Int] =
    sqlu"insert   invoicelog values('sss','2017-05-26 12:00:00','sasas')"

 val b:Future[Int] = db.run(insertInvoice())
   Await.result(b, Duration.Inf)

 implicit val getSupplierResult = GetResult(r => InvoiceLog(r.nextString(), r.nextString, r.nextString))
 def selectInvoice(): DBIO[Seq[InvoiceLog]] =
    sql"select * from invoicelog".as[InvoiceLog]

var a: Future[Seq[InvoiceLog]] = db.run(selectInvoice)
val s = Await.result(a, Duration.Inf)

```
