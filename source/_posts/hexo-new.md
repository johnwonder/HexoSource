title: hexo new命令解析
date: 2016-09-09 22:28:15
tags: hexo new
---

## hexo new命令

hexo new命令是创建新文章的，后面跟文章的名字比如hexo new helloworld,那么就会创建一篇title为helloworld的文章。
```javascript
	 var cmd = '';

    if (!args.h && !args.help) {
      cmd = args._.shift();

      if (cmd) {
        var c = hexo.extend.console.get(cmd);
        if (!c) cmd = 'help';
      } else {
        cmd = 'help';
      }
    } else {
      cmd = 'help';
    }
```
args对象类似`{ _: [ 'n', 'hexo_new' ] }`这种形式，所以args._.shift()得到的结果是n。

`hexo.extend.console.get`是通过定义extend对象，extend对象中定义console对象：
代码在hexo目录index.js文件中：

头部先require入extend文件
```js
	var extend = require('../extend');
```
../extend默认会引入extend目录下的index.js文件：
```js
	'use strict';

	exports.Console = require('./console');
	exports.Deployer = require('./deployer');
	exports.Filter = require('./filter');
	exports.Generator = require('./generator');
	exports.Helper = require('./helper');
	exports.Migrator = require('./migrator');
	exports.Processor = require('./processor');
	exports.Renderer = require('./renderer');
	exports.Tag = require('./tag');
```

`require('./console')`引入同级目录下的console文件：
```js
    ...部分代码省略
	Console.prototype.get = function(name){
		  name = name.toLowerCase();
		  return this.store[this.alias[name]];
	};
	...部分代码省略
```

随后在Hexo构造函数中定义：
```js
	this.extend = {
    console: new extend.Console(),
    deployer: new extend.Deployer(),
    filter: new extend.Filter(),
    generator: new extend.Generator(),
    helper: new extend.Helper(),
    migrator: new extend.Migrator(),
    processor: new extend.Processor(),
    renderer: new extend.Renderer(),
    tag: new extend.Tag()
  };
```

所以我们看到了hexo.extend.console.get的流程，既然是获取，那么肯定有注册的过程


注册时在`hexo.init()`的时候注册的

```js
   ...hexo_cli/lib/hexo.js中 找到hexo模块后调用此方法,部分代码省略
	function(mod) {
    if (mod) hexo = mod;
    log = hexo.log;

    require('./console')(hexo);

    return hexo.init();
  }
```
```js
	Hexo.prototype.init = function(){
	  var self = this;

	  this.log.debug('Hexo version: %s', chalk.magenta(this.version));
	  this.log.debug('Working directory: %s', chalk.magenta(tildify(this.base_dir)));

	  // Load internal plugins
	  //加载内部插件
	  require('../plugins/console')(this);
	}
```

 plugins/console/index.js部分代码如下：
```js
	//先获取在构造函数中定义的console对象
	 var console = ctx.extend.console;

	 ...部分代码省略
	 console.register('new', 'Create a new post.', {
	    usage: '[layout] <title>',
	    arguments: [
	      {name: 'layout', desc: 'Post layout. Use post, page, draft or whatever you want.'},
	      {name: 'title', desc: 'Post title. Wrap it with quotations to escape.'}
	    ],
	    options: [
	      {name: '-r, --replace', desc: 'Replace the current post if existed.'},
	      {name: '-s, --slug', desc: 'Post slug. Customize the URL of the post.'},
	      {name: '-p, --path', desc: 'Post path. Customize the path of the post.'}
	    ]
	  }, require('./new'));
```

```js
	Console.prototype.register = function(name, desc, options, fn){
		  if (!name) throw new TypeError('name is required');

		  if (!fn){
		    if (options){
		      if (typeof options === 'function'){
		        fn = options;

		        if (typeof desc === 'object'){ // name, options, fn
		          options = desc;
		          desc = '';
		        } else { // name, desc, fn
		          options = {};
		        }
		      } else {
		        throw new TypeError('fn must be a function');
		      }
		    } else {
		      // name, fn
		      if (typeof desc === 'function'){
		        fn = desc;
		        options = {};
		        desc = '';
		      } else {
		        throw new TypeError('fn must be a function');
		      }
		    }
		  }

		  if (fn.length > 1){
		    fn = Promise.promisify(fn);
		  } else {
		    fn = Promise.method(fn);
		  }

		  var c = this.store[name.toLowerCase()] = fn;
		  c.options = options;
		  c.desc = desc;
		  //new 变成n ne
		  this.alias = abbrev(Object.keys(this.store));
	}
```

我们看看执行new的函数：
```javascript
	function newConsole(args){
		  /* jshint validthis: true */
		  // Display help message if user didn't input any arguments
		  if (!args._.length){
		    return this.call('help', {_: ['new']});
		    //控制台打出类似`Usage: hexo new [layout] <title>`
		  }

		  var data = {
		    title: args._.pop(),
		    layout: args._.length ? args._[0] : this.config.default_layout,
		    slug: args.s || args.slug,
		    path: args.p || args.path
		  };

		  var keys = Object.keys(args);
		  var key = '';
		  var self = this;

		  for (var i = 0, len = keys.length; i < len; i++){
		    key = keys[i];
		    if (!reservedKeys[key]) data[key] = args[key];
		  }

		  return this.post.create(data, args.r || args.replace).then(function(post){
		    self.log.info('Created: %s', chalk.magenta(tildify(post.path)));
		  });
	}
```
