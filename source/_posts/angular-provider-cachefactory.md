title: angular源码剖析之Provider系列--CacheFactoryProvider
date: 2018-03-01 15:04:17
tags: angular
---

## CacheFactoryProvider 简介

  提供存储和访问缓存对象的服务

## CacheFactoryProvider 用法
```html
<!doctype html>
  <html lang="en" ng-app="myapp">
   <head>
    <meta charset="UTF-8">
    <meta name="Generator" content="EditPlus®">
    <meta name="Author" content="">
    <meta name="Keywords" content="">
    <meta name="Description" content="">
    <title>Document</title>
    <script src="js/angular.js"></script>
   </head>
   <body>
    <div ng-controller="MyController">
    </div>
     <script >
     var app=angular.module('myapp',[]);
     app.controller('MyController',function($scope,$cacheFactory){

     var myCache = $cacheFactory("myCache",{capacity: 6});
     //var myCache1 = $cacheFactory("myCache",{capacity: 6}); //会报错
     myCache.put("name","sssss");
     myCache.put("name1","sssss");
     myCache.put("name","sssss");

     });

     app.controller('getCacheController',['$scope','$cacheFactory',function($scope,$cacheFactory){  
        var cache = $cacheFactory.get('myCache');  
        var name = cache.get('name');  
        console.log(name);  //打印sssss
    }]);  
     </script>
   </body>
  </html>
```


看了上面这个一个简单的例子，读者可能会产生如下疑惑：  
1. 不是名为CacheFactoryProvider吗，怎么在代码里只看到cacheFactory呢？
2. cacheFactory是怎么存储对象的？
下面我们来依次解答这两个问题

## $cacheFactory的注入
我们首先来看第一个问题，这个问题要牵涉到angular里面的依赖注入机制，我们前面的分析也讲过，
angular会在启动之前通过调用`publishExternalAPI` 函数先发布一些扩展API,同时定义ng  
模块，在定义ng模块的时候就传入了注入provider的方法  

```
angularModule('ng', ['ngLocale'], ['$provide',
     function ngModule($provide) {
          ///部分代码省略
         $provide.provider({
           $cacheFactory: $CacheFactoryProvider,
         });
    }])
```
$cacheFactory出现了，它是通过javascript的键值对象作为键传给provider方法。那么它是如何存储
对象的呢？首先我们看它的定义：

## CacheFactoryProvider的定义

内部定义了依赖注入核心的`$get`方法,`$get`方法返回cacheFactory方法(也就是上面实例代码里的
  `$cacheFactory`参数)。

```js
  function $CacheFactoryProvider() {
    //定义$get方法供依赖调用
     this.$get = function() {
       var caches = {};

       function cacheFactory(cacheId, options) {
         //部分代码省略
           if (cacheId in caches) {//如果caches中已经存在cacheId
             //实例代码里抛出的错误就在此处、
             //统一调用minErr函数
             throw minErr('$cacheFactory')('iid', "CacheId '{0}' is already taken!", cacheId);
           }

           var size = 0,
           //把options 和{id:cacheId} 放入{} 中 不是深拷贝
           stats = extend({}, options, {id: cacheId}),
           data = createMap(),
           capacity = (options && options.capacity) || Number.MAX_VALUE,
           lruHash = createMap(),
           freshEnd = null,
           staleEnd = null;
           //返回caches中的一个对象
           return caches[cacheId] = {

           }
           //
           function refresh(entry) {

           }
           //
           function link(nextEntry, prevEntry) {

           }
         }

         //所有的缓存
        cacheFactory.info = function() {
          var info = {};
          forEach(caches, function(cache, cacheId) {
            info[cacheId] = cache.info();
          });
          return info;
        };

        cacheFactory.get = function(cacheId) {
            return caches[cacheId];
        };


        return cacheFactory;
     }
  }
```
## CacheFactoryProvider的存储
