title: hexo-cli源码分析之控制台注册
date: 2016-08-29 23:33:11
tags: hexo
---

##   hexo-cli 1.0.2版本

### loadModule

hexo.js中定义了context.js，用来实例化hexo对象。
```
 	return findPkg(cwd, args).then(function(path) {
    	if (!path) return;

    	hexo.base_dir = path;

	    return loadModule(path, args).catch(function() {
	      log.error('Local hexo not found in %s', chalk.magenta(tildify(path)));
	      log.error('Try running: \'npm install hexo --save\'');
	      process.exit(2);
	    });
	}).then(function(mod) {
		//加载完hexo模块把实例赋给hexo
	    if (mod) hexo = mod;
	    log = hexo.log;
	    //注册help init version
	    require('./console')(hexo);

	    return hexo.init();
	  })
```

### hexo.init()
```
	Hexo.prototype.init = function(){
	  var self = this;

	  this.log.debug('Hexo version: %s', chalk.magenta(this.version));
	  this.log.debug('Working directory: %s', chalk.magenta(tildify(this.base_dir)));

	  // Load internal plugins
	  //注册内部命令 如deploy new ...
	  require('../plugins/console')(this);
	  require('../plugins/filter')(this);
	  require('../plugins/generator')(this);
	  require('../plugins/helper')(this);
	  require('../plugins/processor')(this);
	  require('../plugins/renderer')(this);
	  require('../plugins/tag')(this);

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
	};
```

`require('../plugins/console')(this);` 会加载plugins/console目录下的index.js文件,
```js
	module.exports = function(ctx){
	  var console = ctx.extend.console;

	  console.register('clean', 'Removed generated files and cache.', require('./clean'));

	  console.register('config', 'Get or set configurations.', {
	    usage: '[name] [value]',
	    arguments: [
	      {name: 'name', desc: 'Setting name. Leave it blank if you want to show all configurations.'},
	      {name: 'value', desc: 'New value of a setting. Leave it blank if you just want to show a single configuration.'}
	    ]
	  }, require('./config'));

	  console.register('deploy', 'Deploy your website.', {
	    options: [
	      {name: '--setup', desc: 'Setup without deployment'},
	      {name: '-g, --generate', desc: 'Generate before deployment'}
	    ]
	  }, require('./deploy'));
	}
```

首先获取`ctx.extend.console`，其实就是extend目录下的index.js文件中定义的

```js
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
