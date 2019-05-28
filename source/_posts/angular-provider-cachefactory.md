title: angular源码剖析之Provider系列--CacheFactoryProvider
date: 2018-03-01 15:04:17
tags: angular
---

## CacheFactoryProvider 简介

源码里是这么描述的：
Factory that constructs {@link $cacheFactory.Cache Cache} objects and gives access to
them.
意思就是通过cacheFactory可以构造一个Cache对象来给予访问和执行权限。

这个Cache对象[官方文档](https://code.angularjs.org/1.5.8/docs/api/ng/type/$cacheFactory.Cache)是这么说的：
A cache object used to store and retrieve data, primarily used by $http and the script directive to cache templates and other data.
用我自己的话来说就是 提供存储和访问缓存对象的服务,angular内部主要被$http,script指令用于
缓存template和其他数据。我们自己可以在Controller内部使用。


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
         myCache.put("name","john");
         myCache.put("name1","wonder");
         myCache.put("name","john");
      });

       app.controller('getCacheController',['$scope','$cacheFactory',
          function($scope,$cacheFactory){  
                var cache = $cacheFactory.get('myCache');  
                var name = cache.get('name');  
                console.log(name);  //打印john
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
     //通过参数注入$provide
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
    //controller中获取cacheFactory时会调用此方法
    //这个$get方法也是获取provider的关键方法
     this.$get = function() {
       var caches = {};//闭包的一个运用

       function cacheFactory(cacheId, options) {
         //部分代码省略
         //可以用if来判断
         if (cacheId in caches) {//如果caches中已经存在cacheId
             //实例代码里抛出的错误就在此处、
             //统一调用minErr函数
            throw minErr('$cacheFactory')('iid', "CacheId '{0}' is already taken!"
                                          , cacheId);
          }

           var size = 0,
           //把options 和{id:cacheId} 放入{} 中 不是深拷贝
           stats = extend({}, options, {id: cacheId}),
           data = createMap(),//通过Object.create(null) 创建个空对象
           capacity = (options && options.capacity) || Number.MAX_VALUE,
           lruHash = createMap(),
           freshEnd = null,
           staleEnd = null;
           //返回caches中的一个对象
           return caches[cacheId] = {
                //省略部分代码
                //存储里讲解
                put:function(key,value){
                },
                get: function(key) {
                },
                remove: function(key) {
                },
                removeAll: function() {
                },
                destroy: function() {
                },
                info: function() {
                }
           }
           //刷新节点次序
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

存储分为这几个核心方法:`put`,`refresh`,`remove`,`link`

### put函数

value会放入data对象中，key会放入lruHash链表
```js
      put: function(key, value) {
         if (isUndefined(value)) return;
           //如果设定的capcity小于maxvalue
         if (capacity < Number.MAX_VALUE) {
           //lruHash 存了当前的key 还有可能是 p 和n  (previous和next)
           var lruEntry = lruHash[key] || (lruHash[key] = {key: key});
           //刷新各节点的次序
           refresh(lruEntry);//把当前entry放入链表末尾
         }
         //如果key 在data里不存在 那么增加size
         if (!(key in data)) size++;
         data[key] = value;
         //当大于capacity时 会清除最早加入的那个
         if (size > capacity) {
           this.remove(staleEnd.key);//移除淘汰节点stableEnd
         }
         return value;
     }
```

### get函数

Retrieves named data stored in the {@link $cacheFactory.Cache Cache} object
获取存储在cache对象中的指定数据
```js
        get: function(key) {
          if (capacity < Number.MAX_VALUE) {
            var lruEntry = lruHash[key];
            if (!lruEntry) return;
            // 获取first的时候 因为staleEnd为first 所以会让staleEnd指向 second
            // 内部会执行link 使得 second.p = null
            // first.p =  third  third.n = first
            //stableEnd为 second freshEnd为first
            refresh(lruEntry);
          }
          return data[key];
        }
```

### remove函数

Removes an entry from the {@link $cacheFactory.Cache Cache} object.
从cache对象删除一个entry

```js
remove: function(key) {
          //如果capacity小于maxvalue
          if (capacity < Number.MAX_VALUE) {
            //先取出当前key的entry
            var lruEntry = lruHash[key];
            if (!lruEntry) return;
            //第一次超过时 freshEnd 为third  lryEntry为first
            if (lruEntry == freshEnd) freshEnd = lruEntry.p;
             //第一次超过时 staleEnd 为first  lryEntry为first
             //所以 会让 stalEnd 指向second 以便于下次移除时
            if (lruEntry == staleEnd) staleEnd = lruEntry.n;
            //把淘汰节点的一个节点选中
            //第一次超过时 lryEntry.n为 second  lryEntry.p 为null
            //执行结果为 second.p = null
            link(lruEntry.n,lruEntry.p);
            //把当前key从lruHash中删除
            delete lruHash[key];
          }
          if (!(key in data)) return;
          delete data[key];
          size--;
        }
```

### refresh函数

makes the `entry` the freshEnd of the LRU linked list。
把entry 放入链表的末尾
```js
    function refresh(entry) {
      if (entry != freshEnd) {
        if (!staleEnd) { //staleEnd为空那么就让他指向当前entry
          staleEnd = entry;
        } else if (staleEnd == entry) {
          //如果淘汰节点等于当前节点
          staleEnd = entry.n; //用于把 当前的下一个节点 用作淘汰节点
        }
        //放入第一个元素时 entry.n,entry.p都为undefined
        link(entry.n, entry.p); //当前的上一个节点 和当前的下一个节点
        link(entry, freshEnd); // 当前的节点 和 最新的末尾节点
        freshEnd = entry;  
        freshEnd.n = null;
        //第一次执行完 结果为: freshEnd = first  staleEnd为first  
        //first.p=null first.n=null
        //第二次执行完 结果为：freshEnd = second staleEnd为first
        // first.p=null first.n= second
        // scecond.p = first scecond.n = null
        //第三次执行完 freshEnd = third staleEnd为first first.p=null
        //first.n= second
        // second.p = first second.n = null
        // third.p = second third.n = null
      }
    }
```

### link函数

bidirectionally(双向链表) links two entries of the LRU linked list
双向链接链表里的两个元素。

```js
function link(nextEntry, prevEntry) {
      //undefined 不等于undefined
      if (nextEntry != prevEntry) {
        //
        if (nextEntry) nextEntry.p = prevEntry;
        //p stands for previous, 'prev' didn't minify
        if (prevEntry) prevEntry.n = nextEntry;
         //n stands for next, 'next' didn't minify
      }
    }
```

###### 参考资料：
1. [【算法】—— LRU算法](https://www.cnblogs.com/bopo/p/9255654.html)
2. [缓存淘汰算法--LRU算法](https://flychao88.iteye.com/blog/1977653)
3. [漫画：什么是LRU算法？](https://mp.weixin.qq.com/s/L-rTC-qlnvKxbwUI1FmeHw)
4. [头条面试题：如何实现 LRU 原理？](https://mp.weixin.qq.com/s/HE22NfxaCP6p5t39izvWdg)
