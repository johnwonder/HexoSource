title: javascript数组去重
date: 2017-08-24 11:22:34
tags: javascript
---

看到[面试分享：一年经验初探阿里巴巴前端社招 ](https://github.com/jawil/blog/issues/22)这篇文章，学习到了很多，也发现了自己其实有很多不知道，所以记录下。。

## 数组去重

博主提到了这些方法：
```js
///ES6实现：
[...new Set([1,2,3,1,'a',1,'a'])]

///ES5实现：
[1,2,3,1,'a',1,'a'].filter(function(ele,index,array){
    return index===array.indexOf(ele)
})
```

然后有人回复说用Map,我查了下，应该是[这种方式](http://blog.csdn.net/earscher/article/details/52803988)：

```js
Array.prototype.unique = function () {
        var arr = [];
        var map = new Map();
        for(var i = 0; i < this.length; i++){
            if(!map.has(this[i])){
                arr.push(this[i]);
                map.set(this[i],true)
            }
        }
        return arr;
    };

    Array.prototype.unique2 = function () {
        var arr = [];
        var set = new Set(this);
        set.forEach(function (item) {
            arr.push(item);
        });
        return arr;
    };
```

## 二分查找时间复杂度分析

博主在二分查找的时间复杂度怎么求，是多少 问题上 没回答上来，我看的时候也忘记了，遂查了下：

二分查找的基本思想是将n个元素分成大致相等的两部分，去a[n/2]与x做比较，如果x=a[n/2],则找到x,算法中止；如果x<a[n/2],则只要在数组a的左半部分继续搜索x,如果x>a[n/2],则只要在数组a的右半部搜索x.

时间复杂度无非就是while循环的次数！

总共有n个元素，

渐渐跟下去就是n,n/2,n/4,....n/2^k，其中k就是循环的次数

由于你n/2^k取整后>=1

即令n/2^k=1

可得k=log2n,（是以2为底，n的对数）

参考资料：
[二分查找时间复杂度的计算(转)](http://blog.csdn.net/frances_han/article/details/6458067)
[二分查找时间复杂度分析](http://www.cnblogs.com/qiaozhoulin/p/5328090.html)
[理解O(log2N)和O(Nlog2N)](http://blog.csdn.net/sosesoa/article/details/52775202)
