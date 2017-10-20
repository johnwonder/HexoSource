title: scala之caseClass
date: 2016-10-23 21:16:26
tags: scala
---

## case class

1 构造器中的参数如果不被声明为var的话，它默认的话是val类型的，但一般不推荐将构造器中的参数声明为var

2 自动创建伴生对象，同时在里面给我们实现子apply方法，使得我们在使用的时候可以不直接显示地new对象

3 伴生对象中同样会帮我们实现unapply方法，从而可以将case class应用于模式匹配

```scala
	//抽象类Person
	abstract class Person

	//case class Student
	case class Student(name:String,age:Int,studentNo:Int) extends Person
	//case class Teacher
	case class Teacher(name:String,age:Int,teacherNo:Int) extends Person
	//case class Nobody
	case class Nobody(name:String) extends Person

	object CaseClassDemo{
	  def main(args: Array[String]): Unit = {
	    //case class 会自动生成apply方法，从而省去new操作
	    val p:Person=Student("john",18,1024)  
	    //match case
	    p  match {
	      case Student(name,age,studentNo)=>println(name+":"+age+":"+studentNo)
	      case Teacher(name,age,teacherNo)=>println(name+":"+age+":"+teacherNo)
	      case Nobody(name)=>println(name)
	    }
	  }
	}
```
sbt中的Attributed就是个case class

以下代码摘录自sbt/util仓库：

位置：internal/util-collection/src/main/scala/sbt/internal/util/Attributes.scala

```scala
	/** Associates a `metadata` map with `data`. */
	/*使一个metadata 映射 和data产生关系
	final case class Attributed[D](data: D)(val metadata: AttributeMap) {
	  /** Retrieves the associated value of `key` from the metadata. */
	  def get[T](key: AttributeKey[T]): Option[T] = metadata.get(key)

	  /** Defines a mapping `key -> value` in the metadata. */
	  def put[T](key: AttributeKey[T], value: T): Attributed[D] = Attributed(data)(metadata.put(key, value))

	  /** Transforms the data by applying `f`. */
	  def map[T](f: D => T): Attributed[T] = Attributed(f(data))(metadata)
	}
	object Attributed {
	  /** Extracts the underlying data from the sequence `in`. */
	  def data[T](in: Seq[Attributed[T]]): Seq[T] = in.map(_.data)

	  /** Associates empty metadata maps with each entry of `in`.*/
	  def blankSeq[T](in: Seq[T]): Seq[Attributed[T]] = in map blank

	  /** Associates an empty metadata map with `data`. */
	  def blank[T](data: T): Attributed[T] = Attributed(data)(AttributeMap.empty)
	}
```

关于Attributed类参数有两个括号的问题参考：

[scala函数编程的柯里化](http://www.tuicool.com/articles/6zymymu)
[Scala 函数柯里化(Currying)](https://wizardforcel.gitbooks.io/w3school-scala/content/10-9.html)

[Scala class和case class的区别](https://www.iteblog.com/archives/1508)
[Scala入门到精通](http://www.bkjia.com/yjs/1041692.html)
[Scala单例对象、伴生类以及伴生对象、apply介绍 ](http://blog.csdn.net/yyywyr/article/details/50194005)
