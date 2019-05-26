title: javascript基础知识之bind
date: 2019-05-25 13:19:16
tags: javascript
---

## JavaScript bind方法使用

```js
    var obj = {
      myFun:function(){
         console.log(this.name);
      }
    }
    var db = {
       name :"john"
    }
　  obj.myFun.bind(db)();　　　//john
```
以上出了bind 方法后面多了个 () 外 ，结果返回都一致！
由此得出结论，bind 返回的是一个新的函数，你必须调用它才会被执行

## JavaScript bind多次绑定

昨天看到掘金小册 [前端面试之道](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdc715f6fb9a049c15ea4e0)时 看到bind的特殊用法，之前不知道有这么一回事，所以在此记录下.
下面转载自原文。

如果对一个函数进行多次 bind，那么上下文会是什么呢？

```js
    let a = {}
    let fn = function () { console.log(this) }
    fn.bind().bind(a)() // => ?
```

如果你认为输出结果是 a，那么你就错了，其实我们可以把上述代码转换成另一种形式

```js
    // fn.bind().bind(a) 等于
    let fn2 = function fn1() {
    return function() {
      return fn.apply()
    }.apply(a)
    }
    fn2()
```

可以从上述代码中发现，不管我们给函数 bind 几次，fn 中的 this 永远由第一次 bind 决定，所以结果永远是 window。

```js
    let a = { name: 'yck' }
    function foo() {
    console.log(this.name)
    }
    foo.bind(a)() // => 'yck'
```


参考资料：

1. [前端面试之道](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdc715f6fb9a049c15ea4e0)
