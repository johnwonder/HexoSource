title: angular_$provide
date: 2017-04-17 11:30:02
tags: angular
---

## angular provide源码解析

在createInjector中会调用loadModules方法，loadModules方法中调用如下：

```js
//遍历modulesToLoad数组
 forEach(modulesToLoad, function(module) {
        if (isString(module)) {
          moduleFn = angularModule(module);
          runBlocks = runBlocks.concat(loadModules(moduleFn.requires)).concat(moduleFn._runBlocks);
          runInvokeQueue(moduleFn._invokeQueue);
          //像module.service时加入的service ，那么需要把这些service加入prividerCache
          runInvokeQueue(moduleFn._configBlocks);
        } else if (isFunction(module)) {
            runBlocks.push(providerInjector.invoke(module));
        } else if (isArray(module)) {
          // $provide.value('$rootElement', element);
          //为undefined
          //invoke里会调用function
            runBlocks.push(providerInjector.invoke(module));
        } else {
          assertArgFn(module, 'module');
        }
    }
```

还记得bootstrap中调用是怎么样的么？

```js
   modules = modules || [];
   //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
   modules.unshift(['$provide', function($provide) {
     $provide.value('$rootElement', element);
   }]);
```

那么就会先调用`providerInjector.invoke`然后再push到runBlocks中，$provide定义如下：

```js
function valueFn(value) {return function valueRef() {return value;};}

function createInjector(modulesToLoad, strictDi) {
var providerCache = {
     $provide: {
         provider: supportObject(provider),
         factory: supportObject(factory),
         service: supportObject(service),
         value: supportObject(value),
         constant: supportObject(constant),
         decorator: decorator
       }
   };

   function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      //先实例化
      provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }

   function factory(name, factoryFn, enforce) {
       return provider(name, {
         $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
       });
     }

     function value(name, val) { return factory(name, valueFn(val), false); }
 }
```

看instantiate方法：

```js
function instantiate(Type, locals, serviceName) {
    // Check if Type is annotated and use just the given function at n-1 as parameter
    // e.g. someModule.factory('greeter', ['$window', function(renamed$window) {}]);
    var ctor = (isArray(Type) ? Type[Type.length - 1] : Type);
    var args = injectionArgs(Type, locals, serviceName);
    // Empty object at position 0 is ignored for invocation with `new`, but required.
    args.unshift(null);
    return new (Function.prototype.bind.apply(ctor, args))();
  }
```
