title: angular_decorator
date: 2017-03-28 20:56:23
tags: angular
---


## angular decorator装饰器

先上段代码：  

```js
angular.module('myapp',[]).config(function($provide){
      $provide.decorator('$log',function($delegate){
        angular.foreach(['log','debug','info','warn','error'],function(o){
                      $delegate[o] = decoratorLogger($delegate[o]);
        });
        function decoratorLogger(orignalFn){
                      return function(){
                          var args = Array.prototype.slice.apply(arguments);
                          args.unshift(new Date().toISPString());
                          originalFn.apply(null,args);
                      };
                  }
        });
  });  
```  

config方法之前也分析过,定义如下：  
```js
/*
      返回一个方法
       方法内部调用configBlocks['push'](['$injector','invoke',arguments])
       然后返回moduleInstance

       config就是一个返回moduleInstance的函数
     */
     var config = invokeLater('$injector', 'invoke', 'push', configBlocks);

     /**
        * @param {string} provider
        * @param {string} method
        * @param {String=} insertMethod
        * @returns {angular.Module}
        */
       function invokeLater(provider, method, insertMethod, queue) {
         if (!queue) queue = invokeQueue;
         return function() {
           queue[insertMethod || 'push']([provider, method, arguments]);
           return moduleInstance;
         };
       }
```  

decorator方法定义如下：  

```js
function decorator(serviceName, decorFn) {
 var origProvider = providerInjector.get(serviceName + providerSuffix),
     orig$get = origProvider.$get;//是个字符串和函数的数组

 origProvider.$get = function() { //相当于重新定义了原来的$get方法
   var origInstance = instanceInjector.invoke(orig$get, origProvider);
   return instanceInjector.invoke(decorFn, null, {$delegate: origInstance});
 };
}
```  

$log是在`angularModule('ng', ['ngLocale'], ['$provide', function ngModule($provide) {}])`中配置进去的

```js
  $provide.provider({
     $log: $LogProvider,
  })
```  

$LogProvider在line 13900,定义如下：
```js
function $LogProvider() {
var debug = true,
    self = this;
    this.$get = ['$window', function($window) {}]
  }
```

最终都是调用instanceInjector的getService方法获取
```js
protoInstanceInjector =
          createInternalInjector(instanceCache, function(serviceName, caller) {

            //去providerCache里去找
            var provider = providerInjector.get(serviceName + providerSuffix, caller);
            return instanceInjector.invoke(
                provider.$get, provider, undefined, serviceName);
            //$get 各个provider定义的$get 方法
          })
```
