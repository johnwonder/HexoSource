title: angular-typedArray
date: 2017-01-08 13:46:09
tags: angular
---

## angular1.5.8中对于TypedArray的判断

```js

var TYPED_ARRAY_REGEXP = /^\[object (?:Uint8|Uint8Clamped|Uint16|Uint32|Int8|Int16|Int32|Float32|Float64)Array\]$/;

function isNumber(value) {return typeof value === 'number';}

function isTypedArray(value) {
  return value && isNumber(value.length) && TYPED_ARRAY_REGEXP.test(toString.call(value));
}
```

比如用如下代码测试:  

```js
  var _typedArray= new Uint8Array([-23]);

  console.log(isTypedArray(_typedArray));
```

输出为true

参考资料

[Javascript TypedArray 解惑：Uint8Array 与 Uint8ClampedArray 的区别](http://blog.csdn.net/cuixiping/article/details/42270561)  
[负数的二进制表示方法](http://www.360doc.com/content/12/0801/17/6828497_227700914.shtml)
