title: angular绑定函数
date: 2017-01-14 15:10:23
tags: angular
---

## angular.bind函数分析

解释：返回一个调用self的函数fn（self代表fn里的this）.可以给fn提供参数args.这个功能也被称为局部操作，以区别功能.  
格式：angular.bind(self,fn,args);  
self：object  对象； fn的上下文对象，在fn中可以用this调用  
fn：function； 绑定的方法  

```js

  var obj = { name: "Any" };
  var fn = function (Adj) {
    console.log(this.name + "is a boy!!! And he is " + Adj + " !!!");
  };
  var f = angular.bind(obj, fn, "handsome");
  f();//Any is a boy!!! And he is handsome!!!
  var t = angular.bind(obj, fn);
  t("ugly");// Any is a boy!!! And he is ugly!!!
```

上源码:  

```js

  function concat(array1, array2, index) {
    return array1.concat(slice.call(array2, index));
  }

  function sliceArgs(args, startIndex) {
    return slice.call(args, startIndex || 0);
  }

  function bind(self, fn) {
  var curryArgs = arguments.length > 2 ? sliceArgs(arguments, 2) : [];
  //判断是否是函数 且不能是正则类型
  if (isFunction(fn) && !(fn instanceof RegExp)) {
    return curryArgs.length
      ? function() {
          return arguments.length
            ? fn.apply(self, concat(curryArgs, arguments, 0))
            : fn.apply(self, curryArgs);
        }
      : function() {
          return arguments.length
            ? fn.apply(self, arguments)
            : fn.call(self);
        };
  } else {
    // In IE, native methods are not functions so they cannot be bound (note: they don't need to be).
    return fn;
  }
  }
```
