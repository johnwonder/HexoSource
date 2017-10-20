title: angular源码分析之scope步步解析
date: 2017-08-02 11:31:28
tags: angular
---

看了[Scopes and DirtyChecking](https://teropa.info/build-your-own-angular/build_your_own_angularjs_sample.pdf)这篇文章深有体会，所以打算简单翻译一下以做记录。

## Scope对象

Scopes能通过Scope的构造函数new来创建。结果就是一个简单的javascript对象。先创建一个简单的例子。

```js
function Scope() {
}
var scope = new Scope();
scope.aProperty = 1;
```

## Watching对象属性：$watch和$digest

$watch和$digest是同一个硬币的俩面，他们一起承载消费周期的核心：反应数据的变化

通过$watch你可以添加一个watcher到scope对象上。当scope上有变化发生时watcher会收到通知。

你可以通过两个函数参数调用$watch 函数:

1.一个watch函数，可以指定你感兴趣的数据。
2.一个listener函数，当数据改变时会被调用。

在angular里，你通常会指定一个watch表达式来代替watch函数

一个watch表达式可以是一个字符串，像"user.firstName"，可以放在数据绑定，指令属性或者javascript代码里。

它在angualr内部被解析和编译成watch函数。

另外一个是$digest函数，它会遍历scope上所有的watchers,运行他们的watch和listener函数

Scope首先需要存储所有被注册的watchers：

```js
function Scope() {
 this.$$watchers = [];
}

Scope.prototype.$watch = function(watchFn,listenerFn){
  var watcher = {
     watchFn: watchFn,
     listenerFn:listenerFn
  };
  this.$$watchers.push(watcher);
}
```

最后有个$digest函数，让我们定义个简单版本，只是遍历所有注册过的watchers，然后调用他们的listener函数。

```js
Scope.prototype.$digest = function(){
  _.foreach(this.$$watchers,function(watcher){
      watcher.listenerFn();
  });
};
```

我们也能看到angular scopes有重要的性能指标：
1.不是通过scope附加的data在性能上必须考虑。如果没有watcher监控属性，它就不关心是否是在scope上。angular不会遍历scope上的所有属性，它只会遍历watchers。
2.美一个watch函数在每一个$digest调用中被调用。基于这个原因，得考虑你拥有的watches,还有你的watch函数和表达式的性能。
