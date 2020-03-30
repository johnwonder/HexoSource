title: angularjs1.5.8源码解析之extend函数
date: 2020-03-28 21:33:56
tags: angular
---

angularjs中很多常用函数在我们平时的js开发中很有帮助，可以极大的帮助我们提高开发效率，
今天我们继续来学习下它里面的一个频繁使用的函数extend.

## extend函数

文档中是这样描述的：Extends the destination object `dst` by copying own enumerable properties from the `src` object(s) to `dst`。
翻译下就是通过拷贝src对象的枚举属性到dst对象来实现扩展dst对象。
```js
  function extend(dst) {
  //slice 返回一个新的数组，包含从 start 到 end （如果指定end，不包括该元素）的 arrayObject 中的元素。
  //end 可选。规定从何处结束选取。该参数是数组片断结束处的数组下标。
  //如果没有指定该参数，那么切分的数组包含从 start 到数组结束的所有元素。如果这个参数是负数，那么它规定的是从数组尾部开始算起的元素。
  return baseExtend(dst, slice.call(arguments, 1), false);
  }
```
其实该函数的核心是内部baseExtend函数：

## baseExtend函数

deep参数为true代表是深拷贝，深拷贝简单点说就是通过递归拷贝把所有内部对象克隆复制。
```js
  function baseExtend(dst, objs, deep) {
    var h = dst.$$hashKey;

    for (var i = 0, ii = objs.length; i < ii; ++i) {
      var obj = objs[i];
       //obj 不是对象 且不是函数 那就跳过
      if (!isObject(obj) && !isFunction(obj)) continue;
      var keys = Object.keys(obj);
      for (var j = 0, jj = keys.length; j < jj; j++) {
        var key = keys[j];
        var src = obj[key];

        //如果深拷贝 且当前对象是object类型
        if (deep && isObject(src)) {
          //如果是日期
          if (isDate(src)) {
            dst[key] = new Date(src.valueOf());
          } else if (isRegExp(src)) {
            //如果是正则
            dst[key] = new RegExp(src);
          } else if (src.nodeName) {
            //如果是dom节点
            dst[key] = src.cloneNode(true);
          } else if (isElement(src)) {
            //如果是dom元素或者 jQuery对象
            dst[key] = src.clone();
          } else {
            //先把dst[key]赋空对象或空数组
            if (!isObject(dst[key])) dst[key] = isArray(src) ? [] : {};
            //递归调用
            baseExtend(dst[key], [src], true);
          }
        } else {
          dst[key] = src;
        }
      }
    }

    return dst;
  }
```

angularjs另外一个根extend对应的merge函数就是baseExtend深拷贝的实现。

## merge函数

```js
  function merge(dst) {
  return baseExtend(dst, slice.call(arguments, 1), true);
  }
```
