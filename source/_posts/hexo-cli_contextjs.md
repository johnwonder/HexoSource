title: hexo-cli源码分析之context.js
date: 2016-08-28 21:55:04
tags: hexo context.js Context
---

##   hexo-cli 1.0.2源码解读

### hexo入口函数中的Context

hexo.js中定义了context.js，用来实例化hexo对象。

```js
	function Context(base, args) {
	  base = base || process.cwd();
	  args = args || {};

	  EventEmitter.call(this);//用到了nodejs的EventEmitter

	  this.base_dir = base;
	  this.log = logger(args);

	  this.extend = {
	    console: new ConsoleExtend()
	  };
	}

	require('util').inherits(Context, EventEmitter);

	Context.prototype.init = function() {
	  // Do nothing
	};

	Context.prototype.call = function(name, args, callback) {
	  if (!callback && typeof args === 'function') {//如果没有callback且args为函数
	    callback = args;
	    args = {};
	  }

	  var self = this;

	  //用了bluebird的Promise
	  return new Promise(function(resolve, reject) {
	    var c = self.extend.console.get(name);

	    if (c) {
	      c.call(self, args).then(resolve, reject);
	    } else {
	      reject(new Error('Console `' + name + '` has not been registered yet!'));
	    }
	  }).asCallback(callback);
	};

	Context.prototype.exit = function(err) {
	  if (err) {
	    this.log.fatal(
	      {err: err},
	      'Something\'s wrong. Maybe you can find the solution here: %s',
	      chalk.underline('http://hexo.io/docs/troubleshooting.html')
	    );
	  }
	};

	Context.prototype.unwatch = function() {
	  // Do nothing
	};

	module.exports = Context;
```
