title: jquery中$(this)的流程
date: 2016-08-31 21:52:56
tags: jquery $(this)
---

##   jquery1.0源码解读

### $(this)的流程是怎样的

假定$(this)存在于这样一段代码：
```javascript

	$("#child").each(function(){

 	 	var self = $(this);
 	})
```
那么$(this)会首先进入
### jQuery函数
```javascript
		function jQuery(a,c) {

	    // Shortcut for document ready (because $(document).each() is silly)
	    //处理 ready函数,$(function(){})
		if ( a && a.constructor == Function && jQuery.fn.ready )
			return jQuery(document).ready(a);

		// Make sure that a selection was provided
		a = a || jQuery.context || document;

		// Watch for when a jQuery object is passed as the selector
		//如果a 是jQuery对象，把a和空数组合并，然后返回，这样做的目的是不破坏原来的jQuery对象。
	    //（注：jquery属性是每个jQuery对象都有的，值为jQuery的版本。
		if (a.jquery)
			return $( jQuery.merge( a, [] ) );

		// Watch for when a jQuery object is passed at the context
	    //如果c是jQuery对象，调用find函数，去查找
		if ( c && c.jquery )
			return $( c ).find(a);
		
		// If the context is global, return a new object
		//jquery("#example") 第一次 进入  返回 new Jquery(),
		//进入jQuery.find
		//直接调用jQuery() 那么this == window,再调用 就返回new jQuery()
		if ( window == this )
			return new jQuery(a,c);

		// Handle HTML strings
		//处理html字符串
		//以非<开头 当中<> 非>结束
		//如果a是html代码，$("<div/>")，把html代码转成Dom元素
	    //jQuery.clean 就是把html代码 转换成Dom元素数组
		var m = /^[^<]*(<.+>)[^>]*$/.exec(a);
		if ( m ) a = jQuery.clean( [ m[1] ] );

		// Watch for when an array is passed in
		//如果a是数组或类数组，并且里面装的都是dom元素，把a和空数组合并一下
		//如果是其他情况，就调用find函数,find函数是处理css表达式的
	    //最后调用get方法，做出jQuery对象返回
		this.get( a.constructor == Array || a.length && !a.nodeType && a[0] != undefined && a[0].nodeType
	     ?
		// Assume that it is an array of DOM Elements
	        //假设是个Dom元素数组 ，那么合并
			jQuery.merge( a, [] ) :

			// Find the matching elements and save them for later
			//调用jQuery 静态方法 通过Jquery.extend扩展
			//$(this)就直接调用jQuery静态方法find line:588
			jQuery.find( a, c ) );

	  // See if an extra function was provided
	  //参数里是否有扩展方法提供
		var fn = arguments[ arguments.length - 1 ];
		
		// If so, execute it in context
		if ( fn && fn.constructor == Function )
			this.each(fn);//通过jQuery.prototype  原型链
	}
```
随后根据条件会进入
### new jQuery(a,c)
```javascript
	if ( window == this )
		return new jQuery(a,c);
```
那么就又会进入jQuery函数，这次就会进入
### this.get函数
```javascript
		this.get( a.constructor == Array || a.length && !a.nodeType && a[0] != undefined && a[0].nodeType ?
			// Assume that it is an array of DOM Elements
	        //假设是个Dom元素数组 ，那么合并
			jQuery.merge( a, [] ) :

			// Find the matching elements and save them for later
			//调用jQuery 静态方法 通过Jquery.extend扩展
			//$(this)就直接调用jQuery静态方法find line:588
			jQuery.find( a, c ) );
```
get函数中因为a此时为页面元素，所以根据条件会进入
### jQuery.find静态函数
```javascript
	find: function( t, context ) {
		// Make sure that the context is a DOM Element
		if ( context && context.nodeType == undefined )
			context = null;

		// Set the correct context (if none is provided)
        //设置当前上下文为Document
		context = context || jQuery.context || document;
	
		if ( t.constructor != String ) return [t];
		//....下面代码省略
```
因为满足t.constructor不为string，所以返回[t]数组，t就为当前元素。
再进入jQuery对象函数this.get即
### jQuery.fn.get函数:
```javascript
	get: function( num ) {
		// Watch for when an array (of elements) is passed in
		if ( num && num.constructor == Array ) {

			// Use a tricky hack to make the jQuery object
		    // look and feel like an array
		    //var obj = new Object();
		    //obj.length =0;
		    //[].push.apply(obj,[1,2,3])
		    //console.log(obj.length)
            //3
            //$(this) 又返回一个jQuery对象，又可以调用jQuery的各种方法
			this.length = 0;
			[].push.apply( this, num );
			
			return this;
		} else
			return num == undefined ?

				// Return a 'clean' array
				jQuery.map( this, function(a){ return a } ) :

				// Return just the object
				this[num];
	}
```
最后就调用
### [].push.apply
```javascript
	    this.length = 0;
		[].push.apply( this, num );
		return this;
```
返回jQuery对象数组并包含当前元素