title: sbt linux shell脚本解读
date: 2017-06-14 21:20:36
tags: scala
---

## set -e

你写的每个脚本都应该在文件开头加上set -e,这句语句告诉bash如果任何语句的执行结果不是true则应该退出。这样的好处是防止错误像滚雪球般变大导致一个致命的错误，而这些错误本应该在之前就被处理掉。如果要增加可读性，可以使用set -o errexit，它的作用与set -e相同。

参考资料：
[Unix/Linux 脚本中 “set -e” 的作用](http://blog.csdn.net/todd911/article/details/9954961)

## export

set:显示(设置)shell变量 包括的私有变量以及用户变量，不同类的shell有不同的私有变量 bash,ksh,csh每中shell私有变量都不一样

env:显示(设置)用户变量变量

export:显示(设置)当前导出成用户变量的shell变量。
参考资料：
[ shell环境变量以及set,env,export的区别 ](http://blog.csdn.net/longxibendi/article/details/6125075)
[set,env和export这三个命令的区别 ](http://blog.csdn.net/u013176681/article/details/49097641)

```linux
#!/usr/bin/env bash
#
# A more capable（胜任的） sbt runner, coincidentally（巧合） also called sbt.
# Author: Paul Phillips <paulp@typesafe.com>

# todo - make this dynamic
declare -r（只读） sbt_release_version=0.12.0（sbt发布版本）
declare -r sbt_snapshot_version=0.13.0-SNAPSHOT

unset(unset为shell内建指令，可删除变量或函数) sbt_jar sbt_dir sbt_create sbt_snapshot sbt_launch_dir
unset scala_version java_home sbt_explicit_version
unset verbose debug quiet
```

## /usr/bin/env

在linux的一些脚本里，需在开头一行指定脚本的解释程序，如：
```#!/usr/bin/env python```

再如：
```linux
#!/usr/bin/env perl
#!/usr/bin/env zimbu
```

但有时候也用
```linux
#!/usr/bin/python
```

和
```linux
#!/usr/bin/perl
```

那么 env到底有什么用？何时用这个呢？
脚本用env启动的原因，是因为脚本解释器在linux中可能被安装于不同的目录，env可以在系统的PATH目录中查找。同时，env还规定一些系统环境变量。

执行一下 env 命令后看看打印的内容

可以用env来执行程序：

参考资料：
[ 使用/usr/bin/env的好处 ](http://blog.chinaunix.net/uid-26495963-id-3409921.html)
[#!/usr/bin/env bash和#!/usr/bin/bash的比较](http://blog.csdn.net/austin_zhou001/article/details/46591169)
[Linux set unset命令](http://blog.csdn.net/carolzhang8406/article/details/21246145)
