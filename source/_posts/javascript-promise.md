title: javascript_promise
date: 2017-06-07 10:32:28
tags: javascript
---

## 前端面试1

### promise

```js
  setTimeout(function() {
  console.log(1)
  }, 0);
  new Promise(function executor(resolve) {
  console.log(2);
  for( var i=0 ; i<10000 ; i++ ) {
    i == 9999 && resolve();
  }
  console.log(3);
  }).then(function() {
  console.log(4);
  });
  console.log(5);
```
首先先碰到一个 setTimeout，于是会先设置一个定时，在定时结束后将传递这个函数放到任务队列里面，因此开始肯定不会输出 1 。

然后是一个 Promise，里面的函数是直接执行的，因此应该直接输出 2 3 。

然后，Promise 的 then 应当会放到当前 tick 的最后，但是还是在当前 tick 中。

因此，应当先输出 5，然后再输出 4 。

最后在到下一个 tick，就是 1 。

“2 3 5 4 1”

参考资料：
[Excuse me？这个前端面试在搞事！](https://zhuanlan.zhihu.com/p/25407758?refer=ElemeFE)
