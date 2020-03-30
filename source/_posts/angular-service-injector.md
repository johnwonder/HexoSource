title: angularjs 1.5.8源码解析之service到底干了什么
date: 2017-07-01 22:13:53
tags: angular
---

## $provide.service函数

上篇文章[创建service](https://mp.weixin.qq.com/s/B3-VS8jnOSOKtDz91rtP0g)最后我们提到了
$provide.service,也就是说模块调用service方法 其实内部到loadModules的时候通过runInvokeQueue
会调用内置的$provide.service函数。

比如我写了个方法如下：
```js
var herModule = angular.module('herModule', []);
herModule.service('herService', function() {
    this.her = 1;
});
```
我们一步步来分析它是怎么调用的，首先看第一步

### provideCache

我们在之前的文章[依赖注入器的产生](https://mp.weixin.qq.com/s/1IUhjOQoVfEdds0nt-8Hlw)中看到createInjector的时候会把provideCache传入providerInjector，也就是说我们在调用```providerInjector.get('$provide')```的时候会从provideCache中去拿$provide,看下providerCache的定义。

```js
    function supportObject(delegate) {
       return function(key, value) {       
         //当key是 对象的时候 就把delegate函数放入闭包里
         if (isObject(key)) {
           //reverseParams 为了适应foreach里调用函数的参数顺序
           forEach(key, reverseParams(delegate));// function(value, key) {iteratorFn(key, value);};
         } else {
            //调用参数delegate方法
           return delegate(key, value);
         }
       };
    }
    var providerCache = {
       //supportObject 都返回一个新函数,内部调用service
       $provide: {
           service: supportObject(service),
           //省略部分代码
         }
     }
```

$provide对象不止有service一个属性函数，supportObject 都返回一个新函数,内部调用service。

### 内部service函数

service函数内部其实就是调用factory函数
```js
  function service(name, constructor) {
     //factoryFn就是个对象数组
    return factory(name, ['$injector', function($injector) {
      //实例化构造函数，返回实例
      return $injector.instantiate(constructor);
    }]);
  }
```

### 内部factory函数

factory函数内部又调用provider函数
```js
//factoryFn有可能就是对象数组
//enforce代表是否要有返回值
function factory(name, factoryFn, enforce) {
  return provider(name, {
    $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
  });
}

//返回一个有返回值的方法而已
  function enforceReturnValue(name, factory) {
    return function enforcedReturnValue() {
      var result = instanceInjector.invoke(factory, this);

      return result;
    };
  }
```

### 内部provider函数

```js
function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    return providerCache[name + providerSuffix] = provider_;
  }
```

其中最终调用的就是provider方法把一开始的serviceName加上providerSuffix当作对象key，然后```{$get:['$injector', function($injector) {return $injector.instantiate(constructor);}]}```放入了providerCache。
