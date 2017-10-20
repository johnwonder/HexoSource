title: javascript_replace
date: 2017-04-21 11:34:17
tags: javascript
---

## javascript replace

在看angular源码时 有这么一个方法
```js

//. 匹配除“\r\n”之外的任何单个字符。要匹配包括“\r\n”在内的任何字符，请使用像“[\s\S]”的模式
var SPECIAL_CHARS_REGEXP = /([\:\-\_]+(.))/g;
//感觉应该是/([\:\-\_])+(.)/g;
var MOZ_HACK_REGEXP = /^moz([A-Z])/;

function camelCase(name) {
  return name.
    replace(SPECIAL_CHARS_REGEXP, function(_, separator, letter, offset,all) {
      return offset ? letter.toUpperCase() : letter;
      //all 为字符串本身
    }).
    replace(MOZ_HACK_REGEXP, 'Moz$1');
}
```

怎么使用呢？

```js
var PREFIX_REGEXP = /^((?:x|data)[\:\-_])/i;

var result = 'ng-app'.replace(PREFIX_REGEXP, '');

console.log(camelCase(result));
//输出ngApp
```

第二个参数为函数：

在ECMAScript3推荐使用函数方式，实现于JavaScript1.2.当replace方法执行的时候每次都会调用该函数，返回值作为替换的新值。

 函数参数的规定：

第一个参数为每次匹配的全文本（$&）。
中间参数为子表达式匹配字符串，个数不限.( $i (i:1-99))
倒数第二个参数为匹配文本字符串的匹配下标位置。
最后一个参数表示字符串本身。

参考资料：
[js的replace函数入参为function时的疑问](https://segmentfault.com/q/1010000003949349)
[JavaScript 函数replace揭秘](http://www.cnblogs.com/whitewolf/archive/2013/03/14/2958720.html)
