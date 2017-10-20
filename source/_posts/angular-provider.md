title: angular_provider
date: 2017-03-27 15:00:22
tags: angular
---

## angular provider 注入

```js
angularModule('ng', ['ngLocale'], ['$provide',
  function ngModule($provide) {
    // $$sanitizeUriProvider needs to be before $compileProvider as it is used by it.
    $provide.provider({
      $anchorScroll: $AnchorScrollProvider
    });
  }
]);
```

注意 `['$provide',function ngModule($provide){}]` 这段代码,
在定义angular.module方法时 就加入了configFn参数:  

```js
* @param {Function=} configFn Optional configuration function for the module. Same as
 *        {@link angular.Module#config Module#config()}.
 * @returns {angular.Module} new module with the {@link angular.Module} api.
 */
return function module(name, requires, configFn) {

        //configFn有可能就是['$provide',function($provide){}]
        //config 就是调用configBlocks[insertMethod || 'push']([provider, method, arguments]);
        //这里的arguments就是configFn
        if (configFn) {
          config(configFn);
        }
}
```  

config定义如下：  

```js
/*返回一个方法
  方法内部调用configBlocks['push'](['$injector','invoke',arguments])
  然后返回moduleInstance

  config就是一个返回moduleInstance的函数
*/
var config = invokeLater('$injector', 'invoke', 'push', configBlocks);

function invokeLater(provider, method, insertMethod, queue) {
          if (!queue) queue = invokeQueue;
          return function() {
            queue[insertMethod || 'push']([provider, method, arguments]);
            return moduleInstance;
          };
}
```

configBlocks push了['$injector','invoke',['$provide',function($provide){}]]  

然后在bootstrap方法中createInjector  

createInjector中再调用loadModules  

loadModules中`runInvokeQueue(moduleFn._configBlocks);` 调用providerCache.$injector 返回  

```js
{
    invoke: invoke,
    instantiate: instantiate,
    get: getService,
    annotate: createInjector.$$annotate,
    has: function(name) {
      return providerCache.hasOwnProperty(name + providerSuffix) || cache.hasOwnProperty(name);
    }
  }
```  

继续调用invoke函数 -> injectionArgs -> getService(key) 返回$provide 对象

[AngularJs自定义服务](http://blog.csdn.net/wz172637815/article/details/50595718)
