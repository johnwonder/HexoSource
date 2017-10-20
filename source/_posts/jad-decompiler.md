title: jad_decompiler
date: 2017-07-10 10:49:12
tags: scala
---

用jad来反编译scala代码，看看版本是否是老版本：

```java
jad.exe -r -ff -d src -s java F:\class\**\*.class
```

反编译的代码会在执行jad程序的目录src下

参考文章：
[Windows下利用jad批量反编译class文件](http://blog.csdn.net/jiajane/article/details/50947853)
[反编译jad的命令使用](http://blog.csdn.net/abeetle/article/details/1733407)
