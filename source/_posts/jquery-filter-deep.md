title: jquery_filter函数深入分析
date: 2016-09-02 22:20:19
tags: jquery filter grep
---

##   jquery1.0源码解读

昨天分析each函数时发现最后调用了jQuery.filter('.example',r)函数,我们再深入filter函数看看：

```javascript
	var g = not !== false ? jQuery.grep :
			function(a,f) {return jQuery.grep(a,f,true);};
```
not参数为空，所以g为jQuery.grep。

t为.example，所以满足以下正则:
```javascript
	t && /^[a-z[({<*:.#]/i.test(t) 
```
进入while循环,调用`jQuery.parse`解析数组,满足第四个正则元素`([:.#]*)S`,最后m为
0: ".example"
1: "."
2: "example"
再进入jQuery.expr,因为m[1]为.,所以获得的expr为`jQuery.className.has(a,m[2])`
所以其实到最后就是再调用jQuery.grep函数通过jQuery.className.has筛选元素.
