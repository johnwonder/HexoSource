title: angular1.5.8 源码分析之依赖注入--模块创建
date: 2017-02-21 14:01:32
tags: angular
---

模块创建主要是用到了setupModuleLoader函数.

### setupModuleLoader函数

angular的模块是在依赖注入时必不可少的一个功能单元，所以要弄懂它的依赖注入，那必须讲它的模块设计。
总的来说通过angular.module的实现源码我们可以看到js闭包的强大功能。

```js
/*
  angular 模块加载器
*/
function setupModuleLoader(window) {

 //省略部分代码
function ensure(obj, name, factory) {
  //比如下方的ensure(angular,'module',function(){})
  return obj[name] || (obj[name] = factory());
}
var angular = ensure(window, 'angular', Object);
//此处定义了angular.module方法
return ensure(angular, 'module', function() {
  /** @type {Object.<string, angular.Module>} */
  var modules = {};

  /* 注释掉了 函数的描述*/
  //调用angular.module() 时执行的函数
  //requires 必须传递 ，比如[]数组
  return function module(name, requires, configFn) {
    var assertNotHasOwnProperty = function(name, context) {
      if (name === 'hasOwnProperty') {
        throw ngMinErr('badname', 'hasOwnProperty is not a valid {0} name', context);
      }
    };
    //判断moduleName 是不是 hasOwnProperty
    assertNotHasOwnProperty(name, 'module');
    if (requires && modules.hasOwnProperty(name)) {
      modules[name] = null;
    }
    return ensure(modules, name, function() {
      if (!requires) {
        throw $injectorMinErr('nomod', "Module '{0}' is not available! You either misspelled " +
           "the module name or forgot to load it. If registering a module ensure that you " +
           "specify the dependencies as the second argument.", name);
      }

      /** @type {!Array.<Array.<*>>} */
      //数组里也是数组
      //调用队列
      var invokeQueue = [];
      //数组里是函数
      //配置块
      var configBlocks = [];
      //数组里是函数
      //运行块
      var runBlocks = [];

      /*
          返回一个方法
          方法内部调用configBlocks['push'](['$injector','invoke',arguments])
          然后返回moduleInstance
          config就是一个返回moduleInstance的函数
      */
      //调用config方法会把参数放入configBlocks中
      var config = invokeLater('$injector', 'invoke', 'push', configBlocks);

      var moduleInstance = {
        // Private state
        //私有变量
        _invokeQueue: invokeQueue,
        _configBlocks: configBlocks,
        _runBlocks: runBlocks,
        //传入的依赖
        requires: requires,

        name: name,
        //返回调用$provide的provider方法
        provider: invokeLaterAndSetModuleName('$provide', 'provider'),
        //返回调用$provide的factory方法
        factory: invokeLaterAndSetModuleName('$provide', 'factory'),
        //返回调用$provide的service方法
        service: invokeLaterAndSetModuleName('$provide', 'service'),
        //返回调用$provide的value方法
        value: invokeLater('$provide', 'value'),
        //返回调用$provide的constant方法
        constant: invokeLater('$provide', 'constant', 'unshift'),
        //返回调用$provide的decorator方法
        decorator: invokeLaterAndSetModuleName('$provide', 'decorator'),
        //返回调用$animateProvider的register方法
        animation: invokeLaterAndSetModuleName('$animateProvider', 'register'),
        //返回调用$filterProvider的register方法
        filter: invokeLaterAndSetModuleName('$filterProvider', 'register'),
        //返回调用$controllProvider的register方法
        controller: invokeLaterAndSetModuleName('$controllerProvider', 'register'),
        //返回调用$compileProvider的directive方法
        directive: invokeLaterAndSetModuleName('$compileProvider', 'directive'),
        //返回调用$compileProvider的component方法
        component: invokeLaterAndSetModuleName('$compileProvider', 'component'),

        config: config,

        //调用run方法会把参数block加入runBlocks队列。
        run: function(block) {
          runBlocks.push(block);
          return this;
        }
      };

      //configFn有可能就是['$provide',function($provide){}]
      //config 就是调用configBlocks[insertMethod || 'push']([provider, method, arguments]);
      //这里的arguments就是configFn
      if (configFn) {
        //如果有configFn方法，那么就放入configBlocks队列
        config(configFn);
      }

      return moduleInstance;
    });
  };
});

}
```

### invokeLater

```js

//延迟调用
//module.value 就是先调用的invokeLater返回的一个函数
//然后调用module.value()的时候就是把provider,method  arguments放入 queue
function invokeLater(provider, method, insertMethod, queue) {
  if (!queue) queue = invokeQueue;
  return function() {

    //就是调的数组的[].push方法
    queue[insertMethod || 'push']([provider, method, arguments]);
    return moduleInstance;
  };
}
```

### invokeLaterAndSetModuleName

从名字看来 很像懒加载模式, 返回函数用来稍后调用
把provider放入invokeQueue队列中

调用的时候如果factoryFunction是函数 那么就设置它的moduleName为当前模块名称
```js
      //比如controller是'$controllerProvider', 'register'
      function invokeLaterAndSetModuleName(provider, method) {
        //调用controller 后
        return function(recipeName, factoryFunction) {
          if (factoryFunction && isFunction(factoryFunction)) factoryFunction.$$moduleName = name;
          invokeQueue.push([provider, method, arguments]);
          return moduleInstance;
        };
      }
```

当调用`var myModule = angular.module('myModule', []);`的时候 就返回moduleInstance了。
