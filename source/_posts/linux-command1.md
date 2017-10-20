title: linux_command1
date: 2017-03-14 17:12:29
tags: linux
---

##  ps相关

`ps -ef|grep java` 筛选出跟java相关的进程  

`ps -u -p 22350` 查看进程ID为22350的详细信息```  

##　nohup相关

`nohup java -jar -Xmx2048m  /home/appserver/apiserver/apiserver.jar`

参考资料：  

[Linux 查找指定名称的进程并显示进程详细信息](http://blog.csdn.net/hongweigg/article/details/44828353)  
[linux -- 进程的查看、进程id的获取、进程的杀死](http://www.cnblogs.com/hf8051/p/4494735.html)  

[linux的nohup命令的用法。](http://www.cnblogs.com/allenblogs/archive/2011/05/19/2051136.html)  
[linux下利用nohup后台运行jar文件包程序](http://blog.csdn.net/tang9140/article/details/38899345)
[Linux Top 命令解析 比较详细](http://www.jb51.net/LINUXjishu/34604.html)

[pmap : 理解linux的进程内存占用 ](http://blog.csdn.net/qiuxin315/article/details/6828467)
