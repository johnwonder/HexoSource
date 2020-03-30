title: angular源码分析之依赖注入--注解
date: 2017-03-15 09:45:28
tags: angular
---

## angular 1.5.8 注入annotate方式解析

先看使用案例：

```js
// 创建myModule模块、注册服务
var myModule = angular.module('myModule', []);
myModule.service('myService', function() {
			this.my = 0;
});

// 获取injector
var injector = angular.injector(["myModule"]);

// 第二种annotation
function explicit(serviceA) {alert(serviceA.my);};
explicit.$inject = ['myService'];
injector.invoke(explicit);
```

### angular.module函数

核心angular.module方法是通过执行factory函数返回factory内部的函数,调用
angular.module('myModule', [])方法的时候已经是执行最内部的函数了,最终演变为
modules["myModule"] = factory(); 返回moduleInstance。

```js
function setupModuleLoader(window) {
    function ensure(obj, name, factory) {
    return obj[name] || (obj[name] = factory());
    }

   var angular = ensure(window, 'angular', Object);

   return ensure(angular, 'module', function() {

     var modules = {};

     return function module(name, requires, configFn) {

       return ensure(modules, name, function() {

       }
     }
  }
}
```

### module的service函数

module里包含了一系列方法:provider,factory,service,value,constant,decorator,

animation,filter,controller,directive,component。

value和constance 是invokeLater函数的返回值，其他是invokeLaterAndSetModuleName函数的

返回值。

service函数主要作用就是把函数放入providerCache
service内部 就是实例化传入的函数

最终调用invoke函数的时候 通过$injectorProvider 去调用instantiate函数把service中的function当作参数传入
```js
 service: invokeLaterAndSetModuleName('$provide', 'service');

   function service(name, constructor) {

     return factory(name, ['$injector', function($injector) {
       return $injector.instantiate(constructor);
     }]);
   }

   //强制返回一个方法而已
  function enforceReturnValue(name, factory) {
    return function enforcedReturnValue() {
      var result = instanceInjector.invoke(factory, this);
      if (isUndefined(result)) {
        throw $injectorMinErr('undef', "Provider '{0}' must return a value from $get factory method.", name);
      }
      return result;
    };
  }

    //最终放入provider
    //此处的factoryFn就是 上面的第二个参数 数组了
   function factory(name, factoryFn, enforce) {
    return provider(name, {
      //undefind !== false 为true
      $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
    });
  }

  //返回实例化的 provider
 //比如在定义ng模块的时候函数内部用到的链式调用
  function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    //没有定义$get函数会报错
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }

```

放入moduleInstance的invokeQueue中

### angular.injector函数

angular.injector函数返回一个instanceInjector

调用invoke函数  通过参数解析函数 annotate ，此函数可以直接把$inject属性当作参数传入


核心就是获取$provide  缓存
通过$provide 的providerCache 去获取 Provider

annotate函数在injectionArgs函数中被调用

返回注入的参数后 继续调用getService 去providerCache里去找 provider


### instanceInjector.invoke函数

invoke函数会调用传入的函数 ，如果函数包含$inject属性，那么会直接把$inject属性通过getService获取

该属性返回值，然后当作参数传入函数。


