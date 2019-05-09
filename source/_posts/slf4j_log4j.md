title: log4j_cannot_log_in_tomcat
date: 2016-12-29 14:05:34
tags: [slf4j,java]
---

## slf4j 日志配置

前几天遇到了一个奇怪的问题，log4j在程序调试时是可以输出到日志文件中的，但是放到tomcat下却只生成到了tomcat自带的日志文件stdout中

log4j 配置如下：

```
#开发
  log4j.rootLogger=ALL,CONSOLE,FILE,OUT,FILEERROR,FILEWARN
  #正式
  #log4j.rootLogger=CONSOLE,FILE,OUT,FILEERROR,FILEWARN
  log4j.logger.com.mchange.v2.log.MLog=ALL
  og4j.logger.slick=WARN,ERROR
  log4j.logger.com.ht=ALL
  log4j.logger.akka.http=WARN,ERROR
  log4j.logger.org.reflections=ERROR
  log4j.logger.kafka=ERROR,WARN
  log4j.logger.org.apache.zookeeper=ERROR,WARN
  log4j.logger.org.I0Itec.zkclient=ERROR,WARN

  log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
  log4j.appender.CONSOLE.Target=System.out
  log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
  log4j.appender.CONSOLE.layout.ConversionPattern=%d %-5p [%F(%L)] %m%n

  log4j.appender.FILE=org.apache.log4j.RollingFileAppender  
  log4j.appender.FILE.File=./logs/db.log
  log4j.appender.FILE.Append=true
  log4j.appender.FILE.DatePattern='.'yyyy-MM-dd'.log'
  log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
  log4j.appender.FILE.layout.ConversionPattern=%d %-5p [%c(%L)] %m%n
  log4j.appender.FILE.MaxFileSize=10240KB
  log4j.appender.FILE.MaxBackupIndex=1000
  log4j.appender.FILE.Threshold = DEBUG

```

于是排查了几天，期间也参照tomcat官方的说法把tomcat打日志的方式改成log4j，但是后来发现这只是把tomcat的日志按照log4j的形式输出而已，
根本的问题还是没有解决，就是在上述配置的db.log或者out.log中都没有程序中输出的日志。

当看到stackoverflow上得这个[问题](http://stackoverflow.com/questions/24356319/log4j-creates-log-file-but-does-not-write-to-it)后，似乎有了一点眉头：

The output seems to be of the default format that Java's standard logging framework (JUL) would emit.
So, there are two possibilities (that come to mind):
Your code imports java.util.logging.Logger, rather than org.apache.log4j.Logger.
There exists a library of some sort, in your classpath, that intercepts Log4J calls and converts them to JUL calls.

最后一句话引起了我的注意，因为我的lib中还引入了logback-classic-1.1.3.jar,logback-core-1.1.3.jar两个包。

死马当活马医，我把这两个包删除，然后再重启tomcat,结果DEBUG信息就正常写入了db.log中。

参考的资料有：
[log4j和logback的冲突导致日志输出异常](http://www.tuicool.com/articles/VF32uye)
[JAVA日志组件系列(三)log4j+logback+slf4j的关系与调试](http://phl.iteye.com/blog/2021461)
[Logback浅析](http://www.cnblogs.com/yongze103/archive/2012/05/05/2484753.html)

第二篇文章提到了这点：
只有logback配置文件即可，因为log4j的输出已经委托给了slf4j（通过log4j-over-slf4j），而slf4j的默认实现是logback。
