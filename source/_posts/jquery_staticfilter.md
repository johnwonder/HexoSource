title: jquery之静态filter函数解析
date: 2016-08-27 21:46:59
tags: jquery filter
---
##   jquery1.0源码解读

### filter函数

1.0源码:
```js
	expr: {
		"": "m[2]== '*'||a.nodeName.toUpperCase()==m[2].toUpperCase()",
		"#": "a.getAttribute('id')&&a.getAttribute('id')==m[2]",
		":": {
			// Position Checks
			lt: "i<m[3]-0",
			gt: "i>m[3]-0",
			nth: "m[3]-0==i",
			eq: "m[3]-0==i",
			first: "i==0",
			last: "i==r.length-1",
			even: "i%2==0",
			odd: "i%2",

			// Child Checks
			"first-child": "jQuery.sibling(a,0).cur",
			"last-child": "jQuery.sibling(a,0).last",
			"only-child": "jQuery.sibling(a).length==1",

			// Parent Checks
			parent: "a.childNodes.length",
			empty: "!a.childNodes.length",

			// Text Check
			contains: "(a.innerText||a.innerHTML).indexOf(m[3])>=0",

			// Visibility
			visible: "a.type!='hidden'&&jQuery.css(a,'display')!='none'&&jQuery.css(a,'visibility')!='hidden'",
			hidden: "a.type=='hidden'||jQuery.css(a,'display')=='none'||jQuery.css(a,'visibility')=='hidden'",

			// Form elements
			enabled: "!a.disabled",
			disabled: "a.disabled",
			checked: "a.checked",
			selected: "a.selected"
		},
		".": "jQuery.className.has(a,m[2])",
		"@": {
			"=": "z==m[4]",
			"!=": "z!=m[4]",
			"^=": "!z.indexOf(m[4])",
			"$=": "z.substr(z.length - m[4].length,m[4].length)==m[4]",
			"*=": "z.indexOf(m[4])>=0",
			"": "z"
		},
		"[": "jQuery.find(m[2],a).length"
	},
	// The regular expressions that power the parsing engine
	//正则提供解析能力
	parse: [
		// Match: [@value='test'], [@foo]
		[ "\\[ *(@)S *([!*$^=]*) *Q\\]", 1 ],

		// Match: [div], [div p]
		[ "(\\[)Q\\]", 0 ],

		// Match: :contains('foo')  * 匹配前面的子表达式任意次。例如，zo*能匹配“z”，也能匹配“zo”以及“zoo”。
		[ "(:)S\\(Q\\)", 0 ],//(:)([a-z*_-][a-z0-9_-]*)\\(*'?\"?([^'\"]*?)'?\"? *\\)

		// Match: :even, :last-chlid
		[ "([:.#]*)S", 0 ]
	],
	//t 一般为字符串
	//r为 元素集合
	filter: function(t,r,not) {
		// Figure out if we're doing regular, or inverse, filtering
		//not 为undefined的时候 not!==false返回true
		//所以g = jQuery.grep
		//grep静态函数第三个参数为true的话就返回不满足条件的新的元素
		//t为:not(.example)的时候就会递归调用
		//最终调用的是grep函数
		var g = not !== false ? jQuery.grep :
			function(a,f) {return jQuery.grep(a,f,true);};

		//not(.example) 符合正则
		while ( t && /^[a-z[({<*:.#]/i.test(t) ) {

			var p = jQuery.parse;//parse数组

			for ( var i = 0; i < p.length; i++ ) {
				var re = new RegExp( "^" + p[i][0]

					// Look for a string-like sequence
					.replace( 'S', "([a-z*_-][a-z0-9_-]*)" )

					// Look for something (optionally) enclosed with quotes
					.replace( 'Q', " *'?\"?([^'\"]*?)'?\"? *" ), "i" );

				var m = re.exec( t );

				//:not(.example)递归调用到这
				//满足.example的就是[ "([:.#]*)S", 0 ]
				if ( m ) {
					// Re-organize the match
					if ( p[i][1] )//Match: [@value='test'], [@foo]  p[i][1] 为1
 						m = ["", m[1], m[3], m[2], m[4]];

					// Remove what we just matched
					t = t.replace( re, "" );//一般t就为空了,所以最后就跳出while循环
					//.example的话 t就为空了
					break;//这里就退出循环了
				}
			}

			// :not() is a special case that can be optomized by
			// keeping it out of the expression list
			//:not(.example)
			//m[0] = :not(.example)
			//m[1] = :
			//m[2] = not
			//m[3] = .example
			if ( m[1] == ":" && m[2] == "not" )
				r = jQuery.filter(m[3],r,false).r;//递归下 到里面grep就是为包装grep的函数了

			// Otherwise, find the expression to execute
			//m[0] = .example
			//m[1] = .
			//m[2] = example
			else {

				//m[1]为.那么 在expr数组中就返回jQuery.className.has(a,m[2])
				var f = jQuery.expr[m[1]];
				if ( f.constructor != String )
					f = jQuery.expr[m[1]][m[2]];

				// Build a custom macro to enclose it
				eval("f = function(a,i){" +
					( m[1] == "@" ? "z=jQuery.attr(a,m[3]);" : "" ) +
					"return " + f + "}");

				// Execute it against the current filter
				//f 为 function(a,i){
				//	return jQuery.className.has(a,m[2]) //m[2] 此时为'example'
				//}
				//再结合 g = function(a,f) {return jQuery.grep(a,f,true);};
				//相当于jQuery.grep(r,f,true)
				r = g( r, f );
			}
		}

		// Return an array of filtered elements (r)
		// and the modified expression string (t)
		return { r: r, t: t };
	},
	className: {
		add: function(o,c){
			if (jQuery.className.has(o,c)) return;
			o.className += ( o.className ? " " : "" ) + c;
		},
		remove: function(o,c){
			o.className = !c ? "" :
				o.className.replace(
					new RegExp("(^|\\s*\\b[^-])"+c+"($|\\b(?=[^-]))", "g"), "");
		},
		has: function(e,a) {
			if ( e.className != undefined )
				e = e.className;//直接把e.className给e
			//以a开头和结束
			//或者a前面和后面是空格
			return new RegExp("(^|\\s)" + a + "(\\s|$)").test(e);
		}
	},
```
