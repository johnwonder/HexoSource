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
$cacheFactory出现了，它是通过javascript的键值对象作为键传给provider方法。
