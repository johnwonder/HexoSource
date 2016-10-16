title: hexo中的Promise应用
date: 2016-09-11 10:46:55
tags: hexo Promise
---

## hexo Promise的使用

hexo中的Promise用的是`bluebird`框架提供的，到处都是，基本上每个和执行命令相关的都用上了Promise，我们来看相关代码：

###	Promise.each:

位于hexo/index.js：Hexo.prototype.init
```js
		 // Load config
		  return Promise.each([
		    'update_package', // Update package.json
		    'load_config', // Load config
		    'load_plugins' // Load external plugins & scripts
		  ], function(name){
		    return require('./' + name)(self);
		  }).then(function(){
		    return self.execFilter('after_init', null, {context: self});
		  }).then(function(){
		    // Ready to go!
		    self.emit('ready');
		  });
```

遍历参数数组中的每个元素传递给参数方法

### new Promise(func)

```js
		Hexo.prototype.call = function(name, args, callback){
		  if (!callback && typeof args === 'function'){
		    callback = args;
		    args = {};
		  }

		  var self = this;

		  return new Promise(function(resolve, reject){
		    var c = self.extend.console.get(name);

		    if (c){
		      c.call(self, args).then(resolve, reject);
		    } else {
		      reject(new Error('Console `' + name + '` has not been registered yet!'));
		    }
		  }).nodeify(callback);
		};
```

实例化一个Promise，这个call是调用命令控制台命令的函数。

### Promise.all

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
		  }).nodeify(callback);
		};
```

调用所有数组中的方法。
