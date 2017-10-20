title: html的base节点
date: 2017-06-14 17:24:42
tags: JavaScript
---
这可是高科技了。举个例子。
假如你这个页面的a标签是这样写的；
<a href="xxx.html">text link</a>
默认就是本页面打开；
但是你加了
<base target=_blank />

那这个链接就是新页面打开

再比如：你的a标签还是这么写的

<a href="xxx.html">text link</a>

默认打开的网址就是现在的相对路径下的xxx.html
假如你加了
<base href="http://www.baidu.com" />
那你刚才的那个链接打开的地址就是
http://www.baidu.com/xxx.html
base这个东西有点编程语言里的全局变量；
如果你要改变这个也是可以的。就是在你要改变的A标签里重写它定义的属性，比如刚才的链接你写成这样<a href="xxx.html" target="_self">text link</a>
即使你定义了<base target="_blank" /> 这个加了target的A链接依然是当前页面打开；

转载自：
[在html中base的作用是什么](https://zhidao.baidu.com/question/495713634816948284.html)
