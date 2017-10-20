title: angular_injector2
date: 2017-02-21 14:01:32
tags: angular
---

## angular1.5.8 源码分析之依赖注入

### angular.module的实现

```js
/*
  angular 模块加载器
*/
function setupModuleLoader(window) {

var $injectorMinErr = minErr('$injector');
var ngMinErr = minErr('ng');

function ensure(obj, name, factory) {
  //比如下方的ensure(angular,'module',function(){})
  return obj[name] || (obj[name] = factory());
}

var angular = ensure(window, 'angular', Object);

// We need to expose `angular.$$minErr` to modules such as `ngResource` that reference it during bootstrap
angular.$$minErr = angular.$$minErr || minErr;

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
      var invokeQueue = [];

      /** @type {!Array.<Function>} */
      var configBlocks = [];

      /** @type {!Array.<Function>} */
      var runBlocks = [];

      /*
          返回一个方法
          方法内部调用configBlocks['push'](['$injector','invoke',arguments])
          然后返回moduleInstance
          config就是一个返回moduleInstance的函数
      */
      var config = invokeLater('$injector', 'invoke', 'push', configBlocks);

      var moduleInstance = {
        // Private state
        _invokeQueue: invokeQueue,
        _configBlocks: configBlocks,
        _runBlocks: runBlocks,

        requires: requires,

        name: name,

        provider: invokeLaterAndSetModuleName('$provide', 'provider'),

        factory: invokeLaterAndSetModuleName('$provide', 'factory'),

        service: invokeLaterAndSetModuleName('$provide', 'service'),

        value: invokeLater('$provide', 'value'),

        constant: invokeLater('$provide', 'constant', 'unshift'),

        decorator: invokeLaterAndSetModuleName('$provide', 'decorator'),

        animation: invokeLaterAndSetModuleName('$animateProvider', 'register'),

        filter: invokeLaterAndSetModuleName('$filterProvider', 'register'),

        controller: invokeLaterAndSetModuleName('$controllerProvider', 'register'),

        directive: invokeLaterAndSetModuleName('$compileProvider', 'directive'),

        component: invokeLaterAndSetModuleName('$compileProvider', 'component'),

        config: config,

        run: function(block) {
          runBlocks.push(block);
          return this;
        }
      };

      //configFn有可能就是['$provide',function($provide){}]
      //config 就是调用configBlocks[insertMethod || 'push']([provider, method, arguments]);
      //这里的arguments就是configFn
      if (configFn) {
        config(configFn);
      }

      return moduleInstance;

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

      //比如controller是'$controllerProvider', 'register'
      function invokeLaterAndSetModuleName(provider, method) {
        //调用controller 后
        return function(recipeName, factoryFunction) {
          if (factoryFunction && isFunction(factoryFunction)) factoryFunction.$$moduleName = name;
          invokeQueue.push([provider, method, arguments]);
          return moduleInstance;
        };
      }
    });
  };
});

}
```

当调用`var myModule = angular.module('myModule', []);`的时候 就返回moduleInstance了。
