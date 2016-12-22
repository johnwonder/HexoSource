title: scala_import_problem
date: 2016-12-22 09:27:26
tags: scala
---

## scala文件导入java包

定义一个stringTest.java文件如下：

```java

public class stringTest {

  public static String queryVehicleNum(){

      return "ss";
  }
}
```

用```javac```命令编译此文件，在目录下会生成stringTest.class
此时我们并没有定义它的package,如果在同级目录下新建javaStringTest.scala文件如下：

```java
object testImport{

   def main(args: Array[String]){

     println(stringTest.queryVehicleNum())

   }
}
```

可以看到，此时可以直接引入stringTest类不会报错，用scala命令运行此文件会输出ss

如果在目录下新建com目录，在com目录下新建test目录,把stringTest文件放入test目录，那么再执行scala testImport.scala文件就会报如下错误：

```
java.lang.ClassNotFoundException: stringTest
at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
at Main$.main(javaStringTest.scala:9)
at Main.main(javaStringTest.scala)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
```

所以此时我们要在testImport类中引入stringTest包```import com.test.stringTest```

但是这样我们必须先编译scala文件,```scalac javaStringTest.scala```

然后我们用java命令调用生成的testImportObj ```java -classpath .;scala-library.jar testImportObj```

或者我们也可以用```scala -i testImportObj.java```命令引入scala REPL

然后我们在scala命令行输入``` testImportObj.main(new Array[String](3))```也会输出ss字符串
