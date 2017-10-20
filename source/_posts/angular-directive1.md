title: angular_directive1
date: 2017-04-24 16:07:56
tags: angular
---

## angular 指令多个元素multiElement

```js
// iterate over the attributes
       //遍历所有属性 比如ng-app ng-controller
       for (var attr, name, nName, ngAttrName, value, isNgAttr, nAttrs = node.attributes,
                j = 0, jj = nAttrs && nAttrs.length; j < jj; j++) {
         var attrStartName = false;
         var attrEndName = false;

         attr = nAttrs[j];//
         name = attr.name;
         value = trim(attr.value);

         // support ngAttr attribute binding
         //转换成ngController 这种
         ngAttrName = directiveNormalize(name);
         //ng-attr
         if (isNgAttr = NG_ATTR_BINDING.test(ngAttrName)) {
           //ng-attr-
           name = name.replace(PREFIX_REGEXP, '')
             .substr(8).replace(/_(.)/g, function(match, letter) {
               return letter.toUpperCase();
             });
         }

         var multiElementMatch = ngAttrName.match(MULTI_ELEMENT_DIR_RE);
         //判断是否支持multiElement
         if (multiElementMatch && directiveIsMultiElement(multiElementMatch[1])) {
           attrStartName = name;
           attrEndName = name.substr(0, name.length - 5) + 'end';
           name = name.substr(0, name.length - 6); //去掉-start
         }

         nName = directiveNormalize(name.toLowerCase());
         attrsMap[nName] = name;
         if (isNgAttr || !attrs.hasOwnProperty(nName)) {
             attrs[nName] = value;
             if (getBooleanAttrName(node, nName)) {
               attrs[nName] = true; // presence means true
             }
         }
         addAttrInterpolateDirective(node, directives, value, nName, isNgAttr);
         addDirective(directives, nName, 'A', maxPriority, ignoreDirective, attrStartName,
                       attrEndName);
       }
```

注意到这一段代码
```js
var multiElementMatch = ngAttrName.match(MULTI_ELEMENT_DIR_RE);
//判断是否支持multiElement
if (multiElementMatch && directiveIsMultiElement(multiElementMatch[1])) {
  attrStartName = name;
  attrEndName = name.substr(0, name.length - 5) + 'end';
  name = name.substr(0, name.length - 6); //去掉-start
}
```

就是判断有无ng-show-start这种指令的

参考资料：
[AngularJS multi-element directive](http://stackoverflow.com/questions/25254084/angularjs-multi-element-directive)
[ngAttr with Angular for conditional attribute](http://stackoverflow.com/questions/30301554/ngattr-with-angular-for-conditional-attribute)
[angularJs关于指令的一些冷门属性](http://www.cnblogs.com/HeJason/p/5514690.html)
