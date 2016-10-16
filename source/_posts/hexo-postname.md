title: hexo_postname
date: 2016-09-18 22:34:11
tags: hexo post hexo-util
---

## new_post_path.js

hexo的文章生成名称经过newPostFilter这个过滤器来生成的：
```js
	ctx.execFilter('new_post_path', data, {
      args: [replace],
      context: ctx
    })
```
```js
	function newPostPathFilter(data, replace){
	  /* jshint validthis: true */
	  data = data || {};//data里带有文章标题

	  var sourceDir = this.source_dir;//this指向hexo实例 
	  //this.source_dir = pathFn.join(base, 'source') + sep; hexo/index.js
	  var draftDir = pathFn.join(sourceDir, '_drafts');
	  var postDir = pathFn.join(sourceDir, '_posts');
	  var config = this.config;
	  var newPostName = config.new_post_name;//默认为:title.md
	  var permalinkDefaults = config.permalink_defaults;
	  var path = data.path;//默认为空
	  var layout = data.layout;//默认为空
	  var slug = data.slug;//文章标题
	  var target = '';

	  if (!permalink || permalink.rule !== newPostName){
	    permalink = new Permalink(newPostName); //固定链接 :title.md
	  }

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
	        var date = moment(data.date || Date.now());//moment库
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
	        //console.log('defaults:'+ _.defaults(filenameData, permalinkDefaults).title);
	        //console.log('postDir:'+postDir);

	        //permalink.stringify 正则替换把permalink中的title替换为filenamedata中的titile，就是文章标题
	        //_.defaults lodash库
	        //Assigns own enumerable properties of source object(s) to the destination object for all destination properties that resolve to undefined. Once a property is set, additional values of the same property are ignored. 

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

### hexo-fs/lib/fs.js
```js
	function ensurePath(path, callback){
	  if (!path) throw new TypeError('path is required!');

	  return exists(path).then(function(exist){
	    if (!exist) return path;

	    return readdirAsync(dirname(path)).then(function(files){
	      return _findUnusedPath(path, files);
	    });
	  }).nodeify(callback);
	}
```

```js
	function exists(path, callback){
	  if (!path) throw new TypeError('path is required!');

	  return new Promise(function(resolve, reject){
	    fs.exists(path, function(exist){
	      resolve(exist);
	      if (typeof callback === 'function') callback(exist);
	    });
	  });
	}
```

最终文件名的秘密就在这里,相同的文件名就自动加上数字

### _findUnusedPath
```js
	function _findUnusedPath(path, files){
	  var extname = pathFn.extname(path);
	  var basename = pathFn.basename(path, extname);
	  var regex = new RegExp('^' + escapeRegExp(basename) + '(?:-(\\d+))?' + escapeRegExp(extname) + '$');
	  var item = '';
	  var num = -1;
	  var match, matchNum;

	  for (var i = 0, len = files.length; i < len; i++){
	    item = files[i];
	    if (!regex.test(item)) continue;

	    match = item.match(regex);
	    matchNum = match[1] ? parseInt(match[1], 10) : 0;

	    if (matchNum > num){
	      num = matchNum;
	    }
	  }

	  return join(dirname(path), basename + '-' + (num + 1) + extname);
	}
```