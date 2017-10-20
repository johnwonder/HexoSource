title: jquery-each
date: 2015-09-24 14:22:05
tags: jquery
---

##   jquery1.0源码解读

### each函数

1.0源码:
```js
	each: function( obj, fn, args ) {
		if ( obj.length == undefined )//如果不是数组
			for ( var i in obj )
				fn.apply( obj[i], args || [i, obj[i]] );
		else
			for ( var i = 0; i < obj.length; i++ )
				fn.apply( obj[i], args || [i, obj[i]] );
		return obj;
	}
```
遍历obj对象，直接用fn.apply调用。

再看1.2.6源码：
```js
	each: function( object, callback, args ) {
		var name, i = 0, length = object.length;

		if ( args ) {
			if ( length == undefined ) {
				for ( name in object )
					if ( callback.apply( object[ name ], args ) === false )
						break;
			} else
				for ( ; i < length; )
					if ( callback.apply( object[ i++ ], args ) === false )
						break;

		// A special, fast, case for the most common use of each
		} else {
			if ( length == undefined ) {
				for ( name in object )
					if ( callback.call( object[ name ], name, object[ name ] ) === false )
						break;
			} else
				for ( var value = object[0];
					i < length && callback.call( value, i, value ) !== false; value = object[++i] ){}
		}

		return object;
	}
```
最主要是对第三个参数的调用做修改优化,1.0直接 args || [i,obj[i]],而1.2.6
分开判断 ，在没有args的情况下 用call调用。而且把object.length 先保存在length变量中。
