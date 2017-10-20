title: angular_injector5
date: 2017-06-17 16:57:01
tags: angular
---


## angular injector 剖析流程

angular中的注入器是承载angular整个运行的基石，如果想弄清楚angular的来龙去脉，那injector这块必须得熟悉。

### createInjector

createInjector方法是入口函数，方法参数是modulesToLoad和strictDi,

我们主要关注modulesToLoad参数，字面含义就是待加载的模块。

#### providerCache
函数一开始就定义了providerCache对象，顾名思义就是提供provider的缓存。

providerCache内部又定义了$provide对象属性，暂且不管。

#### providerInjector

定义provider的注入器，同时也定义了providerCache.$injector.
最终是返回一个用于提供protoInstanceInjector查找instance的方法对象

#### publishExternalAPI

通过setupModuleLoader 方法，顾名思义为设置模块加载器， 定义了 angularModule 对象为angular.module方法

里面很重要的一个方法就是
```js
function ensure(obj, name, factory) {
    return obj[name] || (obj[name] = factory());
  }
```

随之定义了angular.module('ng',['ngLocale'], ['$provide',function($provide){}])

把ng模块放入了modules对象集合

先是定义```var config = invokeLater('$injector', 'invoke', 'push', configBlocks);```

##### invokeLater

```js
function invokeLater(provider, method, insertMethod, queue) {
          if (!queue) queue = invokeQueue;
          return function() {
            queue[insertMethod || 'push']([provider, method, arguments]);
            return moduleInstance;
          };
        }
```

通过以下方法把```['$provide',function($provide){}]```放入了configBlocks

```js
if (configFn) {
          config(configFn);
        }
```

configBlocks一开始定义为空数组，然后push 了一个数组对象```['$injector','invoke',['$provide',function($provide){}]]```

  下篇文章继续分析push了以后是从哪调用的。
