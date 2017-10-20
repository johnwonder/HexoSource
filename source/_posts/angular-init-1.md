title: angular_源码分析之启动过程
date: 2017-07-26 12:41:02
tags: angular
---

## 启动过程
在angular启动过程中会经历以下步骤：

### publishExternalAPI

函数中会加载扩展函数，定义```angular.module```方法，载入```ng```模块。

### angularInit

angularInit函数中会调用```bootstrap```,bootstrap中会调用```createInjector```方法完成依赖注入，然后调用```invoke```启动编译服务

```js
////通过createInjector方法会调用注入时提供的ng模块中的configFn
   //configFn中提供了$rootScope, $rootElement, $compile, $injector
   //通过$provide.provider方法提供
   //modules中首先就是ng模块
var injector = createInjector(modules, config.strictDi);
  //$rootScope,$rootElement,$compile,$injector都是在publishExternalAPI中
   //注入ng模块时由$provide.provider提供的
   //$rootElement在bootstrap中
  //  modules.unshift(['$provide', function($provide) {
  //     $provide.value('$rootElement', element);
  //   }]);
   injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector',
      function bootstrapApply(scope, element, compile, injector) {
       scope.$apply(function() { //rootScope
         element.data('$injector', injector);
         compile(element)(scope);
       });
     }]
   );
```

### createInjector

```js
  var runBlocks = loadModules(modulesToLoad);
```
loadModules是createInjector内部方法,方法中会调用```moduleFn = angularModule(module);```
获取模块。  

因为定义ng模块时是push到configBlocks中的，所以会直接调用runInvokeQueue

### runInvokeQueue
```js
//queue就是在定义ng 模块时因为有configFn参数,然后调用config 函数（实际是通过invokeLater返回的函数)push到configBlocks中。
 var config = invokeLater('$injector', 'invoke', 'push', configBlocks);//configBlocks此时为空

 function invokeLater(provider, method, insertMethod, queue) {
          if (!queue) queue = invokeQueue;
          return function() {
            queue[insertMethod || 'push']([provider, method, arguments]);
            return moduleInstance;
          };
}

//此时queue就是['$injector','invoke',[['$provide',function($provide){}]]]
function runInvokeQueue(queue) {
       var i, ii;
       for (i = 0, ii = queue.length; i < ii; i++) {
         var invokeArgs = queue[i],

            //loadModules为内部方法所以能直接调用providerInjector
            //get方法就是返回injector对象的getService方法
             provider = providerInjector.get(invokeArgs[0]);
          //实际就是调用$injector的invoke方法
         provider[invokeArgs[1]].apply(provider, invokeArgs[2]);

       }
     }
```

### invoke函数

```js
//调用runInvokeQueue时,fn就是['$provide',function($provide){}]
function invoke(fn, self, locals, serviceName) {
      if (typeof locals === 'string') {
        serviceName = locals;
        locals = null;
      }

      //注入参数
      //如果fn是数组那么会判断数组最后一个元素是否为函数
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

### providerCache

$provide提供了provider方法,比如```$provide.provider('$compile', $CompileProvider)```

```js
//providerInjector调用invoke函数时,getService
//是从providerCache中获取的
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
```

### provider

```js
  //name $compile
  //provider_  CompileProvider
  //返回CompileProvider实例 并且放入providerCache中。
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
```
provider方法里其实调用了```instantiate```函数
```js
//Type就可能是函数类型
function instantiate(Type, locals, serviceName) {
     // Check if Type is annotated and use just the given function at n-1 as parameter
     // e.g. someModule.factory('greeter', ['$window', function(renamed$window) {}]);
     var ctor = (isArray(Type) ? Type[Type.length - 1] : Type);
     var args = injectionArgs(Type, locals, serviceName);
     // Empty object at position 0 is ignored for invocation with `new`, but required.
     args.unshift(null);
     //这样构造。。返回函数本身的对象，然后就可以链式调用了。
     return new (Function.prototype.bind.apply(ctor, args))();
   }
```
