title: angular-bindjquery
date: 2017-01-19 16:30:17
tags: angular
---

## angular1.5.8 绑定jquery库

先看jq方法:  

```js
  var jq = function() {
  if (isDefined(jq.name_)) return jq.name_;
  var el;
  var i, ii = ngAttrPrefixes.length, prefix, name;
  for (i = 0; i < ii; ++i) {
    //line 1513 var ngAttrPrefixes = ['ng-', 'data-ng-', 'ng:', 'x-ng-'];
    prefix = ngAttrPrefixes[i];
    if (el = window.document.querySelector('[' + prefix.replace(':', '\\:') + 'jq]')) {
      name = el.getAttribute(prefix + 'jq');
      break;//只会获取一次
    }
  }

  return (jq.name_ = name);
  };
```

```js
  var bindJQueryFired = false;
  function bindJQuery() {
  var originalCleanData;

  if (bindJQueryFired) {
    return;
  }
  //http://div.io/topic/1154
  // bind to jQuery if present;
  var jqName = jq(); // 调用jq方法
  jQuery = isUndefined(jqName) ? window.jQuery :   // use jQuery (if present)
           !jqName             ? undefined     :   // use jqLite
                                 window[jqName];   // use jQuery specified by `ngJq`

  // Use jQuery if it exists with proper functionality, otherwise default to us.
  // Angular 1.2+ requires jQuery 1.7+ for on()/off() support.
  //要支持on off 方法
  // Angular 1.3+ technically requires at least jQuery 2.1+ but it may work with older
  // versions. It will not work for sure with jQuery <1.7, though.
  if (jQuery && jQuery.fn.on) {
    jqLite = jQuery;
    extend(jQuery.fn, {
      scope: JQLitePrototype.scope,
      isolateScope: JQLitePrototype.isolateScope,
      controller: JQLitePrototype.controller,
      injector: JQLitePrototype.injector,
      inheritedData: JQLitePrototype.inheritedData
    });

    //monkey patch (猴子补丁)
    //用来在运行时动态修改已有的代码，而不需要修改原始代码。
    //http://blog.csdn.net/fwenzhou/article/details/8742838
    // All nodes removed from the DOM via various jQuery APIs like .remove()
    // are passed through jQuery.cleanData. Monkey-patch this method to fire
    // the $destroy event on all removed nodes.
    originalCleanData = jQuery.cleanData;
    jQuery.cleanData = function(elems) {
      var events;
      for (var i = 0, elem; (elem = elems[i]) != null; i++) {
        events = jQuery._data(elem, "events");
        if (events && events.$destroy) {
          jQuery(elem).triggerHandler('$destroy');
        }
      }
      //最终调用originalCleanData方法 就是jquery.cleanData函数
      originalCleanData(elems);
    };
  } else {
    jqLite = JQLite;//内置的JQLite 库
  }

  angular.element = jqLite;

  // Prevent double-proxying.
  //阻止双代理
  bindJQueryFired = true;
  }
```

我们来看内置的JQLite库：  

```js
    //line 2968
    function JQLite(element) {
    if (element instanceof JQLite) {
    return element;
    }

    var argIsString;

    if (isString(element)) {
    element = trim(element);
    argIsString = true;
    }
    if (!(this instanceof JQLite)) {
    if (argIsString && element.charAt(0) != '<') {
      throw jqLiteMinErr('nosel', 'Looking up elements via selectors is not supported by jqLite! See: http://docs.angularjs.org/api/angular.element');
    }
    return new JQLite(element);
    }

    if (argIsString) {
    jqLiteAddNodes(this, jqLiteParseHTML(element));
    } else {
    jqLiteAddNodes(this, element);
    }
    }
```
参考资料:  
[何使用angular.js中的jqlite](https://segmentfault.com/q/1010000000599102)
[Angular内嵌jqlite语法大全](http://www.jianshu.com/p/e184fc15f3f4)
