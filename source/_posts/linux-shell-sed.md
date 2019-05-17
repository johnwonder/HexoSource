title: linux shell命令之sed
date: 2018-02-28 17:32:48
tags: linux
---

## linux命令学习之sed

今天在看[beego](https://beego.me/)工具[bee](https://github.com/beego/bee)源码的时候，
看到它的makefile文件开头有这么一段命令

```
VERSION = $(shell grep 'const version' cmd/commands/version/version.go | sed -E 's/.*"(.+)"$$/v\1/')
```

动脑思考了下它的含义，总结如下：
调用shell命令grep 查找cmd/commands/version/version.go文件中的const version ，我们看到
这个文件中的const version 一行为```const version = "1.9.1" ```
然后调用sed命令把这个文本替换为v1.9.1 ，最后赋值给VERSION变量
sed -e命令就是多点编辑命令，比如 sed -e '/v1/v2/'就是把v1替换为v2

sed元字符\(..\) 匹配子串，保存匹配的字符，如s/\(love\)able/\1rs，loveable被替换成lovers。
相当于正则表达式

参考资料:
[sed命令](http://man.linuxde.net/sed)
[Linux sed命令详解](https://www.cnblogs.com/ftl1012/p/sed.html)
