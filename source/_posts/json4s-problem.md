title: json4s_problem
date: 2017-02-06 13:27:41
tags: scala
---

##  json4s是个什么东东

首先去[官网](http://json4s.org/),看到标题就是One AST to rule them all。  

[AST](https://www.zhihu.com/question/33107553):抽象语法树（Abstract Syntax Tree）也称为AST语法树，指的是源代码语法所对应的树状结构。也就是说，对于一种具体编程语言下的源代码，通过构建语法树的形式将源代码中的语句映射到树中的每一个节点上。  

At this moment there are at least 6 json libraries for scala, not counting the java json libraries. All these libraries have a very similar AST. This project aims to provide a single AST to be used by other scala json libraries.  
也就是说这个项目是要提供一个抽象语法树给其他scala的json库使用

At this moment the approach taken to working with the AST has been taken from lift-json and the native package is in fact lift-json but outside of the lift project.  
当前可以与这个AST一起使用的是从lift-json借过来的。

##  Jackson
In addition to the native parser there is also an implementation that uses jackson for parsing to the AST. The jackson module includes most of the jackson-module-scala functionality and the ability to use it with the lift-json AST.  

可以用jackson去解析AST.
