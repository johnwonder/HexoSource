title: uglifyjs_purefuncs
date: 2017-06-19 11:30:17
tags: javascript
---

今天在看[我推荐的一些前端开发工具](https://segmentfault.com/a/1190000002895976)

这篇文章时，看到‘之后上线时，可以使用uglify压缩掉所有console的代码’ 这段话，

我就琢磨uglify怎么把console去掉，然后就打开[UglifyJS2](https://github.com/mishoo/UglifyJS2)

看文档，有如下描述：

pure_funcs -- default null. You can pass an array of names and UglifyJS will assume that those functions do not produce side effects. DANGER: will not check if the name is redefined in scope. An example case here, for instance var q = Math.floor(a/b). If variable q is not used elsewhere, UglifyJS will drop it, but will still keep the Math.floor(a/b), not knowing what it does. You can pass pure_funcs: [ 'Math.floor' ] to let it know that this function won't produce any side effect, in which case the whole statement would get discarded. The current implementation adds some overhead (compression will be slower).

drop_console -- default false. Pass true to discard calls to console.* functions. If you wish to drop a specific function call such as console.info and/or retain side effects from function arguments after dropping the function call then use pure_funcs instead.

我就写了个例子测试下：

```js
function test(){
	return "ss";
}
console.info(test());
```

运行如下命令```uglifyjs test.js --compress pure_funcs=['console.info'] -o test.min.js ```

会得到如下输出```function test(){return"ss"}test()```

运行如下命令```uglifyjs test.js --compress pure_funcs=['console.info','test'] -o test.min.js ```

会得到如下输出```function test(){return"ss"}```

运行如下命令```uglifyjs test.js --compress drop_console=true -o test.min.js```

会得到如下输出```function test(){return"ss"}```

参考资料：
[UglifyJS中文文档](https://segmentfault.com/a/1190000008995453)
[小tip：我是如何初体验uglifyjs压缩JS的](http://www.zhangxinxu.com/wordpress/2013/01/uglifyjs-compress-js/)
