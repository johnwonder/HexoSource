title: angularjs1.5.8源码分析之foreach函数
date: 2017-02-21 10:23:12
tags: angular
---

## foreach函数分析

```js
/*
 * @param {Object|Array} obj Object to iterate over.
   //obj是需要遍历的数组或者对象
 * @param {Function} iterator Iterator function.
   iterator是迭代器函数
 * context是执行迭代器函数中的上下文对象
 * @param {Object=} context Object to become context (`this`) for the iterator function.
 * @returns {Object|Array} Reference to `obj`.
 *返回obj对象的引用
 */
function forEach(obj, iterator, context) {
    var key, length;
    //先判断obj是否存在
    if (obj) {
      //如果obj是函数
      if (isFunction(obj)) {

        for (key in obj) {
          //检查hasOwnProperty是否存在,ie8上querySelectorAll的结果是不带
          //hasOwnProperty函数的对象
          // Need to check if hasOwnProperty exists,
          // as on IE8 the result of querySelectorAll is an object without a hasOwnProperty function
          if (key != 'prototype' && key != 'length' && key != 'name' && (!obj.hasOwnProperty || obj.hasOwnProperty(key))) {
            //先传value再传key
            iterator.call(context, obj[key], key, obj); //调用iterator 函数
          }
        }
      } else if (isArray(obj) || isArrayLike(obj)) {
        //如果obj是数组或类数组
        var isPrimitive = typeof obj !== 'object';//是否是原生的数组
        for (key = 0, length = obj.length; key < length; key++) {
          if (isPrimitive || key in obj) {
            //执行迭代器
            iterator.call(context, obj[key], key, obj);
          }
        }
      } else if (obj.forEach && obj.forEach !== forEach) {
          //执行对象本身的foreach函数
          obj.forEach(iterator, context, obj);
      } else if (isBlankObject(obj)) {
        // createMap() fast path --- Safe to avoid hasOwnProperty check because prototype chain is empty
        for (key in obj) {
          iterator.call(context, obj[key], key, obj);
        }
      } else if (typeof obj.hasOwnProperty === 'function') {

        //存在hasOwnProperty方法
        // Slow path for objects inheriting Object.prototype, hasOwnProperty check needed
        //继承自Object.prototype的对象
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
//自定义的hasOwnProperty是Object原型里的函数
var hasOwnProperty = Object.prototype.hasOwnProperty;
//根据对象的key来做排序
function forEachSorted(obj, iterator, context) {
  var keys = Object.keys(obj).sort();
  for (var i = 0; i < keys.length; i++) {
    iterator.call(context, obj[keys[i]], keys[i]);
  }
  return keys;
}
```

我们可以看见里面有很多自定义的函数,比如```isFunction```,```isArray```,```isArrayLike```,```isBlankObject```。
之后我们会逐个分析。

在```publishExternalAPI```方法中通过extend方法定义，然后就可以通过angular.foreach调用了。

### foreach函数用法

```js
    var values = {name: 'misko', gender: 'male'};
    var log = [];
    //value在前，key在后
    angular.forEach(values, function(value, key) {
      this.push(key + ': ' + value);
    }, log);
    expect(log).toEqual(['name: misko', 'gender: male']);
  ```

  angularjs内部很多地方用到了它自定义的foreach函数，通过研究它的实现我们就可以在平时的开发中运用它foreach的设计
  来定义一个适合自己的遍历函数了。
