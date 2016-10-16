title: jquery_parents
date: 2016-09-07 22:02:04
tags: jquery parents
---

##   jquery1.0源码解读

## jquery1.0 版本的 静态parents函数

### jQuery.parents:
{% codeblock lang:javascript %}
	parents: function( elem ){
		var matched = [];
		var cur = elem.parentNode;
		while ( cur && cur != document ) {
			matched.push( cur );
			cur = cur.parentNode;
		}
		return matched;
	},
{% endcodeblock %}

通过jQuery.macros.axis来调用：

### jQuery.macros:
```javascript
	jQuery.each( jQuery.macros.axis, function(i,n){
			jQuery.fn[ i ] = function(a) {
				var ret = jQuery.map(this,n);
				if ( a && a.constructor == String )
					ret = jQuery.filter(a,ret).r;
				return this.pushStack( ret, arguments );
			};
		});
```

### 

```javascript
	jQuery.macros = {
		...省略部分代码
		axis: {

			parent: "a.parentNode",

			ancestors: jQuery.parents,

			parents: jQuery.parents,

			next: "jQuery.sibling(a).next",

			prev: "jQuery.sibling(a).prev",

			siblings: jQuery.sibling,

			children: "a.childNodes"
		}
		...省略部分代码
	}
```

而在1.2.6版本中，我们可以看到做出的变化：

```javascript
		jQuery.each({
		parent: function(elem){return elem.parentNode;},
		parents: function(elem){return jQuery.dir(elem,"parentNode");},
		next: function(elem){return jQuery.nth(elem,2,"nextSibling");},
		prev: function(elem){return jQuery.nth(elem,2,"previousSibling");},
		nextAll: function(elem){return jQuery.dir(elem,"nextSibling");},
		prevAll: function(elem){return jQuery.dir(elem,"previousSibling");},
		siblings: function(elem){return jQuery.sibling(elem.parentNode.firstChild,elem);},
		children: function(elem){return jQuery.sibling(elem.firstChild);},
		contents: function(elem){return jQuery.nodeName(elem,"iframe")?elem.contentDocument||elem.contentWindow.document:jQuery.makeArray(elem.childNodes);}
	}, function(name, fn){
		jQuery.fn[ name ] = function( selector ) {
			var ret = jQuery.map( this, fn );

			if ( selector && typeof selector == "string" )
				ret = jQuery.multiFilter( selector, ret );

			return this.pushStack( jQuery.unique( ret ) );
		};
	});
```

通过调用jQuery.dir静态函数:

### jQuery.dir:
```javascript
	jQuery.dir(elem,"parentNode");
```
jQuery.dir函数如下，其实和1.0版本中的jQuery.parents大同小异:
```javascript
	dir: function( elem, dir ){
		var matched = [],
			cur = elem[dir];
		while ( cur && cur != document ) {
			if ( cur.nodeType == 1 )
				matched.push( cur );
			cur = cur[dir];
		}
		return matched;
	}
```


下篇我们来分析为什么要做出这种改变？