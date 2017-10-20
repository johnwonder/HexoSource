title: linux_php
date: 2017-03-25 22:16:23
tags: php
---

## ubuntu14.04 安装php

###先安装apache2:

`sudo apt-get install apache2`  

### 安装php5:

`sudo apt-get install php5`

### 安装mysql:  

`sudo apt-get install mysql-server`  

### 查看ip地址：

`ifconfig -a `

### 查看ssh是否启动

`sudo ps -e |grep ssh`


### ubuntu 终端修改中文乱码  

修改Ubuntu的配置文件/etc/default/locale
将原来的配置内容修改为
LANG=”en_US.UTF-8″
LANGUAGE=”en_US:en”
再在终端下运行：
$ locale-gen -en_US:en
注销或重启后，Ubuntu Server真正服务器实体终端就恢复成了英文的语言环境。
所以，此方法不是真正意义上的中文化，而是恢复英文的默认编码

参考资料：
[Ubuntu Server 命令行下的默认语言 中文乱码 ](http://blog.csdn.net/dufufd/article/details/51201634)
