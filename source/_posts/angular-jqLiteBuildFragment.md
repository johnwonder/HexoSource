title: angular1.5.8源码解析之jqLiteBuildFragment函数
date: 2020-03-26 21:26:01
tags: angular
---

这次我们继续分析上篇中提到的jqLiteBuildFragment函数，顾名思义，它主要的作用就是根据我们传入的html字符串创建相应的html片段。

## jqLiteBuildFragment
```js
function jqLiteBuildFragment(html, context) {
   var tmp, tag, wrap,
       fragment = context.createDocumentFragment(),
       nodes = [], i;

   if (jqLiteIsTextNode(html)) {
     // Convert non-html into a text node
     //如果是文本那就创建文本
     nodes.push(context.createTextNode(html));
   } else {
     // Convert html into DOM nodes
     tmp = fragment.appendChild(context.createElement("div"));
     tag = (TAG_NAME_REGEXP.exec(html) || ["", ""])[1].toLowerCase();
     wrap = wrapMap[tag] || wrapMap._default;

     //不是属于xhtml标记的 像<div/> 的就变成<div></div>
     tmp.innerHTML = wrap[1] + html.replace(XHTML_TAG_REGEXP, "<$1></$2>") + wrap[2];

     // Descend through wrappers to the right content
     //下降包装器到正确的内容
     i = wrap[0];
     while (i--) {
       tmp = tmp.lastChild;
     }
     //
     nodes = concat(nodes, tmp.childNodes);

     tmp = fragment.firstChild;
     tmp.textContent = "";
   }
   // Remove wrapper from fragment
   //移除包装器
   fragment.textContent = "";
   fragment.innerHTML = ""; // Clear inner HTML
   //遍历nodes
   forEach(nodes, function(node) {
     fragment.appendChild(node);
   });

   return fragment;
 }
```

首先angularjs是如何判断它是文本节点呢？

## jqLiteIsTextNode

```js
      var HTML_REGEXP = /<|&#?\w+;/;
      function jqLiteIsTextNode(html) {
       return !HTML_REGEXP.test(html);
     }
```

通过一个HTML_REGEXP正则表达式，就判断是不是文本节点了，正则意思就是没有<或者且不带&#或&标记的就是文本标记了,&ss;和&#ss;都属于html

接下来如果获取标记名称呢？通过一个TAG_NAME_REGEXP正则
```js
  var TAG_NAME_REGEXP = /<([\w:-]+)/;
```

意思就是带有如<div这样的标记就取出div来。然后通过预先定义好的包装映射集合获取到包装器

```js
  var wrapMap = {
   'option': [1, '<select multiple="multiple">', '</select>'],

   'thead': [1, '<table>', '</table>'],
   'col': [2, '<table><colgroup>', '</colgroup></table>'],
   'tr': [2, '<table><tbody>', '</tbody></table>'],
   'td': [3, '<table><tbody><tr>', '</tr></tbody></table>'],
   '_default': [0, "", ""]
  };
```

这个wrap其实都是应该有父节点的html元素,接下来就个重要的XHTML_TAG_REGEXP正则，可以把我们的
xhtml标记给替换掉
```js
  var XHTML_TAG_REGEXP = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:-]+)[^>]*)\/>/gi;

  var _rh = html.replace(XHTML_TAG_REGEXP, "<$1></$2>");
```
可以把类似<td><div/></td> 替换为<td><div></div></td>这个样子。

最后就是通过包装器预先设置好的层级获取到最后我们传入的html节点，然后遍历节点列表，append到fragment上

这里也巧妙的封装了contact和slice函数

```js
  var slice = [].slice; //数组的slice函数
  function concat(array1, array2, index) {
  return array1.concat(slice.call(array2, index));
  }
```

我们从这里可以学到平时在js里怎么创建自己的html片段了。下篇我们来分析下jqLite的扩展函数。
