title: angularjs源码剖析之minErr函数
date: 2016-12-29 16:09:25
tags: angular
---


## angular1.5.8 minErr函数

在运行过程中如果angularjs脚本出现错误,那么在浏览器控制台我们可以看到自定义的错误信息，
这些信息就是通过这个函数输出的。

### 使用方法
angular.js内部代码有如下用法：  

```js
  ngMinErr  = minErr('ng') //代表是ng模块的错误
```

### 函数解读
```js
  /**
  * @description  描述
  * 这个对象提供一个用于angular创建富错误信息的能力
  * This object provides a utility for producing rich Error messages within
  * Angular. It can be called as follows:
  *
  * 使用方法
  * var exampleMinErr = minErr('example');
  * throw exampleMinErr('one', 'This {0} is {1}', foo, bar);
  *
  * The above creates an instance of minErr in the example namespace. The
  * resulting error will have a namespaced error code of example.one.  The
  * resulting error will replace {0} with the value of foo, and {1} with the
  * value of bar. The object is not restricted in the number of arguments it can
  * take.
  *
  *如果传入比需要少点的参数，那么多出来的参数会保存起来
  * If fewer arguments are specified than necessary for interpolation, the extra
  * interpolation markers will be preserved in the final string.
  *由于数据在生成过程中会被静态解析,在minErr实例创建和调用的过程中一些约束被适当的应用。
  * Since data will be parsed statically during a build step, some restrictions
  * are applied with respect to how minErr instances are created and called.
  * Instances should have names of the form namespaceMinErr for a minErr created
  * using minErr('namespace') .
  * 错误代码，命名空间和模板字符串都应该是静态字符串，不能是变量和生成的表达式
  * Error codes, namespaces and template strings
  * should all be static strings, not variables or general expressions.
  *
  * @param {string} module The namespace to use for the new minErr instance.

  *在angular.js 16314行 定义了一个如下Err 其中就传入了TypeError 对象
  * var $qMinErr = minErr('$q', TypeError);

  * line 16413
  *$qMinErr(
  *        'qcycle',
  *        "Expected promise to be resolved with value other than itself '{0}'",
  *        val)

  * Promise解读
  * 相关链接 http://lib.csdn.net/article/angularjs/33116
  * @param {function} ErrorConstructor Custom error constructor to be instantiated when returning
  *   error from returned function, for cases when a particular type of error is useful.
  * @returns {function(code:string, template:string, ...templateArgs): Error} minErr instance
  */

  function minErr(module, ErrorConstructor) {
      ErrorConstructor = ErrorConstructor || Error;//如果没有提供Error构造函数，就用默认的Error

      //返回真正可以使用的函数
      return function() {
        var SKIP_INDEXES = 2;//跳过的索引

        var templateArgs = arguments,//函数传入的参数
          code = templateArgs[0],//第一个参数：错误代码
          message = '[' + (module ? module + ':' : '') + code + '] ',//初始化错误消息
          template = templateArgs[1],//第二个参数：模板
          paramPrefix, i;

        message += template.replace(/\{\d+\}/g, function(match) {
          //slice {0} 就提取出0
          //slice {1} 就提取出1
          //比如 "Unknown provider: {0}"就提取出{0}
          var index = +match.slice(1, -1),//相当于把{}去掉
            shiftedIndex = index + SKIP_INDEXES;//根据参数顺序要跳过2个

          if (shiftedIndex < templateArgs.length) {
            // shiftedIndex 是 从 SKIP_INDEXES 开始计算的
            // 因为第一个参数为 错误代码 ，第二个参数为 模板
            //相当于格式化参数
            return toDebugString(templateArgs[shiftedIndex]);
          }

          return match;
        });

        message += '\nhttp://errors.angularjs.org/1.5.8/' +
          (module ? module + '/' : '') + code;

        for (i = SKIP_INDEXES, paramPrefix = '?'; i < templateArgs.length; i++, paramPrefix = '&') {
          message += paramPrefix + 'p' + (i - SKIP_INDEXES) + '=' +
            encodeURIComponent(toDebugString(templateArgs[i]));
        }

        return new ErrorConstructor(message);
      };
  }


/* global toDebugString: true */

  function serializeObject(obj) {
    var seen = [];

    //关于JSON.stringify
    //https://developer.mozilla.org/zh-CN/docs/Using_native_JSON
    //该参数可以是多种类型,如果是一个函数,则它可以改变一个javascript对象在字符串化过程中的行为,
    //如果是一个包含 String 和 Number 对象的数组,则它将作为一个白名单.
    //只有那些键存在域该白名单中的键值对才会被包含进最终生成的JSON字符串中.如果该参数值为null或者被省略,则所有的键值对都会被包含进最终生成的JSON字符串中.
    return JSON.stringify(obj, function(key, val) {
      val = toJsonReplacer(key, val);
      if (isObject(val)) {

        if (seen.indexOf(val) >= 0) return '...';//变成省略号

        seen.push(val);
      }
      return val;
    });
  }
  //输出调试信息
  function toDebugString(obj) {
    if (typeof obj === 'function') {
      return obj.toString().replace(/ \{[\s\S]*$/, '');//把函数体去掉
    } else if (isUndefined(obj)) {
      return 'undefined';
    } else if (typeof obj !== 'string') {
      return serializeObject(obj);
    }
    return obj;
  }
```

从这个函数中我们可以学到以下几点

> 函数内部返回函数

> javascript slice的用法

> javascript replace 加正则的用法

> 函数的包装

> 对象的序列化 通过JSON.stringify
