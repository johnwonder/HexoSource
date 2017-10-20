title: angular_指令标准化
date: 2017-08-30 10:34:14
tags: angular
---

## angular 指令匹配

angular 官网自定义指令提到了如下信息：

AngularJS normalizes an element's tag and attribute name to determine which elements match which directives. We typically refer to directives by their case-sensitive camelCase normalized name (e.g. ngModel). However, since HTML is case-insensitive, we refer to directives in the DOM by lower-case forms, typically using dash-delimited attributes on DOM elements (e.g. ng-model).

The normalization process is as follows:

Strip x- and data- from the front of the element/attributes.
Convert the :, -, or _-delimited name to camelCase.

意思就是跳过x-和data-,然后 把:,-,_这种分隔字符 做驼峰处理。比如下面就比配ngBind指令

```html
  <span ng-bind="name"></span> <br/>
  <span ng:bind="name"></span> <br/>
  <span ng_bind="name"></span> <br/>
  <span data-ng-bind="name"></span> <br/>
  <span x-ng-bind="name"></span> <br/>
```

## angular内部源码

```js

var SPECIAL_CHARS_REGEXP = /([\:\-\_]+(.))/g;

/**
 * Converts snake_case to camelCase.
 * Also there is special case for Moz prefix starting with upper case letter.
 * @param name Name to normalize
 *骆驼 https://my.oschina.net/yongqing/blog/300313
 */
function camelCase(name) {
  return name.
    replace(SPECIAL_CHARS_REGEXP, function(_, separator, letter, offset) {
      return offset ? letter.toUpperCase() : letter;
    }).
    replace(MOZ_HACK_REGEXP, 'Moz$1');
}

//非获取匹配，匹配pattern但不获取匹配结果，不进行存储供以后使用。这在使用或字符“(|)”来组合一个模式的各个部分时很有用。
//例如“industr(?:y|ies)”就是一个比“industry|industries”更简略的表达式。
var PREFIX_REGEXP = /^((?:x|data)[\:\-_])/i;
//http://www.cnblogs.com/whitewolf/p/3495822.html
/**
 * Converts all accepted directives format into proper directive name.
 * @param name Name to normalize
 */
function directiveNormalize(name) {
  return camelCase(name.replace(PREFIX_REGEXP, ''));
}
```
