title: lodash之模板设置
date: 2017-08-07 13:23:49
tags: lodash
---

## templateSettings

今天在看lodash的typescript定义文件时，发现有这么一处定义：

```js
  declare namespace _ {
       interface LoDashStatic {

       }

       /**
    * By default, the template delimiters used by Lo-Dash are similar to those in embedded Ruby
    *默认像ruby里的嵌入式
    * (ERB). Change the following template settings to use alternative delimiters.
    **/
    interface TemplateSettings {
        /**
        * The "escape" delimiter.
        **/
        escape?: RegExp;

        /**
        * The "evaluate" delimiter.
        **/
        evaluate?: RegExp;

        /**
        * An object to import into the template as local variables.
        **/
        imports?: Dictionary<any>;

        /**
        * The "interpolate" delimiter.
        **/
        interpolate?: RegExp;

        /**
        * Used to reference the data object in the template text.
        **/
        variable?: string;
    }
  }
```

我们看到lodash的官网上有这么一处举例：

```js
// using custom template delimiters(分隔符)
_.templateSettings.interpolate = /{{([\s\S]+?)}}/g;
var compiled = _.template('hello {{ user }}!');
compiled({ 'user': 'mustache' });
// => 'hello mustache!'
``
