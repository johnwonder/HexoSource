title: bluebird的promiseall浅析
date: 2016-09-14 23:14:18
tags: [bluebird,hexo]
---

## Promise.all

hexo中load方法使用了Promise.all,来看代码：

```js
	Hexo.prototype.load = function(callback){
	  var self = this;

	  return loadDatabase(this).then(function(){
	    return Promise.all([
	      self.source.process(),
	      self.theme.process()
	    ]);
	  }).then(function(){
	    return self._generate({cache: true});
	    //只有当上面两个方法都执行成功后才会调用_
	  }).nodeify(callback);
	};
```

```js
	var Promise = require('bluebird');

	var call = function(name, args, callback){
	  if (!callback && typeof args === 'function'){
	    callback = args;
	    args = {};
	  }

	  return Promise.all([
	      (function(){
	         console.log('1');//这里会打印
	         //var i = '5';
	      })(),
	      (function(){
	         //console.log('2');//这里不会打印

	         throw '2';
	      })(),
	      (function(){
	         console.log('6');//这里不会打印
	      })()
	    ]).then(function(){//数组里的都通过才会打印3
	    console.log('3');
	  }).nodeify(callback);
	};

	 call('new', '2',function(e){
	  console.log('nodeify');
	 	console.log(e);
	 	console.log('callback'); //进入reject会调用callback

	  //throw ;
	 }).then(function() {
	      	console.log('then');
	    }).catch(function(err) {
	      console.log('end');
	      console.log(err);//进入reject会调用
	 });
```
