title: angular_service_injector
date: 2017-07-01 22:13:53
tags: angular
---

## angular service注入解析

### setupModuleLoader

setupModuleLoader方法中定义了module的service方法

```js
/**
           * @ngdoc method
           * @name angular.Module#service
           * @module ng
           * @param {string} name service name
           * @param {Function} constructor A constructor function that will be instantiated.
           * @description
           * See {@link auto.$provide#service $provide.service()}.
           */
    service: invokeLaterAndSetModuleName('$provide', 'service'),
```

比如我写了个方法如下：
```js
var herModule = angular.module('herModule', []);
herModule.service('herService', function() {
    this.her = 1;
});
var injector = angular.injector(["herModule"]);
```

### runInvokeQueue

调用angular.injector([])时，就会调用

```js
function runInvokeQueue(queue) {
      var i, ii;
      for (i = 0, ii = queue.length; i < ii; i++) {
        var invokeArgs = queue[i],
            provider = providerInjector.get(invokeArgs[0]);
            //getService 通过providerCache寻找
               //console.log(invokeArgs[0]); -- $injector  $controllerProvider
        //console.log('invokeArgs[2][0]:'+invokeArgs[2][0]);
        //console.log(invokeArgs[1]);  -- invoke
        provider[invokeArgs[1]].apply(provider, invokeArgs[2]);

      }
    }
```

### $provide.service

```js
function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }

//返回一个方法而已
  function enforceReturnValue(name, factory) {
    return function enforcedReturnValue() {
      var result = instanceInjector.invoke(factory, this);
      if (isUndefined(result)) {
        throw $injectorMinErr('undef', "Provider '{0}' must return a value from $get factory method.", name);
      }
      return result;
    };
  }

  function factory(name, factoryFn, enforce) {
    return provider(name, {
      $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
    });
  }

  function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
  }
```

其中最终调用的就是provider方法把service放入providerCache

先调用service方法放入invokeQueue,然后在angular.injector(["herModule"])放入providerCache,然后在get的时候从providerCache中取出。
