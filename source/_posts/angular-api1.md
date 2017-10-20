title: angular_api1
date: 2017-06-16 21:57:07
tags: angular
---

## angular基础api

### angular.isNumber
```js
/**
 * @ngdoc function
 * @name angular.isNumber
 * @module ng
 * @kind function
 *
 * @description
 * Determines if a reference is a `Number`.
 *判断一个引用是否是数字
 * This includes the "special" numbers `NaN`, `+Infinity` and `-Infinity`.
 *包含特殊number NaN `+Infinity` and `-Infinity`.
 * If you wish to exclude these then you can use the native
 * [`isFinite'](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/isFinite)
 * method.
 *
 * @param {*} value Reference to check.
 * @returns {boolean} True if `value` is a `Number`.
 */
function isNumber(value) {return typeof value === 'number';}
```


### nextUid

```js
/**
 * A consistent(一致的) way of creating unique IDs in angular.
 *
 * Using simple numbers allows us to generate 28.6 million unique ids per second for 10 years before
 * we hit number precision issues in JavaScript.
 *
 * Math.pow(2,53) 2的53次幂/ 60 / 60 / 24 / 365 / 10 = 28.6M
 *
 * @returns {number} an unique alpha-numeric string
 */
function nextUid() {
  return ++uid;
}
```

10年内每秒可以生成 28.6 million的唯一标识
[ js的精确整数最大为:Math.pow(2,53)-1 =9007199254740991.](http://blog.csdn.net/zhaokuner/article/details/23246795)
