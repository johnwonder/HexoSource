title: typescript声明
date: 2017-07-20 16:00:40
tags: typescript
---

今天在看[typescript 导入模块](https://mp.weixin.qq.com/s/LVoa3YnDaldiZroya_EN1A)的时候，看到这么一个导入法：

`import * as express from 'express' `

文中提到了如下的问题

最后一行export = express，并且上面分别定义了一个 function express() 和namespace express，这种写法是比较特殊的，我一时也没法解释清楚，反正多参照 DefinitelyTyped 上其他模块的写法即可。这个问题归根结底是 express 模块通过 import * as express from 'express' 引入的时候，express本身又是一个函数，这种写法在早期的 Node.js 版本的程序和 NPM 模块中是比较流行的。

这个应该只和声明有关

然后就看到[这篇文章](https://segmentfault.com/q/1010000008316030),里面提到了用 tsc 编译成 commonjs

```js
"use strict";
var moment = require('./moment');
console.log(typeof moment)
```

然后我自己试了一下在target为es5的情况下确实是编译成了如下js:
```js
var express = require("express");
var app = express();
app.get('/', function (req, res) {
    res.end('hello, world');
});
app.listen(3000, function () {
    console.log('server is listening');
});
```

文中还提到了如何编写声明文件，然后我看了下express的声明文件：

确实是像如下

```
declare function e(): core.Express;

declare namespace e {
     var static: typeof serveStatic;

     export function Router(options?: RouterOptions): core.Router;

     //内部接口 外部获取不到
     interface RouterOptions {

     }
}
export = e;
```

发现express可以定义Router，和内部的RouterOptions
参考资料[Express 4.x API 中文手册](http://www.expressjs.com.cn/4x/api.html#router)
