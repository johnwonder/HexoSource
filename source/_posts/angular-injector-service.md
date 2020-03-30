title: angularjs源码分析--创建service
date: 2020-03-21 19:57:09
tags: angular
---

angular中的service服务方法很多地方都会可以用到，内部其实也是依赖angularjs强大的依赖注入能力。
我们先看昨天文章[模块创建](https://mp.weixin.qq.com/s/2F0Zs9cRiY5AjEC0Ez0oOw)使用的案例：

## angular.module

```js
// 创建myModule模块
var myModule = angular.module('myModule', []);
myModule.service('myService', function() {
			this.my = 0;
});
//创建注入器，把service放入providerCache
var injector = angular.injector(["myModule"]);

function explicit(serviceA) {alert(serviceA.my);};
explicit.$inject = ['myService'];

//获取providerCache，调用构造函数，弹出0.
injector.invoke(explicit);

```

## service

关于angular模块的创建，我们上一篇文章已经讲过，我们就看下模块的service方法,在新建模块后这个模块
实例就拥有了service函数。

```js
  var invokeQueue = [];

  var moduleInstance = {
    _invokeQueue: invokeQueue,
    service: invokeLaterAndSetModuleName('$provide', 'service');
  }

  function invokeLaterAndSetModuleName(provider, method) {
        //实际的service方法 ，通过闭包把$provide 和service 放入返回函数中
        return function(recipeName, factoryFunction) {
          if (factoryFunction && isFunction(factoryFunction)) factoryFunction.$$moduleName = name;
          invokeQueue.push([provider, method, arguments]);
          //比如service 就是 provider= $provide  method = service
          return moduleInstance;
        };
   }
```

那么我们就可以知道在调用service的方法其实就是把$provide,service,还有参数(这边就是arguments)放入invokeQueue中，
从名字看出来这是个调用队列，那就有个疑问了，这个invokeQueue是干啥的呢？ 什么时候会调用这个invokeQueue呢？
那就是createInjector函数中的loadModules函数。

### loadModules

loadModules函数会在createInjector的时候去调用，看看它对service起的作用

```js
    function createInjector(modulesToLoad){

      var runBlocks = loadModules(modulesToLoad);

       //模块加载
      function loadModules(modulesToLoad) {
        assertArg(isUndefined(modulesToLoad) || isArray(modulesToLoad), 'modulesToLoad', 'not an array');
        var runBlocks = [], moduleFn;
        //遍历modulesToLoad数组
        forEach(modulesToLoad, function(module) {
          if (loadedModules.get(module)) return;
          loadedModules.put(module, true);

          //调用invokeQueue
          function runInvokeQueue(queue) {
            var i, ii;
            for (i = 0, ii = queue.length; i < ii; i++) {
              var invokeArgs = queue[i], //invokeArgs也是个数组
                  //获取provider
              provider = providerInjector.get(invokeArgs[0]);
              //调用provider中的方法  
              //比如injectorProvider  调用invoke方法 传入函数
              //函数的参数可以注入
              provider[invokeArgs[1]].apply(provider, invokeArgs[2]);
            }
          }

          try {
            if (isString(module)) {
              //获取module
              //moduleFn就是 moduleInstance
              moduleFn = angularModule(module);
              //moduleFn.requires 比如['ngLocale']
              runBlocks = runBlocks.concat(loadModules(moduleFn.requires)).concat(moduleFn._runBlocks);
              runInvokeQueue(moduleFn._invokeQueue);
              //像module.service时加入的service ，那么需要把这些service加入prividerCache
              runInvokeQueue(moduleFn._configBlocks);
            } else if (isFunction(module)) {
                runBlocks.push(providerInjector.invoke(module));
            } else if (isArray(module)) {
                runBlocks.push(providerInjector.invoke(module));
            } else {
              assertArgFn(module, 'module');
            }
          } catch (e) {
            if (isArray(module)) {
              module = module[module.length - 1];
            }

            throw $injectorMinErr('modulerr', "Failed to instantiate module {0} due to:\n{1}",
                      module, e.stack || e.message || e);
          }
        });
        return runBlocks;
      }
    }
```

我们看到关键一句代码```runInvokeQueue(moduleFn._invokeQueue);```，调用runInvokeQueue函数。

传入我们创建service时push进去的数组元素，最关键的就是这两句代码.

```js
  function runInvokeQueue(){
    provider = providerInjector.get(invokeArgs[0]);
    //调用provider中的方法  
    //比如injectorProvider  调用invoke方法 传入函数
    //函数的参数可以注入
    provider[invokeArgs[1]].apply(provider, invokeArgs[2]);

}
```

invokeArgs[0]就是$provide,invokeArgs[1]就是service,那么实际调用的就是$provide.service函数，参数就是```['myService', function() {
			this.my = 0;
}]```,关于$provide.service函数我们下篇文章再分析。
