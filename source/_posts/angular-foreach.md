title: angular_foreach
date: 2017-02-21 10:23:12
tags: angular
---

## angular1.5.8 foreach 源码分析

```js
/*
 * @param {Object|Array} obj Object to iterate over.//需要遍历的数组或者对象
 * @param {Function} iterator Iterator function.函数类型
 * 执行iterator函数中的上下文
 * @param {Object=} context Object to become context (`this`) for the iterator function.
 * @returns {Object|Array} Reference to `obj`.
 *返回obj对象的引用
 */
function forEach(obj, iterator, context) {
    var key, length;
    if (obj) {//先判断obj是否存在
      if (isFunction(obj)) {
        for (key in obj) {
          //检查hasOwnProperty是否存在
          // Need to check if hasOwnProperty exists,
          // as on IE8 the result of querySelectorAll is an object without a hasOwnProperty function
          if (key != 'prototype' && key != 'length' && key != 'name' && (!obj.hasOwnProperty || obj.hasOwnProperty(key))) {
            iterator.call(context, obj[key], key, obj); //调用iterator 函数
          }
        }
      } else if (isArray(obj) || isArrayLike(obj)) {
        var isPrimitive = typeof obj !== 'object';//是否是原生的数组
        for (key = 0, length = obj.length; key < length; key++) {
          if (isPrimitive || key in obj) {
            iterator.call(context, obj[key], key, obj);
          }
        }
      } else if (obj.forEach && obj.forEach !== forEach) {
          obj.forEach(iterator, context, obj);
      } else if (isBlankObject(obj)) {
        // createMap() fast path --- Safe to avoid hasOwnProperty check because prototype chain is empty
        for (key in obj) {
          iterator.call(context, obj[key], key, obj);
        }
      } else if (typeof obj.hasOwnProperty === 'function') {//存在hasOwnProperty方法
        // Slow path for objects inheriting Object.prototype, hasOwnProperty check needed
        for (key in obj) {
          if (obj.hasOwnProperty(key)) {
            iterator.call(context, obj[key], key, obj);
          }
        }
      } else {
        // Slow path for objects which do not have a method `hasOwnProperty`
        for (key in obj) {
          if (hasOwnProperty.call(obj, key)) {//调用hasOwnProperty方法成功后
            iterator.call(context, obj[key], key, obj);
          }
        }
      }
    }
    return obj;
}

function forEachSorted(obj, iterator, context) {
  var keys = Object.keys(obj).sort();
  for (var i = 0; i < keys.length; i++) {
    iterator.call(context, obj[keys[i]], keys[i]);
  }
  return keys;
}
```

### 定义

在publishExternalAPI方法中通过extend方法定义，然后就可以通过angular.foreach调用了。

### 用法

```js
    var values = {name: 'misko', gender: 'male'};
    var log = [];
    //value在前，key在后
    angular.forEach(values, function(value, key) {
      this.push(key + ': ' + value);
    }, log);
    expect(log).toEqual(['name: misko', 'gender: male']);
  ```
