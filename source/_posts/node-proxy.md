title: node_proxy
date: 2017-03-26 15:52:33
tags: nodejs
---

## node proxy代理中间件  

今天在研究[node-proxy-middleware](https://github.com/gonzalocasas/node-proxy-middleware)这款代理插件,  

用于在browsersync使用，如下配置：

```js
const conf = require('./gulp.conf');
var url = require('url');
var proxy = require('proxy-middleware');
module.exports = function () {

 var proxyOptions = url.parse('http://localhost:8089/api');
  proxyOptions.route = '/api';

   var proxyOptions1 = url.parse('http://localhost:8089/api1');
  proxyOptions1.route = '/api1';
return {
  server: {
    baseDir: [
      conf.paths.tmp,
      conf.paths.src
    ],
    middleware: [proxy(proxyOptions),proxy(proxyOptions1)]
  },
  open: true
};
};

```

route下的所有请求都会代理到本地8089端口，如果不配置route那么所有请求都会走代理，看代码：

```js
if (typeof options.route === 'string') {
    if (url === options.route) {
      url = '';
    } else if (url.slice(0, options.route.length) === options.route) {
      url = url.slice(options.route.length);
      //一个新的字符串。包括字符串 stringObject 从 start 开始（包括 start）到 end 结束（不包括 end）为止的所有字符。
    } else {
      return next();
    }
  }
```

```js
/*
  如果p1 是 / 结束的
  而且p2是/ 开始的
  那么就把p2开始的斜线去掉
*/
function slashJoin(p1, p2) {
  var trailing_slash = false;

  if (p1.length && p1[p1.length - 1] === '/') { trailing_slash = true; }
  if (trailing_slash && p2.length && p2[0] === '/') {p2 = p2.substring(1); }
 
  return p1 + p2;
}
```
