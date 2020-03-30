title: angularjs1.5.8源码解析之value服务
date: 2020-03-29 09:20:04
tags: angular
---

angularjs中模块的value()方法可以对模块数据模型进行扩展,使得不同controller可以进行通信,
充当全局变量的作用。使用示例：

```js
  var app=angular.module('app',[]);
	app.value('val',{});

	app.controller('con',['$scope','val',function($scope,ssr){
		$scope.msg='';
		$scope.submit=function(temp)
		{
			//向全局变量存入msg
			ssr.save=temp;
		}

	}])

	app.controller('con2',['$scope','val',function($scope,ssr){
			$scope.showMsg='';
			$scope.download=function()
			{
        //获取全局变量
				$scope.showMsg=val.save;
			}

	}])
```

跟我们之前分析过的decorator一样，value方法内部实际上也是调用$provide.value函数，我们先看看
moduleInstance的value方法:

```js
  var moduleInstance = {
      value: invokeLater('$provide', 'value'),
  }

  function invokeLater(provider, method, insertMethod, queue) {
        if (!queue) queue = invokeQueue;
        return function() {
          queue[insertMethod || 'push']([provider, method, arguments]);
          return moduleInstance;
        };
   }
```

可以清楚的看到，调用```app.value('val',{});```后会把参数放入invokeQueue中,然后在调用
loadModules的时候再调用$provide.value方法

```js
  //用个valueFn包装下，返回内部带有val的方法valueRef
  function value(name, val) { return factory(name, valueFn(val), false); }

  function valueFn(value) {return function valueRef() {return value;};}
```

factory方法我们之前分析过，内部会把```valueFn(val)```放入providerCache,那么之后我们调用```instanceInjector.invoke(provider.$get, provider, undefined, serviceName);```的时候实际上就会调用```valueRef```函数把存放的对象取出来。

我们有时候会调用```$rootElement```这个对象，那找遍了angular内部并没有显示的声明$rootElementProvider方法，那么它到底存储在哪呢？
它就是在angular启动的时候调用```doBootstrap```的时候压入modules数组顶部然后调用的$provide.value把找到的ng-app元素用jqLite包装成jqLite对象，
然后放入全局变量$rootElement中，代码如下：

```js
  //传入的modules 比如html中定义的ng-app="myApp"
  //那么为["myApp"]
  //正常的话 这时 modules已经初始化
  modules = modules || [];
  //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
  modules.unshift(['$provide', function($provide) {
   //调用$provide方法
   //设置$rootElement
   $provide.value('$rootElement', element);
  }]);
```

下篇我们分析下angular的启动函数doBootStrap或者jqLite对象中的扩展方法或者一些常用函数。
