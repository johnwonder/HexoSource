title: angular1.5.8源码解析之JQLite函数
date: 2017-04-06 14:01:55
tags: angular
---

前言：为了搞清楚上篇文章提到的data函数，我们必须先讲angularjs的JQLite库。
angularjs有个内嵌的轻量级的jquery:jqLite，它的参数只有两种，一种是Dom元素，一种是类似html元素的字符串
## JQLite定义

从定义可以看到，JQLite不支持不带html格式的字符串传入：
```js
  function JQLite(element) {
    //如果传入的参数已经是JQLite实例，那直接返回
    if (element instanceof JQLite) {
      return element;
    }
    var argIsString;

    if (isString(element)) {
      element = trim(element);
      argIsString = true;
    }
    //this 有可能就是window
    //如果参数是字符串 且不是以<开头
    if (!(this instanceof JQLite)) {
      if (argIsString && element.charAt(0) != '<') {
        throw jqLiteMinErr('nosel', 'Looking up elements via selectors is not supported by jqLite!');
      }
      return new JQLite(element);
    }
    //字符串还需要调用jqLiteParseHTML函数解析
    if (argIsString) {
      jqLiteAddNodes(this, jqLiteParseHTML(element));
    } else {
      jqLiteAddNodes(this, element);
    }
  }
```
如果参数以<开头那还需要```jqLiteParseHTML```函数来解析。

## jqLiteParseHTML

```js
  //简单的标记正则
  var SINGLE_TAG_REGEXP = /^<([\w-]+)\s*\/?>(?:<\/\1>|)$/;

  function jqLiteParseHTML(html, context) {
    context = context || window.document;
    var parsed;

    if ((parsed = SINGLE_TAG_REGEXP.exec(html))) {
      //parsed[1]是类似 div 的字符串
      return [context.createElement(parsed[1])];
    }
    //制造Fragment
    if ((parsed = jqLiteBuildFragment(html, context))) {
      return parsed.childNodes;
    }

    return [];
  }
```
SINGLE_TAG_REGEXP中有几点知识点：

1. 非获取匹配，匹配pattern但不获取匹配结果，不进行存储供以后使用。这在使用或字符“(|)”来组合一个模式的各个部分时很有用。
   例如“industr(?:y|ies)”就是一个比“industry|industries”更简略的表达式

2. \1  \2......  都要和正则表达式集合()一起使用,简单的说就是\1表示重复正则第一个圆括号内匹配到的内容
    \2表示重复正则第二个圆括号内匹配到的内容

3. ?匹配前面的子表达式零次或一次。例如，“do(es)?”可以匹配“do”或“does”。?等价于{0,1}。

如果符合正则SINGLE_TAG_REGEXP那就通过document.createElement来返回，否则 通过jqLiteBuildFragment
函数来构建html片段，关于jqLiteBuildFragment我们下篇文章再分析。

最后调用jqLiteAddNodes把元素放入JQLite对象中。

## jqLiteAddNodes

```js
    function jqLiteAddNodes(root, elements) {

      if (elements) {
        // if a Node (the most common case)
        //绝大多数情况是单个节点
        if (elements.nodeType) {
          root[root.length++] = elements;
        } else {
          var length = elements.length;

          // if an Array or NodeList and not a Window
          //如果是数组或者NodeList 且不是window
          if (typeof length === 'number' && elements.window !== elements) {
            if (length) {
              for (var i = 0; i < length; i++) {
                root[root.length++] = elements[i];
              }
            }
          } else {
            root[root.length++] = elements;
          }
        }
      }
    }
```
我们通过```bindJquery```方法绑定jQuery到jqLite

```js
//try to bind to jquery now so that one can write jqLite(document).ready()
//but we will rebind on bootstrap again.
bindJQuery();
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
var jqName = jq(); // 调用jq方法返回jq库名称
jQuery = isUndefined(jqName) ? window.jQuery :   // use jQuery (if present)
         !jqName             ? undefined     :   // use jqLite
                               window[jqName];   // use jQuery specified by `ngJq`

if (jQuery && jQuery.fn.on) {
  jqLite = jQuery;
  //扩展fn函数
   //省略部分代码，
   //重写了jQuery的cleanData,扩展了jQuery.fn
} else {
  jqLite = JQLite;
}

angular.element = jqLite;
// Prevent double-proxying.
//阻止双代理
bindJQueryFired = true;
}
```

到到这里我们发现angular.element就是jqLite函数。
