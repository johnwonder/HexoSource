title: angular源码分析之reverseParams函数
date: 2017-07-26 11:32:03
tags: angular
---

## reverseParams方法

### 出处

在angular注册指令方法里，如果name是数组，那么会调用`forEach`方法来遍历注册

```js
   forEach(name, reverseParams(registerDirective));
```

### 定义

```js
  /**
  *因为forEach是value,key的顺序，但是key,value是最常用的。
  * when using forEach the params are value, key, but it is often useful to have key, value.
  * @param {function(string, *)} iteratorFn
  * @returns {function(*, string)}
  */
  function reverseParams(iteratorFn) {
  return function(value, key) {iteratorFn(key, value);};
  }
```
