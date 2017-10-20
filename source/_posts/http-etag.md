title: http_etag
date: 2017-03-11 15:09:24
tags: scala
---

## http etag

Etag[1] 是URL的Entity Tag，用于标示URL对象是否改变，区分不同语言和Session等等。具体内部含义是使服务器控制的，就像Cookie那样。
HTTP协议规格说明定义ETag为“被请求变量的实体值”。另一种说法是，ETag是一个可以与Web资源关联的记号（token）。典型的Web资源可以一个Web页，但也可能是JSON或XML文档。服务器单独负责判断记号是什么及其含义，并在HTTP响应头中将其传送到客户端，以下是服务器端返回的格式：ETag:"50b1c1d4f775c61:df3"客户端的查询更新格式是这样的：If-None-Match : W / "50b1c1d4f775c61:df3"如果ETag没改变，则返回状态304然后不返回，这也和Last-Modified一样。测试Etag主要在断点下载时比较有用。

## scalatra cache

先看源码：

```java
trait CacheSupport { self: ScalatraBase =>
implicit val cacheBackend: Cache //在具体的servlet中实现
implicit val keyStrategy: KeyStrategy = DefaultKeyStrategy
implicit val headerStrategy: HeaderStrategy = DefaultHeaderStrategy //重写了HeaderStrategy的方法

def cache[A](key: String, ttl: Option[Duration])(value: => A): A = {
  cacheBackend.get[A](key) match {
    case Some(v) => v
    case None => cacheBackend.put(key, value, ttl)
  }
}

//用于网页设置etag用
def cached[A](ttl: Option[Duration])(result: => A)(implicit keyStrategy: KeyStrategy,
    headerStrategy: HeaderStrategy,
    request: HttpServletRequest,
    response: HttpServletResponse): A = {

    val key = keyStrategy.key

    cacheBackend.get[(String, A)](key) match {
      case Some(v) =>
        if (headerStrategy.isUnchanged(v._1)) halt(304)
        else {
          headerStrategy.setRevision(v._1) // response.setHeader("ETag", revision)
          v._2
        }
      case None =>
        val res = result //网页内容
        val rev = headerStrategy.getNewRevision() //默认获取当前时间
        cacheBackend.put(key, (rev, res), ttl)
        headerStrategy.setRevision(rev)
        res
    }
  }
}
```

[HTTP Header中的ETag ](http://blog.csdn.net/chenzhiqin20/article/details/10947857)
