layout: hexo
title: list_tag
date: 2016-10-01 22:50:29
tags: hexo 
---

## hexo list 命令之tag

hexo list命令是由hexo实例调用的

```js
	//调用hexo类的call方法
	return hexo.call(cmd, args).then(function() {
      return hexo.exit();
    }).catch(function(err) {
      return hexo.exit(err).then(function() {
        handleError(err);
      });
    });

    ...省略
    Hexo.prototype.call = function(name, args, callback){
	  if (!callback && typeof args === 'function'){
	    callback = args;
	    args = {};
	  }

	  var self = this;

	  return new Promise(function(resolve, reject){
	  	//从console类获取存储的命令
	    var c = self.extend.console.get(name);

	    if (c){
	      c.call(self, args).then(resolve, reject);
	    } else {
	      reject(new Error('Console `' + name + '` has not been registered yet!'));
	    }
	  }).nodeify(callback);
	};
```

先调用list命令：
```js
	function listConsole(args){
	  /* jshint validthis: true */
	  var type = args._.shift();
	  var self = this;

	  // Display help message if user didn't input any arguments
	  if (!type || !store.hasOwnProperty(type)){
	    return this.call('help', {_: ['list']});
	  }

	  return this.load().then(function(){
	    return store[type].call(self, args);
	  });
	}
```
加载数据库,统计数量
```js
	Hexo.prototype.load = function(callback){
	  var self = this;

	  return loadDatabase(this).then(function(){
	  	//处理 插入数据库操作
	    return Promise.all([
	      self.source.process(),
	      self.theme.process()
	    ]);
	  }).then(function(){
	    return self._generate({cache: true});
	  }).nodeify(callback);
	};
```

```js
	'use strict';

	var chalk = require('chalk');
	var table = require('text-table');
	var common = require('./common');

	function listTag(){
	  /* jshint validthis: true */
	  //获取hexo实例的model方法
	  var Tag = this.model('Tag');

	  //获取tag数据 从tag model获取
	  var data = Tag.sort({name: 1}).map(function(tag){
	    return [tag.name, String(tag.length), chalk.magenta(tag.path)];
	  });

	  // Table header
	  var header = ['Name', 'Posts', 'Path'].map(function(str){
	    return chalk.underline(str);
	  });

	  //压入表头到data数组
	  data.unshift(header);

	  var t = table(data, {
	    align: ['l', 'r', 'l'],
	    stringLength: common.stringLength
	  });

	  console.log(t);
	  if (data.length === 1) console.log('No tags.');
	}

	module.exports = listTag;
```
