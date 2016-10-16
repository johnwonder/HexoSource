title: bluebird Promise框架学习一
date: 2016-09-13 22:17:53
tags: blue bird
---

hexo中的Promise用了bluebird提供的，比如Promise.each,Promise.all,new Promise,Promise.map 等等。

## Promise.each

```js
	var Promise = require('bluebird');

	Promise.each([
	    'update_package', // Update package.json
	    'load_config', // Load config
	    'load_plugins' // Load external plugins & scripts
	  ], function(name){
	     if(name == 'load_config')
	     	throw 'error';
	     console.log(name);//会输出update_package,其余不会输出
	  }).catch(function(e){

	  		console.log(e);
	  });
```

```js
	var Promise = require('bluebird');

	var call = function(name, args, callback){
	  if (!callback && typeof args === 'function'){
	    callback = args;
	    args = {};
	  }

	 return new Promise(function(resolve, reject){
	    var c = 1;

	    if (!c){
	       console.log('done');
	    } else {
	    	//进入reject
	      reject(new Error('Console `` has not been registered yet!'));
	    }
	  }).nodeify(callback);
	};

	 call('new', '2',function(e){
	 	console.log(e);
	 	console.log('callback'); //进入reject会调用callback
	 }).then(function() {
	      	console.log('then');
	    }).catch(function(err) {
	      console.log(err);//进入reject会调用
	 });
```
