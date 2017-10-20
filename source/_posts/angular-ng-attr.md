title: angular之ng-attr属性
date: 2017-09-01 10:49:55
tags: angular
---

## ngAttr 绑定任意属性

官网上说是为了可以绕开浏览器的约束：

Web browsers are sometimes picky about what values they consider valid for attributes.
For example, considering this template:

```html
<svg>
  <circle cx="{{cx}}"></circle>
</svg>
```

可以用ng-attr-cx来绕开浏览器的限制

当用了ngAttr属性后，$interpolate服务的allOrNothing标记被用到了，任何表达式里的值是undefined的时候，

属性会被移除且不包含到元素上。

下面的属性也会出现类似问题，select的size 属性，textarea的placeholder属性，button的type属性，

progress的value属性。
