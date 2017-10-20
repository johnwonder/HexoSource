title: angular-compileProvider
date: 2017-07-27 14:03:38
tags: angular
---

## compileProvider是如何调用的


### compileProvider定义
```js
//为了在injectionArgs方法中 annotate中返回 然后给injectionArgs 使用
$CompileProvider.$inject = ['$provide', '$$sanitizeUriProvider'];
function $CompileProvider($provide, $$sanitizeUriProvider) {
  this.directive = function registerDirective(name, directiveFactory) {};

  this.$get = [];
}
```

我们回到publishExternalAPI函数看compileProvider的调用：

### publishExternalAPI
```js
 //instanceInjector 调用invoke函数
 //invoke函数内部调用
  $provide.provider('$compile', $CompileProvider)
```

### $provide.provider

实例化provider
```js
function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      //通过providerInjector实例化compileProvider
      //所以是从providerCache中获取参数
      provider_ = providerInjector.instantiate(provider_);
    }
    //没有定义$get函数会报错
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }
```

### instantiate

$provide.Provider的时候会调用此方法实例化provider，比如CompileProvider
```js
    function instantiate(Type, locals, serviceName) {
      // Check if Type is annotated and use just the given function at n-1 as parameter
      // e.g. someModule.factory('greeter', ['$window', function(renamed$window) {}]);
      var ctor = (isArray(Type) ? Type[Type.length - 1] : Type);
      //注入参数 从providerCache中获取参数
      var args = injectionArgs(Type, locals, serviceName);
      // Empty object at position 0 is ignored for invocation with `new`, but required.
      args.unshift(null);
      return new (Function.prototype.bind.apply(ctor, args))();
    }
```

### instanceInjector.invoke

```createInjector``函数会```loadModules```
```js
var injector = createInjector(modules, config.strictDi);
//invoke的时候会从instanceCache中去找，找不到会从providerCache中去找，然后调用provider.$get方法
//所以也就解释了下面为什么能直接调用compile函数
injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector',
       function bootstrapApply(scope, element, compile, injector) {
        scope.$apply(function() { //rootScope
          element.data('$injector', injector);
          compile(element)(scope);
        });
      }]
    );
```
