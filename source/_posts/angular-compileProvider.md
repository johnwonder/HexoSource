title: angular编译服务compileProvider
date: 2017-07-03 17:04:32
tags: angular
---

## compileProvider是什么时候注入的

在调用publishExternalAPI的时候注入的：

```js
angularModule('ng', ['ngLocale'], ['$provide',
  function ngModule($provide) {
    // $$sanitizeUriProvider needs to be before $compileProvider as it is used by it.
    $provide.provider({
      $$sanitizeUri: $$SanitizeUriProvider
    });
    //用 injector.invoke时候放入providerCache
    $provide.provider('$compile', $CompileProvider)
      .directive({

      });
  });
```

实际就是执行的```function module(name, requires, configFn) {}```方法

configFn参数会在调用的时候就执行
```js
//configFn有可能就是['$provide',function($provide){}]
        //config 就是调用configBlocks[insertMethod || 'push']([provider, method, arguments]);
        //这里的arguments就是configFn
  if (configFn) {
          config(configFn);
  }
 var config = invokeLater('$injector', 'invoke', 'push', configBlocks);


 function invokeLater(provider, method, insertMethod, queue) {
          if (!queue) queue = invokeQueue;
          return function() {
            queue[insertMethod || 'push']([provider, method, arguments]);
            return moduleInstance;
          };
 }
```

queue就是configBlocks.

在runInvokeQueue中就会从providerInjector中获取$injector
```js
function runInvokeQueue(queue) {
      var i, ii;
      for (i = 0, ii = queue.length; i < ii; i++) {
        var invokeArgs = queue[i],
            provider = providerInjector.get(invokeArgs[0]);//此处就是$injector

        provider[invokeArgs[1]].apply(provider, invokeArgs[2]);
        // invokeArgs[2]此处就是传入的参数数组
        //['$provide',function ngModule($provide) {}]
      }
    }
```

providerCache在定义的时候就加入了$injector

```js
providerInjector = (providerCache.$injector =
       createInternalInjector(providerCache, function(serviceName, caller) {
         if (angular.isString(caller)) {
           path.push(caller);
         }
         throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
}))
```

$injector在invoke参数数组的时候去providerCache里找到$provide并且调用它的provider方法
```js
function invoke(fn, self, locals, serviceName) {
      if (typeof locals === 'string') {
        serviceName = locals;
        locals = null;
      }

      //注入参数 $provide
      var args = injectionArgs(fn, locals, serviceName);
      if (isArray(fn)) { //比如['$scope',function($scope){}]
        fn = fn[fn.length - 1];//取数组的最后一个
      }

      if (!isClass(fn)) {
        // http://jsperf.com/angularjs-invoke-apply-vs-switch
        // #5388
        //比如 var injector = angular.injector(["myModule"]);
        //injector.invoke(function(myService){alert(myService.my);});
        return fn.apply(self, args);
      } else {
        args.unshift(null);
        return new (Function.prototype.bind.apply(fn, args))();
      }
    }
```
