title: hexo scaffold的创建过程
date: 2016-09-29 22:08:49
tags: hexo scaffold
---

## layout从哪边来

### new.js

```js
	var data = {
    title: args._.pop(),//标题
    layout: args._.length ? args._[0] : this.config.default_layout,
    slug: args.s || args.slug,
    path: args.p || args.path
  };

  ...省略
   return this.post.create(data, args.r || args.replace).then(function(post){
    self.log.info('Created: %s', chalk.magenta(tildify(post.path)));
  });
```
### post.js
```js
	//默认layout为post
	data.layout = (data.layout || config.default_layout).toLowerCase();

	...省略
	this._getScaffold(data.layout)

	...省略
	Post.prototype._getScaffold = function(layout){
	var ctx = this.context;

	//从scaffold目录中获取post.md
	return ctx.scaffold.get(layout).then(function(result){
	    if (result != null) return result;
	    return ctx.scaffold.get('normal');
	  });
	};
```
### hexo-front-matter/front_matter.js
```js
	 var split = yfm.split(scaffold);

	 ...省略
	 var rPrefixSep = /^(-{3,}|;{3,})/;
	 var rFrontMatter = /^(-{3,}|;{3,})\n([\s\S]+?)\n\1(?:$|\n([\s\S]*)$)/;
	 var rFrontMatterNew = /^([\s\S]+?)\n(-{3,}|;{3,})(?:$|\n([\s\S]*)$)/;

	 ...省略
	 function split(str){
	  if (typeof str !== 'string') throw new TypeError('str is required!');
	  console.log(str);
	  if (rFrontMatter.test(str)) return splitOld(str);

	  //post.md中默认内容 匹配rFrontMatterNew
	  if (rFrontMatterNew.test(str)) return splitNew(str);

	  return {content: str};
	}

	function splitNew(str){
	  console.log('splitNew');
	  if (rPrefixSep.test(str)) return {content: str};

	  var match = str.match(rFrontMatterNew);
	 
	  return {
	    data: match[1],
	    content: match[3] || '',
	    separator: match[2],
	    prefixSeparator: false
	  };
	}
```

```js
	  var frontMatter = prepareFrontMatter(_.clone(data));

	  ...省略
	  function prepareFrontMatter(data){
		  var keys = Object.keys(data);
		  var key = '';
		  var item;

		  for (var i = 0, len = keys.length; i < len; i++){
		    key = keys[i];
		    item = data[key];
			//时间格式化
		    if (moment.isMoment(item)){
		      data[key] = item.utc().format('YYYY-MM-DD HH:mm:ss');
		    } else if (moment.isDate(item)){
		      data[key] = moment.utc(item).format('YYYY-MM-DD HH:mm:ss');
		    } else if (typeof item === 'string'){
		      data[key] = '"' + item + '"';
		    }
		  }

		  return data;
	  }
```

最后调用 swig模板引擎 编译splitedata对象

```js
	 var renderedData = swig.compile(split.data)(frontMatter);

	this.compile = function (source, options) {
	    var key = options ? options.filename : null,
	      cached = key ? cacheGet(key, options) : null,
	      context,
	      contextLength,
	      pre;

	    if (cached) {
	      return cached;
	    }

	    context = getLocals(options);
	    contextLength = utils.keys(context).length;
	    pre = this.precompile(source, options);

	    function compiled(locals) {
	      var lcls;
	      if (locals && contextLength) {
	        lcls = utils.extend({}, context, locals);
	      } else if (locals && !contextLength) {
	        lcls = locals;
	      } else if (!locals && contextLength) {
	        lcls = context;
	      } else {
	        lcls = {};
	      }
	      return pre.tpl(self, lcls, filters, utils, efn);
	    }

	    utils.extend(compiled, pre.tokens);

	    if (key) {
	      cacheSet(key, options, compiled);
	    }

	    return compiled;
  };
```