title: scala_mongodb驱动的问题
date: 2017-07-04 17:08:54
tags: scala
---

## scala mongodb groupby 多个字段

今天同事问groupby多个字段是怎么写的

然后上[$group (aggregation)](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#pipe._S_group)
看了下，发现要写表达式，看了这篇文章[MongoDB aggregate 运用篇 个人总结](http://www.tuicool.com/articles/eAFjUbB)，发现要像如下写：

```java

多列group，根据name和status进行多列

db.collection.aggregate([

{$group:{_id:{name:"$name",st:"$status"},count:{$sum:1}}}

]);
```

那么代码中应该怎么写呢，研究了半天，看到这篇文章[MongoDB getCollection with a specific document class](https://stackoverflow.com/questions/36521817/mongodb-getcollection-with-a-specific-document-class)

发现我们得从定义mongodbClient开始定义个codeRegistry

```java

case class GroupByCode(name:String,code:String)  {
    // val id: String = _id.toString
  }
  private[bigdata] val personCodecProvider = Macros.createCodecProvider[GroupByCode]()
  private[bigdata] val codecRegistry = fromRegistries( fromProviders(personCodecProvider), DEFAULT_CODEC_REGISTRY )

  private[bigdata] val connectionString = new ConnectionString(utils.dbUrl)
  private[bigdata] val   options = MongoClientSettings
    .builder()
    .codecRegistry(codecRegistry)
    .clusterSettings(ClusterSettings.builder().applyConnectionString(connectionString).build())
    .connectionPoolSettings(ConnectionPoolSettings.builder().applyConnectionString(connectionString).build())
    .serverSettings(ServerSettings.builder().build()).credentialList(connectionString.getCredentialList)
    .sslSettings(SslSettings.builder().applyConnectionString(connectionString).build())
    .socketSettings(SocketSettings.builder().applyConnectionString(connectionString).build())
    .build()
  //private[bigdata] val mongoClient = MongoClient(utils.dbUrl)
  private[bigdata] val mongoClient = MongoClient(options)
```


然后在用的地方直接

```java
     val groupBy = group(GroupByCode("$userType","$code"),sum("count", 1))

     Await.result(prdb.aggregate(filter(Filters.and(Filters.eq("code", parkCode), Filters.gte("exitDate", date), Filters.lt("exitDate", c.getTime))) :: groupBy :: Nil).toFuture(), read_timeout)
```

当然mongodb-scala-driver得升级到[2.0版](http://mongodb.github.io/mongo-scala-driver/2.1/bson/macros/)才支持caseClassCodec

New in 2.0, the Scala driver allows you to use case classes to represent documents in a collection via the Macros helper. Simple case classes and nested case classes are supported. Hierarchical modelling can be achieve by using a sealed trait and having case classes implement the parent trait.
