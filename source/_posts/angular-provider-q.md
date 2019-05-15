title: angular源码剖析之Provider系列--QProvider
date: 2019-05-14 09:44:14
tags: angular
---

## QProvider 简介

源码里是这么描述的：

A service that helps you run functions asynchronously, and use their return values (or exceptions) when they are done processing.

大概意思是帮助你异步执行方法，且当他们执行完后可以使用他们的返回值。

This is an implementation of promises/deferred objects inspired by Kris Kowal's Q.

这是一个 promises/deferred 对象的实现,灵感来自于 [Kris Kowal's Q](https://github.com/kriskowal/q)

## QProvider 用法

下面的例子假设$q和asyncGreet在当前作用域内是有效的.

```js
    function asyncGreet(name) {
    // perform some asynchronous operation, resolve or reject the promise when appropriate.
    return $q(function(resolve, reject) {
      setTimeout(function() {
        if (okToGreet(name)) {
          resolve('Hello, ' + name + '!');
        } else {
          reject('Greeting ' + name + ' is not allowed.');
        }
      }, 1000);
    });
    }

    var promise = asyncGreet('Robin Hood');
    //then函数放入pending数组
    promise.then(function(greeting) {
    alert('Success: ' + greeting);
    }, function(reason) {
    alert('Failed: ' + reason);
    });
```

下面我们深入源码内部去一探究竟：

## $QProvider 定义

```js
  function $QProvider() {

    this.$get = ['$rootScope', '$exceptionHandler', function($rootScope, $exceptionHandler) {
      return qFactory(function(callback) {
        $rootScope.$evalAsync(callback);
      }, $exceptionHandler);
    }];
  }

  $evalAsync: function(expr, locals) {
       // if we are outside of an $digest loop and this is the first time we are scheduling async
       // task also schedule async auto-flush
       //如果当前不处于$digest或者$apply的过程中(只有在$apply和$digest方法中才会设置$$phase这个字段)，并且asyncQueue数组中还不存在任务时，
       //就会异步调度一轮digest循环来确保asyncQueue数组中的表达式会被执行
       if (!$rootScope.$$phase && !asyncQueue.length) {
         $browser.defer(function() {
           //最终调用的是setTimeout
           if (asyncQueue.length) {
             $rootScope.$digest();//执行消化功能
           }
         });
       }
       asyncQueue.push({scope: this, expression: $parse(expr), locals: locals});
     }
```

由之前 [cacheFactory的分析](http://johnwonder.coding.me/2018/03/01/angular-provider-cachefactory/)
，再结合上面源码我们就知道  注入$q时调用了qFactory工厂方法：

## qFactory

```js
  function qFactory(nextTick, exceptionHandler) {

      function Promise() {
      //初始化Promise的状态对象
      this.$$state = { status: 0 };
    }
    //扩展Promise类原型
    extend(Promise.prototype, {
      //then主要是把一个defer对象和fullfiled reject 函数 放入pending数组
      then: function(onFulfilled, onRejected, progressBack) {
        if (isUndefined(onFulfilled) && isUndefined(onRejected) && isUndefined(progressBack)) {
          return this;
        }
        var result = new Deferred();

        this.$$state.pending = this.$$state.pending || [];
        //把一个新的 Defer 对象push进pending数组
        this.$$state.pending.push([result, onFulfilled, onRejected, progressBack]);

        if (this.$$state.status > 0) scheduleProcessQueue(this.$$state);

        //返回这个新建Defer对象的promise
        //可以形成promise chain
        return result.promise;
      }
      //代码省略
    }
    //通过$q注入时返回Q函数
    //随后Q函数 传入resolver参数调用
    //调用resolver函数时传入包装deferred对象的resolve和 reject函数
    //随后返回promise对象
    //promise对象调用then函数时放入pending队列
    var $Q = function Q(resolver) {
       if (!isFunction(resolver)) {
         throw $qMinErr('norslvr', "Expected resolverFn, got '{0}'", resolver);
       }
       //构造一个Deferred对象
       var deferred = new Deferred();
       function resolveFn(value) {
         deferred.resolve(value);
       }
       function rejectFn(reason) {
         deferred.reject(reason);
       }
        //调用resolver参数函数
        //resolveFn供外部调用
       resolver(resolveFn, rejectFn);
       //返回一个promise对象
       //供调用then函数
       return deferred.promise;
     };
      // Let's make the instanceof operator work for promises, so that
      // `new $q(fn) instanceof $q` would evaluate to true.
      //使得new $q(fn) 也可以调用promise的方法
      $Q.prototype = Promise.prototype;

      //暴露内部方法
      $Q.defer = defer;
      $Q.reject = reject;
      $Q.when = when;
      $Q.resolve = resolve;
      //$q.all是用于执行多个异步任务进行回调，它可以接受一个promise的数组，
      //或是promise的hash(object)。任何一个promise失败，都会导致整个任务的失败。
      //https://blog.csdn.net/shidaping/article/details/52398925
      $Q.all = all;
      //$q.race() 是 Angular 里面的一个新方法，和 $q.all() 类似，但是它只会返回第一个处理完成的 Promise 给你///。假定 API 调用 1 和 API 调用 2 同时执行，而 API 调用 2 在 API 调用 1 之前处理完成，那么你就只会得到 //API 调用 2 的返回对象。换句话说，最快（处理完成）的 Promise 会赢得返回对象的机会：
      $Q.race = race;

      return $Q;
  }
```

调用then方法时实际上是新建一个defer对象放入pending数组，在调用defer.resolve的时候
去调度这个数组中的元素，也就是任务.

## resolve 方法

```js
  extend(Deferred.prototype, {
    resolve: function(val) {
      //第一次resolve的时候为0 所以会往下走
      if (this.promise.$$state.status) return;
      if (val === this.promise) {
        this.$$reject($qMinErr(
          'qcycle',
          "Expected promise to be resolved with value other than itself '{0}'",
          val));
      } else {
        //调度pending数组里的任务
        this.$$resolve(val);
      }
    },

    $$resolve: function(val) {
      var then;
      var that = this;
      var done = false;
      try {
        if ((isObject(val) || isFunction(val))) then = val && val.then;
        if (isFunction(then)) {
           //val.then方法 val是promise的时候
           //resolvePromise函数里放入了当前defer对象
          this.promise.$$state.status = -1;
          then.call(val, resolvePromise, rejectPromise, simpleBind(this, this.notify));
        } else {

          //更新promise的状态对象
          this.promise.$$state.value = val;
          this.promise.$$state.status = 1;
          //调度的时候pending为空就返回了
          scheduleProcessQueue(this.promise.$$state);
        }
      } catch (e) {
        rejectPromise(e);
        exceptionHandler(e);
      }

      function resolvePromise(val) {
        if (done) return;
        done = true;
        that.$$resolve(val);
      }
      function rejectPromise(val) {
        if (done) return;
        done = true;
        that.$$reject(val);
      }
    }
  }
```
### scheduleProcessQueue 方法

```js
  //传入promise的state对象
  function scheduleProcessQueue(state) {
   if (state.processScheduled || !state.pending) return;
   state.processScheduled = true;
   //nextTick里调用$browser.defer函数

   nextTick(function() { processQueue(state); });
  }
```

### processQueue 方法

实际最终处理的还是processQueue函数，里面循环调用pending数组
```js
  function processQueue(state) {
    var fn, deferred, pending;

    pending = state.pending;
    state.processScheduled = false;
    state.pending = undefined;
    for (var i = 0, ii = pending.length; i < ii; ++i) {

      //获取pending数组的元素 元素本身也是数组
      deferred = pending[i][0];
      fn = pending[i][state.status];
      try {
        if (isFunction(fn)) {
          deferred.resolve(fn(state.value));
        } else if (state.status === 1) {
          deferred.resolve(state.value);
        } else {
          deferred.reject(state.value);
        }
      } catch (e) {
        deferred.reject(e);
        exceptionHandler(e);
      }
    }
  }
```

参考资料：

1. [a tiny implementation of Promises/A+.](https://github.com/stefanpenner/es6-promise/)
2. [Angular中的$q的形象解释及深入用法](https://www.cnblogs.com/ZengYunChun/p/6438330.html)
3. [关于 Angular 里的 $q 和 Promise](https://juejin.im/post/58465c4979bc440065c6e720)
4. [理解状态机](https://glumes.com/post/android/understand-state-machine/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
