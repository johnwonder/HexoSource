title: hexo_util工具包分析1
date: 2016-09-15 22:57:09
tags: hexo hexo-util
---

## hexo util包分析

### index.js

```js
	'use strict';

	exports.escapeDiacritic = require('./escape_diacritic');
	exports.escapeHTML = require('./escape_html');
	exports.escapeRegExp = require('./escape_regexp');
	exports.highlight = require('./highlight');
	exports.htmlTag = require('./html_tag');
	exports.Pattern = require('./pattern');
	exports.Permalink = require('./permalink');
	exports.slugize = require('./slugize');
	exports.spawn = require('./spawn');
	exports.stripHTML = require('./strip_html');
	exports.truncate = require('./truncate');
	exports.wordWrap = require('./word_wrap');
```

导出了这么多工具类，我们先看slugize，因为关系到post创建的过程

```js
  var ctx = this.context;
  var config = ctx.config;
  //文章标题
  data.slug = slugize((data.slug || data.title).toString(), {transform: config.filename_case});
  data.layout = (data.layout || config.default_layout).toLowerCase();
  data.date = data.date ? moment(data.date) : moment();
```

```js
	'use strict';
	//最重要的就是这两个函数escapeDiacritic和escapeRegExp
	var escapeDiacritic = require('./escape_diacritic');
	var escapeRegExp = require('./escape_regexp');
	var rControl = /[\u0000-\u001f]/g;
	var rSpecial = /[\s~`!@#\$%\^&\*\(\)\-_\+=\[\]\{\}\|\\;:"'<>,\.\?\/]+/g;

	module.exports = function(str, options){
	  if (typeof str !== 'string') throw new TypeError('str must be a string!');
	  options = options || {};

	  var separator = options.separator || '-';
	  var escapedSep = escapeRegExp(separator);// 输出\-

	  var result = escapeDiacritic(str)
	    // Remove control characters
	    .replace(rControl, '')
	    // Replace special characters
	    .replace(rSpecial, separator)
	    // Remove continous separators
	    .replace(new RegExp(escapedSep + '{2,}', 'g'), separator)
	      //n是一个非负整数。至少匹配n次。例如，“o{2,}”不能匹配“Bob”中的“o”，但能匹配“foooood”中的所有o。“o{1,}”等价于“o+”。“o{0,}”则等价于“o*”。
	    // Remove prefixing and trailing separtors
	    .replace(new RegExp('^' + escapedSep + '+|' + escapedSep + '+$', 'g'), '');

	  switch (options.transform){
	    case 1:
	      return result.toLowerCase();

	    case 2:
	      return result.toUpperCase();

	    default:
	      return result;
	  }
	};
```
