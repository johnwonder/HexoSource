title: angular-源码分析之rootScope
date: 2017-07-27 15:49:22
tags: angular
---

## $rootScope 到底是啥

### 出处

在bootstrap函数中会调用injector.invoke，里面用到了rootScope:

```js
injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector',
       function bootstrapApply(scope, element, compile, injector) {
        scope.$apply(function() { //rootScope
          element.data('$injector', injector);
          compile(element)(scope);
        });
      }]
    );
```

从之前的文章中我们可以知道$rootScope从$rootScopeProvider.$get方法返回的，那么我们就可以看$rootScopeProvider.$get方法返回了什么。

### $RootScopeProvider
```js
   function Scope() {
      this.$id = nextUid();
      this.$$phase = this.$parent = this.$$watchers =
                     this.$$nextSibling = this.$$prevSibling =
                     this.$$childHead = this.$$childTail = null;
      this.$root = this;
      this.$$destroyed = false;
      this.$$listeners = {};
      this.$$listenerCount = {};
      this.$$watchersCount = 0;
      this.$$isolateBindings = null;
    }

    Scope.prototype = {
      constructor: Scope,
      $new: function(isolate, parent) {

      },
      //$apply方法就是在这定义的
        //expr 可以为function
      $apply: function(expr) {
        try {
                 beginPhase('$apply');
                 try {
                   return this.$eval(expr);
                 } finally {
                   clearPhase();
                 }
               } catch (e) {
                 $exceptionHandler(e);
               } finally {
                 try {
                   $rootScope.$digest();
                 } catch (e) {
                   $exceptionHandler(e);
                   throw e;
                 }
               }
      }
      //省略
    };
       //定义了Scope对象
   var $rootScope = new Scope();

    //The internal queues. Expose them on the $rootScope for debugging/testing purposes.
    var asyncQueue = $rootScope.$$asyncQueue = [];
    var postDigestQueue = $rootScope.$$postDigestQueue = [];
    var applyAsyncQueue = $rootScope.$$applyAsyncQueue = [];

    var postDigestQueuePosition = 0;

    return $rootScope;
```
