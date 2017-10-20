title: linux_vi1
date: 2017-04-11 16:08:09
tags: linux
---

## linux vi学习笔记1

:wq 保存文件并退出vi

:w 保存文件但不退出vi

:q 不保存文件，退出vi

:$ 跳到文件最后一行

1 按大写的G 跳到最后一行。 然后按小写的O键，增加一行。

2 也可以：
搜索
:set nu (设置行号)
:14 （这里只是示例，是指的最后一行的行号）
o

3 输入 $ 然后按大写的O键 向上增加一行
:$
O

参考资料：
[linux vi 在最后一行增加一行](https://zhidao.baidu.com/question/583768034414660125.html)
[vi/vim 删除：一行, 一个字符, 单词, 每行第一个字符 命令  ](http://blog.163.com/chen_dawn/blog/static/112506320111145955649/)
[每天一个Linux命令（48）ping命令](http://www.cnblogs.com/MenAngel/p/5586677.html)
