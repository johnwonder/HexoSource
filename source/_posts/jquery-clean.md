title: jquery_clean源码分析
date: 2016-09-05 22:16:24
tags: jquery clean
---

##   jquery1.0源码解读

### jQuery函数
```javascript
	// Handle HTML strings
	//处理html字符串
	//以非<开头 当中<> 非>结束
	//如果a是html代码，$("<div/>")，把html代码转成Dom元素
    //jQuery.clean 就是把html代码 转换成Dom元素数组
	var m = /^[^<]*(<.+>)[^>]*$/.exec(a);
	if ( m ) a = jQuery.clean( [ m[1] ] );
	...省略
```

```javascript
    //比如$('<div>123</div>,<a><223/a>')
    //用调用jQuery.clean方法返回
	clean: function(a) {
		var r = [];
		for ( var i = 0; i < a.length; i++ ) {
			if ( a[i].constructor == String ) {

				var table = "";
	            //indexOf 返回0 !0才是正确的
				if ( !a[i].indexOf("<thead") || !a[i].indexOf("<tbody") ) {
					table = "thead";
					a[i] = "<table>" + a[i] + "</table>";
				} else if ( !a[i].indexOf("<tr") ) {
					table = "tr";
					a[i] = "<table>" + a[i] + "</table>";
				} else if ( !a[i].indexOf("<td") || !a[i].indexOf("<th") ) {
					table = "td";
					a[i] = "<table><tbody><tr>" + a[i] + "</tr></tbody></table>";
				}
	
				var div = document.createElement("div");
				div.innerHTML = a[i];//构造div 把table放在div中。
	
				if ( table ) {
					div = div.firstChild;
					if ( table != "thead" ) div = div.firstChild;
					if ( table == "td" ) div = div.firstChild;
				}
	
				for ( var j = 0; j < div.childNodes.length; j++ )
					r.push( div.childNodes[j] );
				} else if ( a[i].jquery || a[i].length && !a[i].nodeType )
					for ( var k = 0; k < a[i].length; k++ )
						r.push( a[i][k] );
				else if ( a[i] !== null )
					r.push(	a[i].nodeType ? a[i] : document.createTextNode(a[i].toString()) );
		}
		return r;
	}
```
[jQuery.clean()方法源码分析（一）](http://www.cnblogs.com/yy-hh/p/4517223.html)
[[原创] jQuery源码分析-13 CSS操作-CSS-样式表-jQuery.fn.css()](http://www.cnblogs.com/nuysoft/archive/2011/12/22/2297923.html)
