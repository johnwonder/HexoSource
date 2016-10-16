title: jquery的filter和grep函数思考
date: 2016-09-03 22:11:04
tags: jquery filter grep
---

##   jquery1.0源码解读

filter函数首先根据参数not确定是不是筛选出不满足参数t的：
1.not为true时筛选出不满足参数t的元素:
  构造`function(a,f) {return jQuery.grep(a,f,true);}`，因为要传递true，所以只能构造。

2.not为false时筛选出满足参数t的元素：调用`jQuery.grep `

然后根据`while ( t && /^[a-z[({<*:.#]/i.test(t) )`来循环筛选元素

1.必须满足`/^[a-z[({<*:.#]/i`正则才会进入`while`循环，否则直接返回`return { r: r, t: t };`

2.进入循环后就开始应用`jQuery.parse`数组循环筛选：
  
  因为jQuery.parse数组中包含S，Q两个正则常量:
  1.//Look for a string-like sequence
    replace( 'S', "([a-z*_-][a-z0-9_-]*)" )
    就是要找类似字符串的
  2.//Look for something (optionally) enclosed with quotes
	.replace( 'Q', " *'?\"?([^'\"]*?)'?\"? *" ), "i" );
	寻找通过引号关闭的
    如果满足parse数组中的某个，那么就跳出for循环，这时候`t = t.replace( re, "" )`，t就把满足正则的替换为空了(ps:一般到这t就为空了。)
  3.如果匹配出的结果中包含not,那么就递归进入filter继续筛选出not的元素。
  4.如果没有not，那么就根据`jQuery.expr`找出符合筛选规则的函数进行eval
    因为要传入字符串，所以用eval
  ```javascript
  	 eval("f = function(a,i){" + 
					( m[1] == "@" ? "z=jQuery.attr(a,m[3]);" : "" ) + 
					"return " + f + "}");
  ```
  最后通过grep函数传入f规则函数 再筛选出剩余的元素。


