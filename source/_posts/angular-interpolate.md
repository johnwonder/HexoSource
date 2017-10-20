title: angular_interpolate
date: 2017-05-05 22:44:06
tags: angular
---

## angular 插值

### angular interpolate

```js
    // Concatenating(连接) expressions makes it hard to reason about whether some combination of
    // concatenated values are unsafe to use and could easily lead to XSS.  By requiring that a
    // single expression be used for iframe[src], object[src], etc., we ensure that the value
    // that's used is assigned or constructed by some JS code somewhere that is more testable or
    // make it obvious that you bound the value to some user controlled value.  This helps reduce
    // the load when auditing for XSS issues.
    if (trustedContext && concat.length > 1) {
        $interpolateMinErr.throwNoconcat(text);
    }
```

什么时候报错呢？

像ng-src="img/{{imgSource}}"就会报错了

```js
var $interpolateMinErr = angular.$interpolateMinErr = minErr('$interpolate');
//https://segmentfault.com/q/1010000007677889?_ea=1423254
$interpolateMinErr.throwNoconcat = function(text) {
throw $interpolateMinErr('noconcat',
    "Error while interpolating: {0}\nStrict Contextual Escaping disallows " +
    "interpolations that concatenate multiple expressions when a trusted value is " +
    "required.  See http://docs.angularjs.org/api/ng.$sce", text);
};

$interpolateMinErr.interr = function(text, err) {
return $interpolateMinErr('interr', "Can't interpolate: {0}\n{1}", text, err.toString());
};
```

参考资料：
[AugularJS通过服务器请求图片时总是报错 $interpolate:noconcat](https://segmentfault.com/q/1010000007677889?_ea=1423254)
[浅谈AngularJS的$interpolate服务 1](https://segmentfault.com/a/1190000002753321)
