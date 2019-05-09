title: database_find
date: 2016-10-03 23:44:30
tags: hexo 
---

## hexo 的json database之find

### 查找当前写的源文章
```js
	Box.prototype._getExistingFiles = function(){
	  var base = this.base;
	  var ctx = this.context;

	  //输出base:source\
	  //base:themes\landscape-f\
	  // console.log('base:'+base.substring(ctx.base_dir.length));

	  var relativeBase = escapeBackslash(base.substring(ctx.base_dir.length));

	  console.log('^' + escapeRegExp(relativeBase));
	  //输出^source\/
	  //^themes\/landscape\-f\/
	  var regex = new RegExp('^' + escapeRegExp(relativeBase));
	  var baseLength = relativeBase.length;

	  return this.Cache.find({_id: regex}).map(function(item){
	    return item._id.substring(baseLength);
	  });
};
```

我们看到`this.Cache.find`这句代码是从缓存中查找，那么Cache是什么呢？

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
	exports.Tag = require('./tag');


	var models = require('../models');

		module.exports = function(ctx){
		  var db = ctx.database;

		  var keys = Object.keys(models);
		  var key = '';

		  for (var i = 0, len = keys.length; i < len; i++){
		    key = keys[i];
		    db.model(key, models[key](ctx));
		  }
	};
```

是从models文件夹下注册的model，然后把model放入db中。

```js
	Database.prototype.model = function(name, schema){
	  if (this._models[name]){
	    return this._models[name];
	  }

	  var model = this._models[name] = new this.Model(name, schema);
	  return model;
	};
```

我们看下`Cache`的Model:
```js
	module.exports = function(ctx){
	  var Cache = new Schema({
	    _id: {type: String, required: true},
	    shasum: {type: String},
	    modified: {type: Number, default: Date.now}
	  });

	  return Cache;
	};
```

声明了一个Schema,Scheme构造函数中会把当前schema参数加入path中，以便于查询
```js
	if (schema){
    	this.add(schema);
  	}

  	Schema.prototype.add = function(schema, prefix_){
		  var prefix = prefix_ || '';
		  var keys = Object.keys(schema);
		  var len = keys.length;
		  var key, value;

		  if (!len) return;

		  for (var i = 0; i < len; i++){
		    key = keys[i];
		    value = schema[key];

		    this.path(prefix + key, value);
		  }
	};

	Schema.prototype.path = function(name, obj){
		  if (obj == null){
		    return this.paths[name];
		  }

		  var type;
		  var nested = false;

		  if (obj instanceof SchemaType){
		    type = obj;
		  } else {
		    switch (typeof obj){
		      case 'function':
		        type = getSchemaType(name, {type: obj});
		        break;

		      //比如Cache的model就是{type: String, required: true}
		      case 'object':
		        if (obj.type){
		          type = getSchemaType(name, obj);
		        } else if (isArray(obj)){
		          type = new Types.Array(name, {
		            child: obj.length ? getSchemaType(name, obj[0]) : new SchemaType(name)
		          });
		        } else {
		          type = new Types.Object();
		          nested = Object.keys(obj).length > 0;
		        }

		        break;

		      default:
		        throw new TypeError('Invalid value for schema path `' + name + '`');
		    }
		  }

		  this.paths[name] = type;
		  this._updateStack(name, type);

		  if (nested) this.add(obj, name + '.');
	};

	function getSchemaType(name, options){
		  var Type = options.type || options;
		  var typeName = Type.name;

		  //String就是内置的类型
		  if (builtinTypes[typeName]){
		    return new Types[typeName](name, options);
		  } else {
		    return new Type(name, options);
		  }
	}

	var builtinTypes = {
	  String: true,
	  Number: true,
	  Boolean: true,
	  Array: true,
	  Object: true,
	  Date: true
	};

```

最重要的就是各种Types:

```js
	exports.Mixed = require('../schematype');
	exports.String = require('./string');
	exports.Number = require('./number');
	exports.Boolean = require('./boolean');
	exports.Array = require('./array');
	exports.Object = require('./object');
	exports.Date = require('./date');
	exports.Virtual = require('./virtual');
	exports.CUID = require('./cuid');
	exports.Enum = require('./enum');
	exports.Integer = require('./integer');
```

比如String:

```js
	function SchemaTypeString(name, options){
	  SchemaType.call(this, name, options);
	}

	//
	SchemaTypeString.prototype.match = function(value, query, data){
	  if (typeof query.test === 'function'){
	    return query.test(value);
	  } else {
	    return value === query;
	  }
	};
```
这边的match方法就是在schema类的_parseQuery中有可能用到的，下回再讲。
