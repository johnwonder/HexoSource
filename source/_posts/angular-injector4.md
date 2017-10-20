title: angular_injector4
date: 2017-03-15 09:45:28
tags: angular
---

## angular 1.5.8 注入解析

```js

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

////////////////////////////////////
// $provider
////////////////////////////////////

////////////////////////////////////
// internal Injector
////////////////////////////////////

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

[理解angular中的module和injector，即依赖注入](http://www.mamicode.com/info-detail-247448.html)
