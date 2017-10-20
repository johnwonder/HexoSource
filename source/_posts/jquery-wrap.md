title: jquery_wrap
date: 2016-12-30 21:47:01
tags: jquery
---

## jquery1.0 wrap函数分析

先上代码：

```js
    wrap: function() {
      // The elements to wrap the target around
      //包装目标元素的元素
      var a = jQuery.clean(arguments);

      // Wrap each of the matched elements individually
      return this.each(function(){
        // Clone the structure that we're using to wrap
        var b = a[0].cloneNode(true);//选择a[0]元素

        // Insert it before the element to be wrapped
              //在当前元素之前插入包装节点
        this.parentNode.insertBefore( b, this );

        // Find he deepest point in the wrap structure
        //找出最深的第一个包装的元素
        while ( b.firstChild )
          b = b.firstChild;

        // Move the matched element to within the wrap structure
        //把匹配的当前元素移动到包装元素内
        b.appendChild( this );
      });
    }
```

从中我们可以学到以下几点：

> dom元素的操作

>jquery.clean方法的应用

关于jquery.clean方法我们已经在之前[这篇博客](http://johnwonder.github.io/2016/09/05/jquery-clean/)中分析过
