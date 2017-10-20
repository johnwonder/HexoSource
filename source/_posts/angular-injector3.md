title: angular_injector3
date: 2017-02-22 13:40:45
tags: angular
---

## angular1.5.8源码分析

###setupModuleLoader

模仿setupModuleLoader方法看看如何angular是如何构造angular.module的

```js
function setupModuleLoader(window) {

function ensure(obj, name, factory) {
  return obj[name] || (obj[name] = factory());
}

var angular = ensure(window, 'angular', Object);

return ensure(angular, 'module', function() {

    var modules = {};

    return function module(name, requires, configFn) {

          return ensure(modules, name, function() {

               var moduleInstance = {
                  hello : 'hello angular'
               };

               return moduleInstance;
          });
    };

})
}

var window = {};

angularModule = setupModuleLoader(window);

angularModule('ng', ['ngLocale'], ['$provide', function ngModule($provide) {}]);

//已经存在ng这个module
console.log(window.angular.module('ng').hello)
```

控制台输出hello angular