```js


//注解函数
function annotate(fn, strictDi, name) {
var $inject,
    argDecl,
    last;

if (typeof fn === 'function') {
  if (!($inject = fn.$inject)) { //如果fn有$inject属性，那么$injector就不为空
    $inject = [];
    if (fn.length) {
       console.log('fnlength:'+fn.length);//function length 等于1
    }

    console.log($inject.length);
    fn.$inject = $inject;
  }
} else if (isArray(fn)) { //比如['$scope',function($scope){}]
  last = fn.length - 1;
  //assertArgFn(fn[last], 'fn');//判断参数fn是否是个函数
  $inject = fn.slice(0, last);
} else {
  //assertArgFn(fn, 'fn', true);
}
return $inject;
}

//经过publishExternalAPI发布后 该函数可以通过angular.injector调用
function createInjector(modulesToLoad, strictDi) {
strictDi = (strictDi === true);
var INSTANTIATING = {},//instantiating
    providerSuffix = 'Provider',
    path = [],
    //loadedModules = new HashMap([], true),
    providerCache = {
      $provide: {
          provider: {},
          factory:  {},
          service: {},
          value: {},
          constant: {},
          decorator: function(){}
        }
    },

    //createInternalInjector 返回
    //return {
  //   invoke: invoke,
  //   instantiate: instantiate,
  //   get: getService,
  //   annotate: createInjector.$$annotate,
  //   has: function(name) {
  //     return providerCache.hasOwnProperty(name + providerSuffix) || cache.hasOwnProperty(name);
  //   }
  // };

    //createInternalInjector(cache,factory)
    providerInjector = (providerCache.$injector =
        createInternalInjector(providerCache, function(serviceName, caller) {
          if (angular.isString(caller)) {
            path.push(caller);
          }
          //throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
        })),
    instanceCache = {},
    protoInstanceInjector =
        createInternalInjector(instanceCache, function(serviceName, caller) {

          //去providerCache里去找
          var provider = providerInjector.get(serviceName + providerSuffix, caller);

          console.log(serviceName+ providerSuffix);
          return instanceInjector.invoke(
              provider.$get, provider, undefined, serviceName);
          //$get 就返回下面的valueFn
        }),
    instanceInjector = protoInstanceInjector;

//为了下面的protoInstanceInjector.get里的调用

//$get 返回一个 返回protoInstanceInjector的函数
//function valueFn(value) {return function valueRef() {return value;};}
providerCache['$injector' + providerSuffix] = { $get: valueFn(protoInstanceInjector) };

//var runBlocks = loadModules(modulesToLoad);
//loadModules很重要
//在injector.injector(['myModule'])时加载

//调用providerInjector.get

//调用的时候 instanceInjector还是protoInstanceInjector
//而且instanceCache里还没有任何属性

//去ProviderInjector的 providerCache里去找

//invoke $get就是返回  protorInstanceInjector

//为了之后调用getService('$injector',serviceName)时能找到
instanceInjector = protoInstanceInjector.get('$injector');//上面定义的providerCache.$jnjector
instanceInjector.strictDi = strictDi;
//forEach(runBlocks, function(fn) { if (fn) instanceInjector.invoke(fn); });
console.log(instanceInjector === protoInstanceInjector);
return instanceInjector;

  function createInternalInjector(cache, factory) {

    //没有cache里的serviceName的时候 ，再调用factory方法。
    function getService(serviceName, caller) {
      //http://blog.csdn.net/webdesman/article/details/20040815
    //对象是否有自己的属性 不在原型链上扩展的
      if (cache.hasOwnProperty(serviceName)) {
        if (cache[serviceName] === INSTANTIATING) {
          throw $injectorMinErr('cdep', 'Circular dependency found: {0}',
                    serviceName + ' <- ' + path.join(' <- '));
        }
        return cache[serviceName];
      } else {
        try {
          //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
          path.unshift(serviceName);
          cache[serviceName] = INSTANTIATING;
          //如果没有属性 那就调用factory函数 传入serviceName和caller
          //caller也是函数
          return cache[serviceName] = factory(serviceName, caller);
        } catch (err) {
          if (cache[serviceName] === INSTANTIATING) {
            delete cache[serviceName];
          }
          throw err;
        } finally {
            //shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值
          path.shift();
        }
      }
    }


    function injectionArgs(fn, locals, serviceName) {
      var args = [],
          //$$annotate就是外面的annotate函数
          //line 3966
          //函数参数列表
          $inject = createInjector.$$annotate(fn, strictDi, serviceName);

      for (var i = 0, length = $inject.length; i < length; i++) {
        var key = $inject[i];
        if (typeof key !== 'string') {
          throw $injectorMinErr('itkn',
                  'Incorrect injection token! Expected service name as string, got {0}', key);
        }
        args.push(locals && locals.hasOwnProperty(key) ? locals[key] :
                                                         getService(key, serviceName));
      }
      return args;
    }

    function isClass(func) {
      // IE 9-11 do not support classes and IE9 leaks with the code below.
      if (msie <= 11) {
        return false;
      }
      // Support: Edge 12-13 only
      // See: https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/6156135/
      return typeof func === 'function'
        && /^(?:class\b|constructor\()/.test(stringifyFn(func));
    }

    function invoke(fn, self, locals, serviceName) {
      if (typeof locals === 'string') {
        serviceName = locals;
        locals = null;
      }

      console.log(fn);
      //注入参数
      var args = injectionArgs(fn, locals, serviceName);

        console.log("argslength:"+args.length);
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
        args.unshift(null);//在头部插入Null
        return new (Function.prototype.bind.apply(fn, args))();
      }
    }


    return {
      invoke: invoke,
      get: getService,
      // instantiate: instantiate,
      annotate: createInjector.$$annotate,
      has: function(name) {
        return providerCache.hasOwnProperty(name + providerSuffix) || cache.hasOwnProperty(name);
      }
    };
  }
}

createInjector.$$annotate = annotate;
var msie = false;//window.document.documentMode;
function valueFn(value) {return function valueRef() {return value;};}
var isArray = Array.isArray;
```

createInjector 函数整体返回

```js
  return {
    invoke: invoke,
    instantiate: instantiate,
    get: getService,
    annotate: createInjector.$$annotate,
    has: function(name) {
      return providerCache.hasOwnProperty(name + providerSuffix) || cache.hasOwnProperty(name);
    }
 }
```

[理解angular中的module和injector，即依赖注入](http://www.mamicode.com/info-detail-247448.html)
