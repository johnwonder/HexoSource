title: angular_jqlite
date: 2017-04-06 14:01:55
tags: angular
---

## angular 1.5.8 angular.element

angular.element 不支持不带html格式的字符串传入，定义如下：

```js
  function JQLite(element) {
    if (element instanceof JQLite) {
      return element;
    }

    var argIsString;

    if (isString(element)) {
      element = trim(element);
      argIsString = true;
    }
    //this 有可能就是window
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

在bindJquery方法中调用 line 31764:  

```js
//try to bind to jquery now so that one can write jqLite(document).ready()
//but we will rebind on bootstrap again.
bindJQuery();

publishExternalAPI(angular);
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
    originalCleanData(elems);
  };
} else {
  jqLite = JQLite;
}

angular.element = jqLite;

// Prevent double-proxying.
bindJQueryFired = true;
}
```
