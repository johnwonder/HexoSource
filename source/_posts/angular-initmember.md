title: angular_源码分析之初始变量
date: 2016-12-30 22:37:20
tags: angular
---

## 变量定义

```js
  //String正则
  //. 正则表示 除\n外的任意字符
  // + 表示 1 到无穷
  var REGEX_STRING_REGEXP = /^\/(.+)\/([a-z]*)$/;

  // The name of a form control's ValidityState property.
  // This is used so that it's possible for internal tests to create mock ValidityStates.
  var VALIDITY_STATE_PROPERTY = 'validity';

  var hasOwnProperty = Object.prototype.hasOwnProperty;

  var lowercase = function(string) {return isString(string) ? string.toLowerCase() : string;};
  var uppercase = function(string) {return isString(string) ? string.toUpperCase() : string;};


  var manualLowercase = function(s) {
  /* jshint bitwise: false */
  //根据charCodeAt来区分

  //charCodeAt() 方法可返回指定位置的字符的 Unicode 编码。这个返回值是 0 - 65535 之间的整数。
  return isString(s)
      ? s.replace(/[A-Z]/g, function(ch) {return String.fromCharCode(ch.charCodeAt(0) | 32);})
      : s;
  };
  var manualUppercase = function(s) {
  /* jshint bitwise: false */

 //位运算符由否定好(~)表示，它是ECMAScript中为数不多的与二进制算术有关的运算符之一。位运算符NOT是三步的处理过程：
 //把运算数转为32位二进制数字
 //再转为它的二进制反码
 //把二进制反码转为浮点数
 //参考 https://my.oschina.net/luozt/blog/302294
 //http://www.cnblogs.com/oneword/archive/2009/12/23/1631039.html
 //10 = 1010
 // 反码 0101
//http://blog.sina.com.cn/s/blog_77abbf390100qwss.html

//负数的二进制表示方法
 //http://www.360doc.com/content/12/0801/17/6828497_227700914.shtml

  return isString(s)
      ? s.replace(/[a-z]/g, function(ch) {return String.fromCharCode(ch.charCodeAt(0) & ~32);})
      : s;
  };

  //Turkish 是啥玩意
  // String#toLowerCase and String#toUpperCase don't produce correct results in browsers with Turkish
  // locale, for this reason we need to detect this case and redefine lowercase/uppercase methods
  // with correct but slower alternatives. See https://github.com/angular/angular.js/issues/11387
  if ('i' !== 'I'.toLowerCase()) {
  lowercase = manualLowercase;
  uppercase = manualUppercase;
  }


  var
    msie,             // holds major version number for IE, or NaN if UA is not IE.
    jqLite,           // delay binding since jQuery could be loaded after us.
    jQuery,           // delay binding
    slice             = [].slice,
    splice            = [].splice,
    push              = [].push,
    toString          = Object.prototype.toString,
    getPrototypeOf    = Object.getPrototypeOf,
    ngMinErr          = minErr('ng'),

    /** @name angular */
    angular           = window.angular || (window.angular = {}),
    angularModule,
    uid               = 0;

  /**
  *documentMode 是IE独有的属性
  * documentMode is an IE-only property
  * http://msdn.microsoft.com/en-us/library/ie/cc196988(v=vs.85).aspx
  */
  msie = window.document.documentMode;
```

### REGEX_STRING_REGEXP用法

REGEX_STRING_REGEXP 在源码22232行使用:

里面涉及到了forEach函数还有ngAttributeAliasDirectives

```js
    //别名属性对象
    var ALIASED_ATTR = {
      'ngMinlength': 'minlength',
      'ngMaxlength': 'maxlength',
      'ngMin': 'min',
      'ngMax': 'max',
      'ngPattern': 'pattern'
    };
    // aliased input attrs are evaluated
    //别名属性被求值
    //遍历
    //定义ngAttributeAliasDirectives
    forEach(ALIASED_ATTR, function(htmlAttr, ngAttr) {
    ngAttributeAliasDirectives[ngAttr] = function() {
      return {
        priority: 100,
        link: function(scope, element, attr) {
          //ngPattern的特殊例子
          //当一个字面意义的正则被用作表达式时
          //这种我们就不必watch任何东西了。
          //比如ng-pattern="/[a-zA-Z]/"
          //special case ngPattern when a literal regular expression value
          //is used as the expression (this way we don't have to watch anything).
          if (ngAttr === "ngPattern" && attr.ngPattern.charAt(0) == "/") {
            var match = attr.ngPattern.match(REGEX_STRING_REGEXP);
            if (match) {
              attr.$set("ngPattern", new RegExp(match[1], match[2]));
              return;
            }
          }

          scope.$watch(attr[ngAttr], function ngAttrAliasWatchAction(value) {
            attr.$set(ngAttr, value);
          });
        }
      };
    };
    });
```

### foreach函数

foreach第一个参数为值，第二个参数为键，源码分析可以看我的另一篇博客
[angular1.5.8 foreach 源码分析](http://johnwonder.github.io/2017/02/21/angular-foreach/)
```js
for (key in obj) {
       if (hasOwnProperty.call(obj, key)) {
         iterator.call(context, obj[key], key, obj);
       }
     }
```

### ngAttributeAliasDirectives

`ngAttributeAliasDirectives`在`publishExternalAPI`函数中使用到了

```js
$provide.provider('$compile', $CompileProvider).
      directive({
          a: htmlAnchorDirective,
          input: inputDirective,
          textarea: inputDirective,
          form: formDirective,
          script: scriptDirective,
          select: selectDirective,
          style: styleDirective,
          option: optionDirective,
          ngBind: ngBindDirective,
          ngBindHtml: ngBindHtmlDirective,
          ngBindTemplate: ngBindTemplateDirective,
          ngClass: ngClassDirective,
          ngClassEven: ngClassEvenDirective,
          ngClassOdd: ngClassOddDirective,
          ngCloak: ngCloakDirective,
          ngController: ngControllerDirective,
          ngForm: ngFormDirective,
          ngHide: ngHideDirective,
          ngIf: ngIfDirective,
          ngInclude: ngIncludeDirective,
          ngInit: ngInitDirective,
          ngNonBindable: ngNonBindableDirective,
          ngPluralize: ngPluralizeDirective,
          ngRepeat: ngRepeatDirective,
          ngShow: ngShowDirective,
          ngStyle: ngStyleDirective,
          ngSwitch: ngSwitchDirective,
          ngSwitchWhen: ngSwitchWhenDirective,
          ngSwitchDefault: ngSwitchDefaultDirective,
          ngOptions: ngOptionsDirective,
          ngTransclude: ngTranscludeDirective,
          ngModel: ngModelDirective,
          ngList: ngListDirective,
          ngChange: ngChangeDirective,
          pattern: patternDirective,
          ngPattern: patternDirective,
          required: requiredDirective,
          ngRequired: requiredDirective,
          minlength: minlengthDirective,
          ngMinlength: minlengthDirective,
          maxlength: maxlengthDirective,
          ngMaxlength: maxlengthDirective,
          ngValue: ngValueDirective,
          ngModelOptions: ngModelOptionsDirective
      }).
      directive({
        ngInclude: ngIncludeFillContentDirective
      }).
      directive(ngAttributeAliasDirectives).
      directive(ngEventDirectives);
```

参考资料:  
[AngularJS使用ngMessages进行表单验证](http://www.jb51.net/article/77049.htm)  
[AngularJS 最常用的八种功能](http://blog.csdn.net/w329300817/article/details/51981955)
