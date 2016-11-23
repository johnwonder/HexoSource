title: hexo流程梳理之database
date: 2016-10-17 22:50:07
tags: warehouse
---

## Hexo List命令

Hexo List命令实际调用的是`plugins/console/list/index.js`文件,核心代码如下：

```js
    var store = {
    page: require('./page'),
    post: require('./post'),
    route: require('./route'),
    tag: require('./tag')
    };

    //外部是调用hexo实例的call函数
    //call函数内部从Console中获取命令函数，然后再用js原生的call函数通过hexo实例调用
    function listConsole(args){
    /* jshint validthis: true */
    var type = args._.shift();//弹出第一个参数
    var self = this;//hexo实例

    // Display help message if user didn't input any arguments
    //如果没有输入参数就调用help命令
    //this指向hexo对象
    //比如输入list tag 那么type就是tag
    if (!type || !store.hasOwnProperty(type)){
      //如果只输入hexo list 或者 输入store对象中没有的参数
      //先调用hexo实例的call方法
      //方法内部调用help的入口函数
      return this.call('help', {_: ['list']});
    }
    //调用hexo实例的load方法
    //加载数据库db.json
    //然后调用store中存储的type命令
    return this.load().then(function(){
      return store[type].call(self, args);
    });
    }

```

load函数内部核心代码：

```js
  //...省略

  //调用load_database.js文件，传入hexo实例
  return loadDatabase(this).then(function(){
  return Promise.all([
    self.source.process(),
    self.theme.process()
  ]);
  }).then(function(){
  //return;
  return self._generate({cache: true});
  }).nodeify(callback);
```

传入this参数是为了在loadDatabase内部调用hexo的db变量，也就是调用warehouse模块的load函数，db是在hexo构造函数中初始化的：
```js

  //Database为warehouse模块中的database.js
  this.database = new Database({
  version: dbVersion,//初始化为1
  path: pathFn.join(base, 'db.json')//传入文件名
  });

  //db.json
  return fs.exists(path).then(function(exist){
  if (!exist) return;

  log.debug('Loading database.');
    return db.load();//load函数之后再展开细讲
  }).then(function(){
    ctx._dbLoaded = true;
  }, function(){
    log.error('Database load failed. Deleting database.');
    return fs.unlink(path);
  });
```

## Hexo warehouse模块

### database.js构造函数

hexo在构造函数中实例化Database之后，就通过`register_models.js`注册了database的model，相当于数据库的模型，这样就为之后调用model做准备：

```js
  //传入options参数，如上面所说的version和path
  function Database(options){
  /**
   * Database options.
   *
   * @property {Object} options
   * @private
   */
   //调用扩展函数
  this.options = extend({
    version: 0,
    onUpgrade: function(){},
    onDowngrade: function(){}
  }, options);

  /**
   * Models.
   *
   * @property {Object} _models
   * @private
   */
  this._models = {};

  /**
   * Model constructor for this database instance.
   *
   * @property {Function} Model
   * @param {String} name
   * @param {Schema|Object} [schema]
   * @constructor
   * @private
   */
  var _Model = this.Model = function(name, schema){
    //内部调用Model.js
    Model.call(this, name, schema);
  };

  util.inherits(_Model, Model);
  _Model.prototype._database = this;
  }
```

### database.js model方法
```js

    //hexo构造函数：传入hexo对象
    registerModels(this);

    //registerModels内部代码：
    //加载所有模型对象
    var models = require('../models');

    module.exports = function(ctx){
    var db = ctx.database;//又一次引用database

    var keys = Object.keys(models);
    var key = '';

    for (var i = 0, len = keys.length; i < len; i++){
      key = keys[i];
      db.model(key, models[key](ctx));//db.model方法
    }
  };
```
其中`../models`包含的模型如下：

```js
  exports.Asset = require('./asset');
  exports.Cache = require('./cache');
  exports.Category = require('./category');
  exports.Data = require('./data');
  exports.Page = require('./page');
  exports.Post = require('./post');
  exports.PostAsset = require('./post_asset');
  exports.PostCategory = require('./post_category');
  exports.PostTag = require('./post_tag');
  exports.Tag = require('./tag');//tag模型
```
```js
  Database.prototype.model = function(name, schema){
    //构造函数中初始化this._models
    if (this._models[name]){
      return this._models[name];
    }
    //调用this.Model构造函数
    var model = this._models[name] = new this.Model(name, schema);
    return model;
  };
```

1.  流程如下：
> * hexo实例化初始化Database,然后注册各个model
> * hexo list命令调用database的load方法加载
