title: javascript之__proto__
date: 2017-07-28 16:20:20
tags: javascript
---

## proto作用

### 用法
今天在看chalk源码时，发现这么一段话：
```js
function build(_styles) {
	var builder = function () {
		return applyStyle.apply(builder, arguments);
	};

	builder._styles = _styles;
	builder.enabled = this.enabled;
	// __proto__ is used because we must return a function, but there is
	// no way to create a function with a different prototype.
	/* eslint-disable no-proto */
	builder.__proto__ = proto;

	return builder;
}
```
用__proto__是因为要返回一个方法，而且要带上一个不同的原型。

### 实验
我们用以下方法来做个测试：

```js
var proto1 = defineProps(function chalk1() {}, {

	'ss':  {
				get: function () {
				return  "ssssss";
			}
		}
});

console.log(proto1.ss);
function build(_styles) {
	var builder = function () {

	};

	// __proto__ is used because we must return a function, but there is
	// no way to create a function with a different prototype.
	/* eslint-disable no-proto */
	builder.__proto__ = proto1;
	return builder;
}

var _b = build();
console.log(_b.ss);
```

### 输出:
```js
ssssss
ssssss
```

参考资料：  
[create function in javascript with custom prototype](https://stackoverflow.com/questions/12443769/create-function-in-javascript-with-custom-prototype)  
[Is it possible to create a function with another prototype than Function.prototype?](https://stackoverflow.com/questions/5135326/is-it-possible-to-create-a-function-with-another-prototype-than-function-prototy)
