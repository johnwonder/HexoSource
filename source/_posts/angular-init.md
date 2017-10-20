title: angular_源码分析之初始化
date: 2017-02-24 15:35:27
tags: angular
---

## angular初始化过程

### 页面加载完成调用angularInit

跟jquery中的ready函数很像吧
```js
  jqLite(window.document).ready(function() {
    angularInit(window.document, bootstrap);
  });
```

### jqLite.ready函数

```js
var JQLitePrototype = JQLite.prototype = {
    ready: function(fn) {
      var fired = false;
      //trigger方法中再调用参数fn
      function trigger() {
        if (fired) return;
        fired = true;
        fn();//调用fn参数 函数
      }

      // check if document is already loaded
      if (window.document.readyState === 'complete') {
        window.setTimeout(trigger);
      } else {
        this.on('DOMContentLoaded', trigger); // works for modern browsers and IE9
        // we can not use jqLite since we are not done loading and jQuery could be loaded later.
        // jshint -W064
        JQLite(window).on('load', trigger); // fallback to window.onload for others
        // jshint +W064
      }
    }
  }
```

### angularInit

```js
  //element元素 ，从上文看就是window.document
  //bootstrap函数
  function angularInit(element, bootstrap) {
      var appElement,
          module,
          config = {};

      //element比其他元素优先级高
      // The element `element` has priority over any other element.
      forEach(ngAttrPrefixes, function(prefix) {
        var name = prefix + 'app';

        if (!appElement && element.hasAttribute && element.hasAttribute(name)) {
          appElement = element;
          module = element.getAttribute(name);
        }
      });
      forEach(ngAttrPrefixes, function(prefix) {
        var name = prefix + 'app';
        var candidate;
        //从element元素中用querySelector方法查找带ng-app的元素
        if (!appElement && (candidate = element.querySelector('[' + name.replace(':', '\\:') + ']'))) {
          appElement = candidate;
          module = candidate.getAttribute(name);
        }
      });
      //ng-app标记的元素，
      //模块比如ng-app="oaApp"，那么module就等于oaApp
      if (appElement) {
        config.strictDi = getNgAttribute(appElement, "strict-di") !== null;
        //启动
        bootstrap(appElement, module ? [module] : [], config);
      }
  }
```

### bootstrap

最终调用的还是bootstrap函数
```js
    function bootstrap(element, modules, config) {
    if (!isObject(config)) config = {};
    var defaultConfig = {
      strictDi: false
    };
    //扩展默认配置
    config = extend(defaultConfig, config);
    var doBootstrap = function() {
      element = jqLite(element);//用jqLite包装下

      //判断injector方法调用的结果
      //一开始调用element.injector()时是返回false的，当调用到
      //element.data('$injector', injector);就为true了
      if (element.injector()) {
        var tag = (element[0] === window.document) ? 'document' : startingTag(element);
        // Encode angle brackets to prevent input from being sanitized to empty string #8683.
        throw ngMinErr(
            'btstrpd',
            "App already bootstrapped with this element '{0}'",
            tag.replace(/</,'&lt;').replace(/>/,'&gt;'));
      }

      modules = modules || [];
      //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
      modules.unshift(['$provide', function($provide) {
        $provide.value('$rootElement', element);
      }]);

      if (config.debugInfoEnabled) {
        // Pushing so that this overrides `debugInfoEnabled` setting defined in user's `modules`.
        modules.push(['$compileProvider', function($compileProvider) {
          $compileProvider.debugInfoEnabled(true);
        }]);
      }
      //ng模块放入第一位
      modules.unshift('ng');
      var injector = createInjector(modules, config.strictDi);
      injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector',
         function bootstrapApply(scope, element, compile, injector) {
          scope.$apply(function() {
            element.data('$injector', injector);
            compile(element)(scope);
          });
        }]
      );
      return injector;
    };

    var NG_ENABLE_DEBUG_INFO = /^NG_ENABLE_DEBUG_INFO!/;
    var NG_DEFER_BOOTSTRAP = /^NG_DEFER_BOOTSTRAP!/;

    if (window && NG_ENABLE_DEBUG_INFO.test(window.name)) {
      config.debugInfoEnabled = true;
      window.name = window.name.replace(NG_ENABLE_DEBUG_INFO, '');
    }

    if (window && !NG_DEFER_BOOTSTRAP.test(window.name)) {
      return doBootstrap();
    }

    window.name = window.name.replace(NG_DEFER_BOOTSTRAP, '');
    angular.resumeBootstrap = function(extraModules) {
      forEach(extraModules, function(module) {
        modules.push(module);
      });
      return doBootstrap();
    };

    if (isFunction(angular.resumeDeferredBootstrap)) {
      angular.resumeDeferredBootstrap();
    }
    }
```

element.injector在哪边定义的呢？

