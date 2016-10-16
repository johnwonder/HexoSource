title: jquery-find
date: 2015-09-25 15:03:05
tags:
---

##   jquery1.0源码解读

### find函数

1.0源码:
{% codeblock lang:javascript %}
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
		} else if ( !t.indexOf("/") ) {
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
					done = jQuery.merge( done, ret );
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
						//参数为div#example，那么会先找到div 
						//调用getElementsByTagName,获取所有div
						for ( var i = 0; i < ret.length; i++ )
							r = jQuery.merge( r,
								m[2] == "*" ?
									jQuery.getAll(ret[i]) :
									ret[i].getElementsByTagName(m[2])
							);
					}
				}
			}
	
			if ( t ) {//获取所有div后再通过调用jQuery静态filter函数
				  //过滤出example元素
				var val = jQuery.filter(t,r);
				ret = r = val.r;
				t = jQuery.trim(val.t);
			}
		}
	
		if ( ret && ret[0] == context ) ret.shift();
		done = jQuery.merge( done, ret );
	
		return done;
	}
{% endcodeblock %}

### filter函数
{% codeblock lang:javascript %}
filter: function(t,r,not) {
		// Figure out if we're doing regular, or inverse, filtering
		//grep查找满足过滤函数的数组元素。原始数组不受影响
		var g = not !== false ? jQuery.grep :
			function(a,f) {return jQuery.grep(a,f,true);};
		
		while ( t && /^[a-z[({<*:.#]/i.test(t) ) {

			var p = jQuery.parse;//解析数组存放 规则

			for ( var i = 0; i < p.length; i++ ) {
				var re = new RegExp( "^" + p[i][0]

					// Look for a string-like sequence
					.replace( 'S', "([a-z*_-][a-z0-9_-]*)" )

					// Look for something (optionally) enclosed with quotes
					.replace( 'Q', " *'?\"?([^'\"]*?)'?\"? *" ), "i" );
				//i 在 RegExp级别下

				var m = re.exec( t );

				if ( m ) {//div#example 匹配[ "([:.#]*)S", 0 ]
					// Re-organize the match
					if ( p[i][1] )//检查parse数组 第二个元素是0还是1 0就跳过
						m = ["", m[1], m[3], m[2], m[4]];

					// Remove what we just matched
					//div#exmaple 把div给替换掉了 接下来再遍历匹配#example
					t = t.replace( re, "" ); 

					break;
				}
			}
	
			// :not() is a special case that can be optomized by
			// keeping it out of the expression list
			if ( m[1] == ":" && m[2] == "not" )
				r = jQuery.filter(m[3],r,false).r;
			
			// Otherwise, find the expression to execute
			else {
				var f = jQuery.expr[m[1]]; //寻找表达式数组匹配元素
				//	"": "m[2]== '*'||a.nodeName.toUpperCase()==m[2].toUpperCase()",
				// "#": "a.getAttribute('id')&&a.getAttribute('id')==m[2]",
				if ( f.constructor != String )
					f = jQuery.expr[m[1]][m[2]];
					
				// Build a custom macro to enclose it
				//制造一个自定义的函数
				eval("f = function(a,i){" + 
					( m[1] == "@" ? "z=jQuery.attr(a,m[3]);" : "" ) + 
					"return " + f + "}");
				
				// Execute it against the current filter
				
				//过滤 通过f 函数
				r = g( r, f );
			}
		}
	
		// Return an array of filtered elements (r)
		// and the modified expression string (t)
		return { r: r, t: t };
	}
{% endcodeblock %}
