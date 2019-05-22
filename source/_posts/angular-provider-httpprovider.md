title: angular源码剖析之Provider系列-$http
date: 2019-05-22 22:21:52
tags: angular
---

## 前言

虽然现在angular.js已经不怎么流行，也逐渐被react,vue等主流框架取代，但是angular.js中的一些
设计我觉得非常值得学习，我们不仅仅要跟随潮流，也得了解曾经优秀流行的框架是怎么设计的，为什么要这么
设计，这样我觉得学习新技术才会思路更清晰，才能更快上手。

## $http简介

先来看官方文档的简介:
The $http service is a core AngularJS service that facilitates communication with
 the remote HTTP servers via the browser's XMLHttpRequest object or via JSONP.

 $http服务是AngularJs的核心服务，是通过浏览器的XMLHttpRequest对象或者JSONP来跟远程http服务
 进行通信的.也就是说$http内部必然是通过封装的ajax来完成他的设计的。
