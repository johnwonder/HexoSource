title: angularjs源码分析--依赖注入器的产生
date: 2017-02-20 21:13:55
tags: angular
---

## angularjs依赖注入介绍

依赖注入在当今很多框架中都是必不可少要提供的功能，比如Java世界中强大的Spring框架。

我们在使用angularjs时会大量用到它内部依赖注入提供的能力，比如在controller里我们可以直接
使用controller函数的参数对象而无需关注这些对象怎么传入，这就是依赖注入的魔力。

下面就开始分篇介绍angularjs是怎么一步步注入依赖的。

### createInjector函数

createInjector可以说是angularjs实现依赖注入的核心函数，最终这个函数返回的是一个instanceInjector 实例注入器，就可以利用这个实例注入器来注入一些需要的资源。
```js
  function createInjector(modulesToLoad, strictDi) {

    //省略部分代码
    providerInjector = (providerCache.$injector =
          createInternalInjector(providerCache, function(serviceName, caller) {
            if (angular.isString(caller)) {
              path.push(caller);
            }
            throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
    })),
    //protoInstanceInjector就是通过createInternalInjector函数来获取
    var instanceInjector = protoInstanceInjector.get('$injector');//上面定义的providerCache.$jnjector
    instanceInjector.strictDi = strictDi;
    //if(fn) 因为有的方法invoke没有返回值
    //invoke的时候 通过injectionArgs方法把参数对象注入
    //runBlocks是数组
    forEach(runBlocks, function(fn) { if (fn) instanceInjector.invoke(fn); });

    return instanceInjector;
    }
```

### createInternalInjector函数

首先讲到angularjs依赖注入很关键的一个函数createInternalInjector，顾名思义，这个方法的目的就是创建内部注入器。

函数传入了一个factory工厂方法参数，这个factory工厂设计模式我们在angular内部可以看到很多地方被运用，我们要明白一个简单的道理 ：工厂设计模式的好处就是管理了内部逻辑，使用者只管调用，工厂自然地会帮你生产出你想要的东西。

我们还可以观察到该方法内部会创建一个内部getService方法，然后以其他形式返回供外部使用，这种设计在javascript中很常见，就是所谓的闭包 ，在angularjs框架内部也是随处可见。

getService方法内部会对cache参数做读写，可以维护这个外部函数传入的cache参数。
createInternalInjector函数最终会返回一个注入器对象供外部使用，
```js
/*
  创建内部注入器
*/
function createInternalInjector(cache, factory) {

  //cache和factory参数在getService内部使用
  //angularjs内部闭包使用很频繁
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
  //省略部分代码

  return {
     //省略部分代码
     get: getService,
   };
}
```

createInternalInjector方法的内部使用，最终返回给providerInjector(提供者注入器)，protoInstanceInjector（原型对象注入器）。

```js
//providerCache从此有了$injector属性
providerInjector = (providerCache.$injector =createInternalInjector(providerCache, function(serviceName, caller) {
          if (angular.isString(caller)) {
            path.push(caller);
          }
          //$injectorMinErr 已经是个函数 返回的是个Error对象
          //unpr 是个code
          //Unknown。。 是具体的模板消息 {0} 是格式化的参数
          throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
        })),
instanceCache = {},
protoInstanceInjector =createInternalInjector(instanceCache, function(serviceName, caller) {

     var provider = providerInjector.get(serviceName + providerSuffix, caller);

    //invoke函数 fn,self,locals,serviceName
    //调用provider的$get函数放入instanceCache中。
    return instanceInjector.invoke(
                        provider.$get, provider, undefined, serviceName);
                    //$get 就返回下面的valueFn
  })
```

我们看到在创建原型对象注入器的时候工厂方法内部会有对instanceInjector（对象注入器的使用),
这个instanceInjector是通过 protoInstanceInjector 来获取的。

protoInstanceInjector注入器的工厂方法内有很重要的两行代码，首先通过提供者注入器获取serviceName加上providerSuffix对应的provider，然后通过instanceInjector对象实例注入器调用invoke方法，传入提供者的$get属性等，返回instance放入instanceCache中。

我们之后会分析下这边的invoke方法和provider的$get方法。
