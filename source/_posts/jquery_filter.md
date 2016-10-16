title: jquery之filter函数解析
date: 2016-08-27 08:46:49
tags: jquery filter
---

##   jquery1.0源码解读

### filter函数

1.0源码:
{% codeblock lang:javascript %}
	filter: function(t) {

		//返回pushStack的结果
		//这种 与加或 的方法值得学习
		return this.pushStack(
			//如果参数是数组 那么就调用map函数
			//返回新的结果
			t.constructor == Array &&
			jQuery.map(this,function(a){
				for ( var i = 0; i < t.length; i++ )
					if ( jQuery.filter(t[i],[a]).r.length )
						return a;
			}) ||
			//如果参数是true那么就直接get()获取
			//如果参数是false那么就返回空数组
			t.constructor == Boolean &&
			( t ? this.get() : [] ) ||
			//如果参数是函数那么就调用grep函数
			t.constructor == Function &&
			jQuery.grep( this, t ) ||
			//如果参数是其他类型那么就调用静态的filter函数
			jQuery.filter(t,this).r, arguments );
	}
{% endcodeblock %}
## 根据函数类型判断再内部调用不同的函数：
### 1.grep函数内部就是遍历元素数组，调用参数函数t，如果满足t函数，那么就push到result数组。
{% codeblock lang:javascript %}
	grep: function(elems, fn, inv) {
		// If a string is passed in for the function, make a function
		// for it (a handy shortcut)
		if ( fn.constructor == String )
			fn = new Function("a","i","return " + fn);
		//定义局部数组	
		var result = [];
		
		// Go through the array, only saving the items
		// that pass the validator function
		for ( var i = 0; i < elems.length; i++ )
			if ( !inv && fn(elems[i],i) || inv && !fn(elems[i],i) )
				result.push( elems[i] );
		
		return result;
	}
{% endcodeblock %}

比如 
{% codeblock lang:javascript %}
	var elems = $('li');

	elems.filter(function(index) {
		//index就是elems的一个元素
  		return $("strong",index).length == 1;//
	}).css('background-color', 'red');
{% endcodeblock %}

jQuery的静态方法是通过extent扩展方法生成的：
{% codeblock lang:javascript %}
	function jQuery(){

	}

	jQuery.extend = function(obj,prop) {
		//巧妙的运用了obj =this;转换了角色
		//obj就变成jQuery
		//prop就变成了obj
		if ( !prop ) { prop = obj; obj = this; }
		for ( var i in prop ) obj[i] = prop[i];
		return obj;
	};

	//jQuery静态方法就可以通过如下方法引入
	jQuery.extend({

	})
{% endcodeblock %}