title: angular_源码分析之rootElement
date: 2017-07-26 16:15:30
tags: angular
---

## rootElementProvider
rootElementProvider是从哪来的，我们先来看段源码中的话：

```js
/*
 * @ngdoc service
 * @name $rootElement
 *
 * @description
 * The root element of Angular application. This is either the element where {@link
 * ng.directive:ngApp ngApp} was declared or the element passed into
 * {@link angular.bootstrap}. The element represents the root element of application. It is also the
 *根元素
 * location where the application's {@link auto.$injector $injector} service gets
 * published, and can be retrieved using `$rootElement.injector()`.
 */
// the implementation is in angular.bootstrap
//实现在angular.bootstrap函数中
```
我们回到bootstrap函数中一探究竟：

### bootstrap
```js
function bootstrap(element, modules, config) {
      modules = modules || [];
      //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
      modules.unshift(['$provide', function($provide) {
        //调用$provide方法
        $provide.value('$rootElement', element);
      }]);
  }
```

调用了$provide的value方法

### $provide.value

```js
//用个valueFn包装下,再调用factory方法
//不需要返回值，所以第三个参数传false
  function value(name, val) { return factory(name, valueFn(val), false); }
```
### $provide.factory

```js
  function factory(name, factoryFn, enforce) {
   return provider(name, {
     $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
   });
  }

  //返回一个方法而已,方法里要调用factory方法
  //相当于一定要有返回值
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

### $provide.provider
最终调用provider方法,provider后缀为```  providerSuffix = 'Provider',```
```js

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
