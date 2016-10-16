title: hexo 命令call是如何调用的
date: 2016-09-11 20:03:17
tags: hexo call
---

## hexo.call

首先调用hexo对象中的call方法：
```js
	return hexo.call(cmd, args).then(function() {
      return hexo.exit();
    }).catch(function(err) {
      return hexo.exit(err).then(function() {
        handleError(err);
      });
    });
```

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

注意到，通过`self.extend.console.get(name)`获取到的其实是个函数function，所以`c.call(self,args)`其实是调用的js自带的call函数。因为console.register的时候是放入的函数，注意`  var c = this.store[name.toLowerCase()] = fn;`这句代码，关于注册命令的分析，我们已经在(hexo new命令解析)[https://johnwonder.github.io/2016/09/09/hexo-new/]中分析过。

所以这里的call就是调用的每个命令js文件中定义的函数，如plugins/console/new.js中定义的：
```js
	function newConsole(args){
	  /* jshint validthis: true */
	  // Display help message if user didn't input any arguments
	  //console.log('参数是：'+args._);
	  if (!args._.length){
	    return this.call('help', {_: ['new']});
	  }

	  var data = {
	    title: args._.pop(),//标题
	    //这里layout就是可以通过 hexo n  title layout获取
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

这里的this对象就是hexo本身，所以this.post就是hexo对象中定义的post变量：

```js
	Post.prototype.create = function(data, replace, callback){
	  if (!callback && typeof replace === 'function'){
	    callback = replace;
	    replace = false;
	  }

	  var ctx = this.context;
	  var config = ctx.config;

	  data.slug = slugize((data.slug || data.title).toString(), {transform: config.filename_case});
	  data.layout = (data.layout || config.default_layout).toLowerCase();
	  data.date = data.date ? moment(data.date) : moment();

	  return Promise.all([
	    // Get the post path
	    ctx.execFilter('new_post_path', data, {
	      args: [replace],
	      context: ctx
	    }),
	    // Get the scaffold
	    this._getScaffold(data.layout)
	  ]).spread(function(path, scaffold){
	    // Split data part from the raw scaffold
	    var split = yfm.split(scaffold);
	    var separator = split.separator || '---';
	    var jsonMode = separator[0] === ';';
	    var frontMatter = prepareFrontMatter(_.clone(data));
	    var content = '';

	    // Compile front-matter with data
	    var renderedData = swig.compile(split.data)(frontMatter);

	    // Parse front-matter
	    var compiled;

	    if (jsonMode){
	      compiled = JSON.parse('{' + renderedData + '}');
	    } else {
	      compiled = yaml.load(renderedData);
	    }

	    // Add data which are not in the front-matter
	    var keys = Object.keys(data);
	    var key = '';
	    var obj = compiled;

	    for (var i = 0, len = keys.length; i < len; i++){
	      key = keys[i];

	      if (!preservedKeys[key] && obj[key] == null){
	        obj[key] = data[key];
	      }
	    }

	    // Prepend the separator
	    if (split.prefixSeparator) content += separator + '\n';

	    content += yfm.stringify(obj, {
	      mode: jsonMode ? 'json' : ''
	    });

	    // Concat content
	    content += split.content;

	    if (data.content){
	      content += '\n' + data.content;
	    }

	    var result = {
	      path: path,
	      content: content
	    };

	    return Promise.all([
	      // Write content to file
	      fs.writeFile(path, content),
	      // Create asset folder
	      createAssetFolder(path, config.post_asset_folder)
	    ]).then(function(){
	      ctx.emit('new', result);
	    }).thenReturn(result);
	  }).nodeify(callback);
	}
```
