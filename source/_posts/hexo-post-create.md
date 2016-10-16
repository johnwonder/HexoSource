title: hexo post.create的过程分析
date: 2016-09-11 20:21:31
tags: hexo post 
---

## hexo的post创建过程

要讲hexo的post.create先要看它的传入参数。
```js
	this.post.create(data, args.r || args.replace)
```

data定义如下：
```js
	var data = {
	    title: args._.pop(),//标题
	    //这里layout就是可以通过 hexo n  title layout获取
	    layout: args._.length ? args._[0] : this.config.default_layout,
	    slug: args.s || args.slug,//为undefined
	    path: args.p || args.path//为undefined
	  };
```

hexo/post.js：
```js
	Post.prototype.create = function(data, replace, callback){
	  //callback没传入 replace 为undefined
	  if (!callback && typeof replace === 'function'){
	    callback = replace;
	    replace = false;
	  }

	  var ctx = this.context;//为hexo对象
	  var config = ctx.config; //_.clone(defaultConfig) _:lodash库 浅拷贝
	  //https://github.com/lodash 

	  //slugize为hexo_util库中的函数
	  data.slug = slugize((data.slug || data.title).toString(), {transform: config.filename_case});
	  data.layout = (data.layout || config.default_layout).toLowerCase();
	  data.date = data.date ? moment(data.date) : moment();//获取当前时间 moment库

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
	      fs.writeFile(path, content),//写入内容
	      // Create asset folder
	      createAssetFolder(path, config.post_asset_folder)
	    ]).then(function(){
	    	
	      //http://biyeah.iteye.com/blog/1308954
	      ctx.emit('new', result);
	    }).thenReturn(result);
	  }).nodeify(callback);
	}
```

```js
	ctx.execFilter('new_post_path', data, {
      args: [replace],
      context: ctx
    })
```
执行new_post_path类型的过滤器,hexo/index.js代码如下：
```js
	Hexo.prototype.execFilter = function(type, data, options){
  		return this.extend.filter.exec(type, data, options);
	};
```
路径hexo/extend/filter.js：
```js
	Filter.prototype.exec = function(type, data, options){
		  options = options || {};

		  var filters = this.list(type);
		  var ctx = options.context;//hexo实例对象
		  var args = options.args || []; //上面的[replace]

		  args.unshift(data);

		  return Promise.each(filters, function(filter){
		    return Promise.method(filter).apply(ctx, args).then(function(result){
		      args[0] = result == null ? data : result;
		      return args[0];
		    });
		  }).then(function(){
		    return args[0];
		  });
	};
```

这里我们执行的是new_post_path的filter:
文件路径为hexo/lib/plugins/filter/new_post_path.js
```js
	function newPostPathFilter(data, replace){
	  /* jshint validthis: true */
	  data = data || {};

	  var sourceDir = this.source_dir;
	  var draftDir = pathFn.join(sourceDir, '_drafts');
	  var postDir = pathFn.join(sourceDir, '_posts');
	  var config = this.config;
	  var newPostName = config.new_post_name;
	  var permalinkDefaults = config.permalink_defaults;
	  var path = data.path;
	  var layout = data.layout;
	  var slug = data.slug;
	  var target = '';

	  if (!permalink || permalink.rule !== newPostName){
	    permalink = new Permalink(newPostName);
	  }
	  //path为undefined
	  if (path){
	    switch (layout){
	      case 'page':
	        target = pathFn.join(sourceDir, path);
	        break;

	      case 'draft':
	        target = pathFn.join(draftDir, path);
	        break;

	      default:
	        target = pathFn.join(postDir, path);
	    }
	  } else if (slug){
	    switch (layout){
	      case 'page':
	        target = pathFn.join(sourceDir, slug, 'index');
	        break;

	      case 'draft':
	        target = pathFn.join(draftDir, slug);
	        break;

	      default:
	        var date = moment(data.date || Date.now());
	        var keys = Object.keys(data);
	        var key = '';

	        var filenameData = {
	          year: date.format('YYYY'),
	          month: date.format('MM'),
	          i_month: date.format('M'),
	          day: date.format('DD'),
	          i_day: date.format('D'),
	          title: slug
	        };

	        for (var i = 0, len = keys.length; i < len; i++){
	          key = keys[i];
	          if (!reservedKeys[key]) filenameData[key] = data[key];
	        }

	        target = pathFn.join(postDir, permalink.stringify(
	          _.defaults(filenameData, permalinkDefaults)));
	    }
	  } else {
	    return Promise.reject(new TypeError('Either data.path or data.slug is required!'));
	  }

	  if (!pathFn.extname(target)){
	    target += pathFn.extname(newPostName) || '.md';
	  }

	  if (replace){
	    return Promise.resolve(target);
	  } else {
	    return fs.ensurePath(target);
	  }
	}
```
