title: css3动画探秘1
date: 2017-06-15 14:18:35
tags: CSS
---

## 百度地图动画研究

今天在百度地图上看到了一个动画效果，遂找了下他的源码研究一下：

```css
.poidetail-widget-generalInfo .generalInfo-function-buttons .generalInfo-function-buttons-fav:hover{cursor:pointer}
.poidetail-widget-generalInfo .generalInfo-function-buttons .generalInfo-function-buttons-fav:hover .buttons-fav-icon{-webkit-animation-name:bigsmall;animation-name:bigsmall;-webkit-animation-timing-function:ease-in-out;animation-timing-function:ease-in-out}
@keyframes bigsmall{0%{-webkit-transform:scale(1);transform:scale(1)}30%{-webkit-transform:scale(1.3);transform:scale(1.3)}60%{-webkit-transform:scale(0.7);transform:scale(0.7)}85%{-webkit-transform:scale(1.2);transform:scale(1.2)}100%{-webkit-transform:scale(1);transform:scale(1)}}

```

主要就是定义animation-name,和@keyframes

参考资料：
[CSS3 animation-name 属性](http://www.w3school.com.cn/cssref/pr_animation-name.asp)
[CSS3 @keyframes 规则](http://www.w3school.com.cn/cssref/pr_keyframes.asp)
