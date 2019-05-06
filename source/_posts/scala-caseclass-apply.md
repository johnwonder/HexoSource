title: scala中caseclass自定义的apply方法
date: 2019-01-07 10:17:38
tags: scala
---

今天在看scalaj-http源码的时候，看到了case class 里定义了apply函数，这个函数在初始化的

时候并不调用，需要再次初始化下才调用这个apply函数，源码定义如下：

```java
case class StringBodyConnectFunc(data: String) extends Function2[HttpRequest, HttpURLConnection, Unit] {
  //apply函数在第一次初始化的时候不会调用
def apply(req: HttpRequest, conn: HttpURLConnection): Unit = {
  conn.setDoOutput(true)
  conn.connect
  conn.getOutputStream.write(data.getBytes(req.charset))
}

override def toString = "StringBodyConnectFunc(" + data + ")"
}
```

关于这个我们可以自己测试下，代码如下：

```java
case class caseclass1(x:Int,y:String) {
	// def apply() = {

	// 	println(x)
	// 	println("case apply")
	// }
	def apply() = {

		caseclass1(5,"3")
	}
}

//定义两个apply会报错  error: method apply is defined twice
object caseclass1 {
	// def apply() = {

	// 	println("object apply")
	// }
}
val class2 = caseclass1(1,"ss")
val class1 = caseclass1(2,"33")

println(class2.x)
println(class1.x)
```

输出
1
2
