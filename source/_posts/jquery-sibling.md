title: jquery_sibling
date: 2016-09-06 21:37:19
tags: jquery
---

##   jquery1.0源码解读

jquery 实例方法siblings 定义在jQuery.macros对象中,通过jQuery.each定义:

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

```javascript
axis: {

		parent: "a.parentNode",

		ancestors: jQuery.parents,

		parents: jQuery.parents,

		next: "jQuery.sibling(a).next",

		prev: "jQuery.sibling(a).prev",

		siblings: jQuery.sibling,

		children: "a.childNodes"
	}
```

我们看到siblings实际是调用jQuery静态方法sibling:

```javascript
  sibling: function(elem, pos, not) {
		var elems = [];

		var siblings = elem.parentNode.childNodes;
		for ( var i = 0; i < siblings.length; i++ ) {
			if ( not === true && siblings[i] == elem ) continue;

			if ( siblings[i].nodeType == 1 )//表明是元素
				elems.push( siblings[i] );
			if ( siblings[i] == elem )
				elems.n = elems.length - 1;
		}

		return jQuery.extend( elems, {
			last: elems.n == elems.length - 1,
			//0%2等于0
			cur: pos == "even" && elems.n % 2 == 0 || pos == "odd" && elems.n % 2 || elems[pos] == elem,
			prev: elems[elems.n - 1],
			next: elems[elems.n + 1]
		});
	}
```
用法例如：
`$('li.third-item').siblings().css('background-color', 'red');`
