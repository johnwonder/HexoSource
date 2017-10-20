title: angular_injector1
date: 2017-02-20 21:13:55
tags: angular
---

## angular中的注入分析

### 先来看一段简单的代码：

```js
/*
  创建内部注入器
*/
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

  //此处方法先省略
}
```

### 如何使用这个方法呢

```js
//providerCache从此有了$injector属性
providerInjector = (providerCache.$injector =
        createInternalInjector(providerCache, function(serviceName, caller) {
          if (angular.isString(caller)) {
            path.push(caller);
          }
          //$injectorMinErr 已经是个函数 返回的是个Error对象
          //unpr 是个code
          //Unknown。。 是具体的模板消息 {0} 是格式化的参数
          throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
        })),
```

那么factory就是

```js
  function(serviceName, caller) {
    if (angular.isString(caller)) {
      path.push(caller);//向数组尾部加入caller字符串
    }
    throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
  }
```  
```js
  //minErr返回一个方法
  //这个方法的返回值是个ErrorConstructor
  //  ErrorConstructor = ErrorConstructor || Error;/
  //$injector是个模块
  var $injectorMinErr = minErr('$injector')
```

类似的还有：
```js
   var $qMinErr = minErr('$q', TypeError);//这里就传入了一个TypeError对象
```
