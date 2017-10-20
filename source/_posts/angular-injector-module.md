title: angular_injector_module
date: 2017-03-18 15:49:16
tags: angular
---

## angular 1.5.8源码解读

有如下一段代码：
```js
// 创建myModule模块、注册服务
var myModule = angular.module('myModule', []);
myModule.service('myService', function() {
    this.my = 0;
});
// 创建herModule模块、注册服务
var herModule = angular.module('herModule', []);
herModule.service('herService', function() {
    this.her = 1;
});
// 加载了2个模块中的服务
var injector = angular.injector(["myModule","herModule"]);
alert(injector.get("myService").my);
alert(injector.get("herService").her);
```

我们先看看service方法是如何定义的  

```js

var moduleInstance = {
  //service
  service: invokeLaterAndSetModuleName('$provide', 'service')
}

/**
 * @param {string} provider
 * @param {string} method
 * @returns {angular.Module}
 */
function invokeLaterAndSetModuleName(provider, method) {
  return function(recipeName, factoryFunction) {
    if (factoryFunction && isFunction(factoryFunction)) factoryFunction.$$moduleName = name;
    invokeQueue.push([provider, method, arguments]);
    return moduleInstance;
  };
}
```  
也就是如果加入了个module，那么module.service就是
```js
module.service = function(recipeName, factoryFunction) {
  if (factoryFunction && isFunction(factoryFunction))
      factoryFunction.$$moduleName = name;

  //arguments就是 recipeName 和factoryFunction的数组
  invokeQueue.push([provider, method, arguments]);
  return moduleInstance;
};
```  

invokeQueue就push 进去了```$provide service recipeName, factoryFunction```  
provider定义的时候 定义了service方法:   

```js
$provide: {
          provider: supportObject(provider),//supportObject返回了
          factory: supportObject(factory),
          service: supportObject(service),
          value: supportObject(value),
          constant: supportObject(constant),
          decorator: decorator
        }
```  

supportObject方法:    
```js
  function supportObject(delegate) {
    return function(key, value) {
      if (isObject(key)) {
        forEach(key, reverseParams(delegate));
      } else {
        return delegate(key, value);
      }
    };
  }
```  
$provide.service 就是```function(key,value){ }```
如果是service的话，那么此处的delegate就是外部定义的service方法：  
```js
  function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
  }
```  

再按一定的顺序调用如下方法：
```js
 //factoryFn 就是['$injector',function(){$injector}{}]
 function factory(name, factoryFn, enforce) {
   return provider(name, {
     $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
   });//注意 enforceReturnValue只是返回一个方法而已
   //如果enforce 为true 或者undefined
   //那么factoryFn 最终 被instanceInjector.invoke(factory, this);
 }

  //provider_就是上面定义的{ $get: function }对象
  function provider(name, provider_) {
     assertNotHasOwnProperty(name, 'service');
     if (isFunction(provider_) || isArray(provider_)) {
       provider_ = providerInjector.instantiate(provider_);
     }
     if (!provider_.$get) {//必须含有$get方法
       throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
     }
     return providerCache[name + providerSuffix] = provider_;
     //probider_ 定义了 { $get:  enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn }
     //enforceReturnValue 返回了 一个调用factoryFn的方法
     //相当于调用$get的时候调用
     //instanceInjector.invoke(factory)
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
```



我们来看看``` angular.injector(["myModule","herModule"]);```angular内部是如何调用的：

首先angular.injector就是调用的createInjector方法  

createInjector方法内部有句```var runBlocks = loadModules(modulesToLoad);```  

```js
////////////////////////////////////
// Module Loading
////////////////////////////////////
function loadModules(modulesToLoad) {
  //判断是否是数组
  assertArg(isUndefined(modulesToLoad) || isArray(modulesToLoad), 'modulesToLoad', 'not an array');
  var runBlocks = [], moduleFn;
  //遍历modulesToLoad数组
  forEach(modulesToLoad, function(module) {
    if (loadedModules.get(module)) return;
    loadedModules.put(module, true);

    function runInvokeQueue(queue) {
      var i, ii;
      for (i = 0, ii = queue.length; i < ii; i++) {
        var invokeArgs = queue[i],
            provider = providerInjector.get(invokeArgs[0]);//invokeArgs[0]就是$provide

          //invokeArgs[1]就是service
          //invokeArgs[2]就是 recipeName factoryFunction的数组
        provider[invokeArgs[1]].apply(provider, invokeArgs[2]);
      //  provider['service'] 就是调用```provider[invokeArgs[1]].apply(provider, invokeArgs[2]);```  
      //invokeArgs[2]就是一个数组，传入了service的key,value


  //      function(key, value) {
  //          if (isObject(key)) {
  //            forEach(key, reverseParams(delegate));
  //          } else {
  //            return delegate(key, value);
  //          }
  // }
      }
    }

    try {
      if (isString(module)) {
        moduleFn = angularModule(module);
        runBlocks = runBlocks.concat(loadModules(moduleFn.requires)).concat(moduleFn._runBlocks);
        runInvokeQueue(moduleFn._invokeQueue);//_invokeQueue是在定义service 等时候push进去的
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
      if (e.message && e.stack && e.stack.indexOf(e.message) == -1) {
        // Safari & FF's stack traces don't contain error.message content
        // unlike those of Chrome and IE
        // So if stack doesn't contain message, we create a new string that contains both.
        // Since error.stack is read-only in Safari, I'm overriding e and not e.stack here.
        /* jshint -W022 */
        e = e.message + '\n' + e.stack;
      }
      throw $injectorMinErr('modulerr', "Failed to instantiate module {0} due to:\n{1}",
                module, e.stack || e.message || e);
    }
  });
  return runBlocks;
}
```  

service调用如下：  

```js
//当调用injector.get("myService").my时

var provider = providerInjector.get(serviceName + providerSuffix, caller);
return instanceInjector.invoke(provider.$get, provider, undefined, serviceName);
//此处的provider.$get 就是如下方法


function enforcedReturnValue() {
      var result = instanceInjector.invoke(factory, this);
      if (isUndefined(result)) {
        throw $injectorMinErr('undef', "Provider '{0}' must return a value from $get factory method.", name);
      }
      return result;
    }

//factory方法就是
['$injector', function($injector) {return $injector.instantiate(constructor);}]

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
