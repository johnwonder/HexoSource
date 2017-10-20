title: angular-assertArg
date: 2017-03-14 12:53:11
tags: angular
---

## angular 1.5.8 源码解析

```js
    /**
    * throw error if the argument is falsy.
    */
    function assertArg(arg, name, reason) {
        //如果arg 判断错误 就抛出ngMinErr
        if (!arg) {
          throw ngMinErr('areq', "Argument '{0}' is {1}", (name || '?'), (reason || "required"));
        }
        return arg;
    }

    //接受数组的annotate
    //比如['$scope',function($scope){}]
    function assertArgFn(arg, name, acceptArrayAnnotation) {
        if (acceptArrayAnnotation && isArray(arg)) {
            arg = arg[arg.length - 1];
        }

        assertArg(isFunction(arg), name, 'not a function, got ' +
            (arg && typeof arg === 'object' ? arg.constructor.name || 'Object' : typeof arg));
        return arg;
    }
```
