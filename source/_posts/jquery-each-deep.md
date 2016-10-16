title: jquery_each函数深入分析
date: 2016-09-01 21:58:35
tags: jquery each
---

##   jquery1.0源码解读

### $().each(function(){})的流程是怎样的

顾名思义,each就是对集合中的每个元素调用传入的function函数.

### jQuery.fn.each
```javascript
	//each实例方法
	each: function (fn, args) {
        //调用jQuery静态方法each
		return jQuery.each( this, fn, args );
	}
```

### jQuery.each
那就找到jQuery静态方法each:
```javascript
	each: function( obj, fn, args ) {
		if ( obj.length == undefined )//如果不是数组
			for ( var i in obj )
				fn.apply( obj[i], args || [i, obj[i]] );
		else
			for ( var i = 0; i < obj.length; i++ )
				//如果存在参数args那么就传索引和当前jquery元素给参数fn函数
				//fn内部引用的就是当前obj[i]元素
				fn.apply( obj[i], args || [i, obj[i]] );
		return obj;
	}
```

因为$(".li")返回的都是类数组对象，索引obj.length 就是当前数组元素的个数

$(".li")首先调用jQuery.find( a, c )静态函数,c为空

### jQuery.find
```javascript
	find: function( t, context ) {
		// Make sure that the context is a DOM Element
		if ( context && context.nodeType == undefined )
			context = null;

		// Set the correct context (if none is provided)
        //设置当前上下文为Document
		context = context || jQuery.context || document;
	
		if ( t.constructor != String ) return [t];
	
		if ( !t.indexOf("//") ) {
			context = context.documentElement;
			t = t.substr(2,t.length);
		} else if ( !t.indexOf("/") ) { // ".li".indexOf("/")为-1 那么就为false
			context = context.documentElement;
			t = t.substr(1,t.length);
			// FIX Assume the root element is right :(
			if ( t.indexOf("/") >= 1 )
				t = t.substr(t.indexOf("/"),t.length);
		}
	
		var ret = [context];
		var done = [];
		var last = null;
	
		while ( t.length > 0 && last != t ) {
			var r = [];
			last = t;
			
			//去除空格并把//替换为空
			t = jQuery.trim(t).replace( /^\/\//i, "" );
			
			var foundToken = false;
			//这里调用token 正则匹配 然后在map里传入function的字符串。。
			for ( var i = 0; i < jQuery.token.length; i += 2 ) {
				var re = new RegExp("^(" + jQuery.token[i] + ")");
				var m = re.exec(t);
				
				if ( m ) {
					r = ret = jQuery.map( ret, jQuery.token[i+1] );
					t = jQuery.trim( t.replace( re, "" ) );
					foundToken = true;
				}
			}
			
			if ( !foundToken ) {
				if ( !t.indexOf(",") || !t.indexOf("|") ) {
					if ( ret[0] == context ) ret.shift();
					done = jQuery.merge( done, ret );//把跟context一样的元素删除
					r = ret = [context];
					t = " " + t.substr(1,t.length);
				} else {
					var re2 = /^([#.]?)([a-z0-9\\*_-]*)/i;//# 或 .开头 
					var m = re2.exec(t);
		
					if ( m[1] == "#" ) {
						// Ummm, should make this work in all XML docs
						var oid = document.getElementById(m[2]);
						r = ret = oid ? [oid] : [];
						t = t.replace( re2, "" );
					} else {
						if ( !m[2] || m[1] == "." ) m[2] = "*";
		
						for ( var i = 0; i < ret.length; i++ )
							r = jQuery.merge( r,
								m[2] == "*" ?
									jQuery.getAll(ret[i]) :
									ret[i].getElementsByTagName(m[2])
							);
					}
				}
			}
	
			if ( t ) {
				var val = jQuery.filter(t,r);
				//fileter用来过滤 如 div#example
				//先把div过滤掉 然后 根据通过正则取出 example
				//再通过getAttribute('id') 比较id名称是否一样

				// $(".examle") 如果页面中有两个example元素
				//那么 经过jQuery.filter(".example",r)
				//r为遍历出来的 所有页面元素
				//val.r为 两个元素的数组
				//val.t为""
				ret = r = val.r;
				t = jQuery.trim(val.t);//用于遍历".example.e"这种情况
			}
		}
	
		if ( ret && ret[0] == context ) ret.shift();//弹出context返回数组原来的第一个元素的值 //该方法会改变数组的长度
		done = jQuery.merge( done, ret );//合并元素数组
	
		return done;
	}
```

### jQuery.token数组

一共8个，最后一个为function
```javacript
 	token: [
		"\\.\\.|/\\.\\.", "a.parentNode",
		">|/", "jQuery.sibling(a.firstChild)",
		"\\+", "jQuery.sibling(a).next",
		"~", function(a){
			var r = [];
			var s = jQuery.sibling(a);
			if ( s.n > 0 )
				for ( var i = s.n; i < s.length; i++ )
					r.push( s[i] );
			return r;
		}
	]
```
`".li"`一个都不匹配,进入`(!findToken)` 匹配`/^([#.]?)([a-z0-9\\*_-]*)/i;`，且顺序如下：

0: ".example"
1: "."
2: "example"

### jQuery.merge
进入jQuery.merge函数,此时m[2]为"*",r为[]数组：
```javascript
		jQuery.merge( r,
					 m[2] == "*" ?
					jQuery.getAll(ret[i]) :
					ret[i].getElementsByTagName(m[2])
		);
```

### jQuery.getAll静态函数：
```javascript
	getAll: function(o,r) {
		r = r || [];
		var s = o.childNodes;
		for ( var i = 0; i < s.length; i++ )
			if ( s[i].nodeType == 1 ) {
				r.push( s[i] );
				jQuery.getAll( s[i], r );
			}
		return r;
	}
```
getAll递归返回所有o的子节点

0: html
1: head
2: script
3: script
4: body
5: div.example
6: div.example
7: script

### jQuery.filter 函数过滤：
```javascript
	jQuery.filter(t,r);
```

### jQuery.merge再合并：
```javascript
	if ( ret && ret[0] == context ) ret.shift();//弹出context返回数组原来的第一个元素的值 //该方法会改变数组的长度
		done = jQuery.merge( done, ret );//合并元素数组
	
		return done;
```
返回的done就包含合并后的所有元素数组
