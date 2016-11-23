title: scala_apply函数
date: 2016-11-02 20:58:41
tags: scala
---

## scala apply方法

```scala
	class ApplyOperation{

	}

	class ApplyTest{
		def apply() = println("I am into spark so much!")
		def haveTry: Unit ={
			println("have a try on apply")
		}
	}

	object ApplyTest{
		def apply() ={
			println("I am into Scala so much")

			new ApplyTest
		}
	}

	object ApplyOperation{
		def main(args : Array[String]){
			val array = Array(1,2,3,4)

			val a = ApplyTest()// 调用object 的apply

			a.haveTry

			a() // class apply use
		}
	}
```

输出
I am into Scala so much
have a try on apply
I am into Spark so much!!!

参考
[scala apply方法 笔记](http://blog.csdn.net/pzw_0612/article/details/48576569)
[动手实战Scala中的apply方法和单例对象](http://book.51cto.com/art/201408/449448.htm)

## scala case class apply

 Scala会给case类自动添加一个单例对象

 参考[探索Scala（4）-- Case Classes ](http://blog.csdn.net/zxhoo/article/details/40454075)