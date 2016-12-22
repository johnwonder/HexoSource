title: jquery之map函数解析
date: 2016-08-27 21:10:02
tags: jquery map
---

##   jquery1.0源码解读

### filter函数

1.0源码:
```js
	map: function(elems, fn) {
		// If a string is passed in for the function, make a function
		// for it (a handy shortcut)
		if ( fn.constructor == String )
			fn = new Function("a","return " + fn);

		var result = [];//初始化一个空数组

		// Go through the array, translating each of the items to their
		// new value (or values).
		for ( var i = 0; i < elems.length; i++ ) {
			var val = fn(elems[i],i);//调用参数fn 函数 返回fn中的新结果

			//跟grep的函数区别就在于返回的是函数中新的返回值
			//这就是映射
			//而不是返回满足函数条件的旧元素
			if ( val !== null && val != undefined ) {
				if ( val.constructor != Array ) val = [val];
				result = jQuery.merge( result, val );//合并
			}
		}

		return result;
	}
```
