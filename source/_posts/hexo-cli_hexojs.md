title: hexo-cli源码分析之入口文件hexo.js
date: 2016-08-28 10:40:21
tags: hexo
---

##   hexo-cli 1.0.2源码解读

### hexo入口函数

package.json中定义了
```js
  "bin": {
    "hexo": "./bin/hexo"
  }
```
所以到bin目录下的hexo
```
#!/usr/bin/env node

'use strict';

require('../lib/hexo')();
```

再到lib目录下的hexo.js，发现定义了module.exports= entry,那么找到entry方法

在没有找到hexo模块的情况下什么命令都不起作用
```js
	function entry(cwd, args) {
	  cwd = cwd || process.cwd();
	  //如果没有输入cwd那么就获取运行当前脚本的工作目录的路径
	  //ChangeWorkingDirectory cwd
	  //比如 hexo n 那么就是{ _: [ 'n' ] }
	  //hexo n --cwd e:\ss 就是{ _: [ 'help' ], cwd: 'e:\\ss' }
	  args = camelCaseKeys(args || minimist(process.argv.slice(2)));

	  //Context中注册了EventEmitter
	  var hexo = new Context(cwd, args);
	  var log = hexo.log;

	  // Change the title in console
	  process.title = 'hexo';

	  function handleError(err) {
	    log.fatal(err);
	    process.exit(2);
	  }
	  //findPkg其实就是检查package.json中是否有hexo
	  //并加载node_modules中的hexo 模块
	  return findPkg(cwd, args).then(function(path) {
	    if (!path) return;

	    hexo.base_dir = path;
	    //加载node_modules中的hexo模块
	    return loadModule(path, args).catch(function() {
	      log.error('Local hexo not found in %s', chalk.magenta(tildify(path)));
	      log.error('Try running: \'npm install hexo --save\'');
	      process.exit(2);
	    });
	  }).then(function(mod) {
	    if (mod) hexo = mod;
	    log = hexo.log;

	    require('./console')(hexo);

	    return hexo.init();
	  }).then(function() {
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

	    watchSignal(hexo);

	    return hexo.call(cmd, args).then(function() {
	      return hexo.exit();
	    }).catch(function(err) {
	      return hexo.exit(err).then(function() {
	        handleError(err);
	      });
	    });
	  }).catch(handleError);
	}
```

[process](http://www.css88.com/archives/4548)
[NodeJs中的EventEmitter](http://biyeah.iteye.com/blog/1308954)
