title: jquery_attr
date: 2016-12-30 22:07:48
tags: jquery
---

## jquery1.0 attr 函数分析

先上代码：

```js

    attr: function( key, value, type ) {
      // Check to see if we're setting style values
      //检查是否是设置属性
      return key.constructor != String || value != undefined ?
        this.each(function(){
          // See if we're setting a hash of styles
          //key是一个对象
          //例如 { "width":“15px”}
          if ( value == undefined )
            // Set all the styles
            for ( var prop in key )
              jQuery.attr(
                type ? this.style : this,
                prop, key[prop]
              );

          // See if we're setting a single key/value style
          else
            jQuery.attr(
              type ? this.style : this,//这边的this已经是dom对象
              key, value
            );
        }) :

        // Look for the case where we're accessing a style value
        //读取属性值 css ->调用静态方法jQuery.curCSS
        jQuery[ type || "attr" ]( this[0], key );
    }
```

### jquery1.0 静态 attr函数

```js
    attr: function(elem, name, value){
      var fix = {
        "for": "htmlFor",
        "class": "className",
        "float": "cssFloat",
        innerHTML: "innerHTML",
        className: "className"
      };

      if ( fix[name] ) {
        if ( value != undefined ) elem[fix[name]] = value;
        return elem[fix[name]];
      } else if ( elem.getAttribute ) {
        if ( value != undefined ) elem.setAttribute( name, value );
        return elem.getAttribute( name, 2 );
      } else {
        name = name.replace(/-([a-z])/ig,function(z,b){return b.toUpperCase();});
        if ( value != undefined ) elem[name] = value;
        return elem[name];
      }
    }
```

从中我们可以学到如下几点：

> jquery.each方法

>jquery静态方法的调用

>dom是如何给元素设置属性的

>一个函数中既包含设置又包含读取的功能