通过forEach遍历函数
```js
  //line 3347
  forEach({
  data: jqLiteData,
  inheritedData: jqLiteInheritedData,

  scope: function(element) {
    // Can't use jqLiteData here directly so we stay compatible with jQuery!
    return jqLite.data(element, '$scope') || jqLiteInheritedData(element.parentNode || element, ['$isolateScope', '$scope']);
  },

  isolateScope: function(element) {
    // Can't use jqLiteData here directly so we stay compatible with jQuery!
    return jqLite.data(element, '$isolateScope') || jqLite.data(element, '$isolateScopeNoTemplate');
  },

  controller: jqLiteController,

  injector: function(element) {
    return jqLiteInheritedData(element, '$injector');
  }}, function(fn, name) {
      //fn相当于value name相当于key
      //jQLite.prototype['injector'] =
      JQLite.prototype[name] = function(arg1, arg2) {
      var i, key;
      var nodeCount = this.length;

      //jqLiteHasClass是个只读函数，所以需要特殊处理
      // jqLiteHasClass has only two arguments, but is a getter-only fn, so we need to special-case it
      // in a way that survives minification.
      //缩小范围
      // jqLiteEmpty takes no arguments but is a setter.
      //jqLiteEmpty 无参函数 但是 是一个setter方法

      if (fn !== jqLiteEmpty &&
          (isUndefined((fn.length == 2 && (fn !== jqLiteHasClass && fn !== jqLiteController)) ? arg1 : arg2))) {
        if (isObject(arg1)) {

          // we are a write, but the object properties are the key/values
          for (i = 0; i < nodeCount; i++) {
            if (fn === jqLiteData) {
              // data() takes the whole object in jQuery
              //this[i] 是通过jqLiteAddNodes函数往jqLite对象中加入索引
              fn(this[i], arg1);
            } else {
              for (key in arg1) {
                fn(this[i], key, arg1[key]);
              }
            }
          }
          // return self for chaining
          return this;
        } else {
          //element.injector() 走到这里
          // we are a read, so read the first child.
          // TODO: do we still need this?
          var value = fn.$dv;
          // Only if we have $dv do we iterate over all, otherwise it is just the first element.
          var jj = (isUndefined(value)) ? Math.min(nodeCount, 1) : nodeCount;
          for (var j = 0; j < jj; j++) {
            var nodeValue = fn(this[j], arg1, arg2);
            value = value ? value + nodeValue : nodeValue;
          }
          return value;
        }
      } else {
        // we are a write, so apply to all children
        for (i = 0; i < nodeCount; i++) {
          fn(this[i], arg1, arg2);
        }
        // return self for chaining
        return this;
      }
    };
  });
```

### jqLiteInheritedData

```js
  function jqLiteInheritedData(element, name, value) {
    // if element is the document object work with the html element instead
    // this makes $(document).scope() possible
    if (element.nodeType == NODE_TYPE_DOCUMENT) {
      element = element.documentElement;
    }
    var names = isArray(name) ? name : [name];

    while (element) {
      for (var i = 0, ii = names.length; i < ii; i++) {
        if (isDefined(value = jqLite.data(element, names[i]))) return value;
      }

      // If dealing with a document fragment node with a host element, and no parent, use the host
      // element as the parent. This enables directives within a Shadow DOM or polyfilled Shadow DOM
      // to lookup parent controllers.
      element = element.parentNode || (element.nodeType === NODE_TYPE_DOCUMENT_FRAGMENT && element.host);
    }
  }
```

### jqLite.data(jqLiteData)：

```js
    function jqLiteData(element, key, value) {
      if (jqLiteAcceptsData(element)) {

        var isSimpleSetter = isDefined(value);
        //element.injector() 就是 获取$injector
        //那么就是isSimpleGetter
        var isSimpleGetter = !isSimpleSetter && key && !isObject(key);
        var massGetter = !key;

        //element.injector()
        //!isSimpleGetter传false
        var expandoStore = jqLiteExpandoStore(element, !isSimpleGetter);
        var data = expandoStore && expandoStore.data;

        if (isSimpleSetter) { // data('key', value)
          data[key] = value;
        } else {
          if (massGetter) {  // data()
            return data;
          } else {
            if (isSimpleGetter) { // data('key')
              // don't force creation of expandoStore if it doesn't exist yet
              return data && data[key];
            } else { // mass-setter: data({key1: val1, key2: val2})
              extend(data, key);
            }
          }
        }
      }
  }
```

### jqLiteExpandoStore函数

```js
  function jqLiteExpandoStore(element, createIfNecessary) {
  var expandoId = element.ng339,
      expandoStore = expandoId && jqCache[expandoId];

  if (createIfNecessary && !expandoStore) {
    element.ng339 = expandoId = jqNextId();
    expandoStore = jqCache[expandoId] = {events: {}, data: {}, handle: undefined};
  }

  return expandoStore;
  }
```

### jqCache对象

`var jqCache = JQLite.cache = {};`
