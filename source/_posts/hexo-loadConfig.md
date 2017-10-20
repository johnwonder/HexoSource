title: hexo源码分析之加载配置文件
date: 2016-09-25 10:01:17
tags: hexo
---

## hexo是如何加载配置文件的

### hexo/index.js

首先在构造函数里初始化配置文件路径
```js
	//_config.yml就是根目录下的
	this.config_path = args.config ? pathFn.resolve(base, args.config)
                                 : pathFn.join(base, '_config.yml');
```

hexo有个默认的配置文件

### default_config.js
```js
	'use strict';

	module.exports = {
	  // Site
	  title: 'Hexo',
	  subtitle: '',
	  description: '',
	  author: 'John Doe',
	  language: '',
	  timezone: '',
	  // URL
	  url: 'http://yoursite.com',
	  root: '/',
	  permalink: ':year/:month/:day/:title/',
	  permalink_defaults: {},
	  // Directory
	  source_dir: 'source',
	  public_dir: 'public',
	  tag_dir: 'tags',
	  archive_dir: 'archives',
	  category_dir: 'categories',
	  code_dir: 'downloads/code',
	  i18n_dir: ':lang',
	  skip_render: [],
	  // Writing
	  new_post_name: ':title.md',
	  default_layout: 'post',
	  titlecase: false,
	  external_link: true,
	  filename_case: 0,
	  render_drafts: false,
	  post_asset_folder: false,
	  relative_link: false,
	  future: true,
	  highlight: {
	    enable: true,
	    auto_detect: true, // Maintain consistent with previous version.
	    line_number: true,
	    tab_replace: '',
	  },
	  // Category & Tag
	  default_category: 'uncategorized',
	  category_map: {},
	  tag_map: {},
	  // Date / Time format
	  date_format: 'YYYY-MM-DD',
	  time_format: 'HH:mm:ss',
	  // Pagination
	  per_page: 10,
	  pagination_dir: 'page',
	  // Extensions
	  theme: 'landscape',
	  // Deployment
	  deploy: {}
	};

```
```js
	  this.config = _.clone(defaultConfig);
```

随后在init函数中初始化配置：

### Hexo.prototype.init
```js
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
```

init函数就是在hexo-cli/hexo.js中加载完hexo模块后就调用的：
```js
	return findPkg(cwd, args).then(function(path) {
    if (!path) return;

    hexo.base_dir = path;

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
  })
```
