title: angular_api
date: 2017-06-06 21:49:46
tags: angular
---

## angular 基础api介绍

### angular.isString

angular.isString判断输入参数是否是字符串,源码如下:
```js
/**
* @ngdoc function
* @name angular.isString
* @module ng
* @kind function 函数
*
* @description
* Determines if a reference is a `String`.
*
* @param {*} value Reference to check. 参数
* @returns {boolean} True if `value` is a `String`. 返回
*/
function isString(value) {return typeof value === 'string';}
```

### angular.isObject

angular.isObject 判断输入参数是否是Object类型
```js
/**
* @ngdoc function
* @name angular.isObject
* @module ng
* @kind function
*
* @description
* Determines if a reference is an `Object`. Unlike `typeof` in JavaScript, `null`s are not
* considered to be objects. Note that JavaScript arrays are objects.
在angular中null是不被考虑为object的。  JavaScript的array是object
*
* @param {*} value Reference to check.
* @returns {boolean} True if `value` is an `Object` but not `null`.
*/
function isObject(value) {
// http://jsperf.com/isobject4
return value !== null && typeof value === 'object';
}
```

### angular.isNullObject

判断引用参数是否是带空原型链的对象
```js
/**
* Determine if a value is an object with a null prototype

*
* @returns {boolean} True if `value` is an `Object` with a null prototype
*/
function isBlankObject(value) {
//var getPrototypeOf  = Object.getPrototypeOf,
return value !== null && typeof value === 'object' && !getPrototypeOf(value);
}
```

### isArrayLike

```js
/**
* @private
* @param {*} obj
* @return {boolean} Returns true if `obj` is an array or array-like object (NodeList, Arguments,
*                   String ...)
如果obj参数是数组或者类似数组的对象(节点列表,参数,字符串)
*/
function isArrayLike(obj) {

//null undefined window 不是array-like
// `null`, `undefined` and `window` are not array-like
if (obj == null || isWindow(obj)) return false;

// arrays, strings and jQuery/jqLite objects are array like
// * jqLite is either the jQuery or jqLite constructor function
// * we have to check the existence of jqLite first as this method is called
//必须先检测jqLite是否存在
//因为当jqLite实例化时forEach调用了这个方法 
//   via the forEach method when constructing the jqLite object in the first place
if (isArray(obj) || isString(obj) || (jqLite && obj instanceof jqLite)) return true;

// Support: iOS 8.2 (not reproducible in simulator)
// "length" in obj used to prevent JIT error (gh-11508)
var length = "length" in Object(obj) && obj.length;

// NodeList objects (with `item` method) and
// other objects with suitable length characteristics are array-like
return isNumber(length) &&
  (length >= 0 && ((length - 1) in obj || obj instanceof Array) || typeof obj.item == 'function');

}
```
