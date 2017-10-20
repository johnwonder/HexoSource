title: angular-源码分析之isClass函数
date: 2017-07-27 16:05:36
tags: angular
---

## isClass函数

isClass函数顾名思义是判断函数是否是类的
```js
function stringifyFn(fn) {
  // Support: Chrome 50-51 only
  // Creating a new string by adding `' '` at the end, to hack around some bug in Chrome v50/51
  // (See https://github.com/angular/angular.js/issues/14487.)
  // TODO (gkalpak): Remove workaround when Chrome v52 is released
  return Function.prototype.toString.call(fn) + ' ';
}
function isClass(func) {
    // IE 9-11 do not support classes and IE9 leaks with the code below.
    if (msie <= 11) {
      return false;
    }
    // Support: Edge 12-13 only
    // See: https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/6156135/
    /*调用 stringifyFn字符串化后再用正则判断*/
    return typeof func === 'function'
      && /^(?:class\b|constructor\()/.test(stringifyFn(func));
  }
```
### 实验

```js
class Person {
  constructor(name) {
    this.name=name||"Default";
  }
}
console.log(stringifyFn(Person));
console.log(typeof Person === 'function');
console.log(isClass(Person));
```

### 输出
```js
class Person {
  constructor(name) {
    this.name=name||"Default";
  }
  // toString(){
  //   return 'Name:'+this.name;
  // }
}
true
true
```
