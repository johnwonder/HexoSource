title: angular1.5.8源码解析之decorator原理
date: 2017-03-28 20:56:23
tags: angular
---


## angularjs装饰器的使用

我们定义一个模块，并且配置一个configBlocks

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

  //调用log函数加上时间戳
  angular.element(document).injector().get('$log').log('hello')
```

此处可以有几个疑问：
1. config函数做了什么?
2. decorator函数又做了什么?
3. angular.element(document)为什么能获取injector?

### config函数
```js
      /*
       方法内部调用configBlocks['push'](['$injector','invoke',arguments])
       然后返回moduleInstance，config就是一个返回moduleInstance的函数
     */
     var config = invokeLater('$injector', 'invoke', 'push', configBlocks);
     //此处就是传入配置块
     if (configFn) {
          config(configFn);
     }
     function invokeLater(provider, method, insertMethod, queue) {
         if (!queue) queue = invokeQueue;
         return function() {
           queue[insertMethod || 'push']([provider, method, arguments]);
           return moduleInstance;
         };
       }

```
模块的config函数就是把参数configBlocks放入configBlocks数组中，然后在loadModules的时候
调用runInvokeQueue，获取$injectorProvider也就是instanceInjector后调用invoke函数,
传入```function($provide){}```函数体，最后执行这个函数体。

### $provide.decorator函数  

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

### $LogProvider
```js
  $provide.provider({
     $log: $LogProvider,
  });

  function $LogProvider() {
  var debug = true,
      self = this;
      this.$get = ['$window', function($window) {

          return {
            log:consoleLog('log')
            //省略部分代码
          }
      }]
    }
```  
执行decorator方法的时候先获取原来的$logProvider和它的$get属性，然后重新赋值$logProvider的
$get属性，把原来的$get属性放入新的$get函数内。

最终调用instanceInjector的getService方法获取，内部先从providerCache中获取后，再调用
对应$logProvider的新的$get函数，函数内部首先调用原先保存的旧的$get属性，获取原来的```{log:consoleLog('log')}```,
然后调用decorFn,传入```{$delegate: origInstance}```,最后就是对原来的$log实例的属性函数做了修改。

装饰器设计模式就是如此奇妙！

文章末尾彩蛋：
### $rootElement.data函数

在angular调用```doBootstrap```启动函数的时候，内部会调用```element.data('$injector', injector);```
把创建好的instanceInjector放入element的data缓存中，所以能直接调用injector()函数，此部分的
源码设计我们会在之后的文章中进行剖析。
