title: hexo流程梳理
date: 2016-10-15 21:58:37
tags: hexo
---

## hexo 命令注册

hexo构造函数中实例化extend.console,在hexo cli中先注册hexo help version命令

核心的注册逻辑：
```js
	  var c = this.store[name.toLowerCase()] = fn;
	  c.options = options;
	  c.desc = desc;

	  this.alias = abbrev(Object.keys(this.store));
```
### abbrev模块

abbrev是Isaac Z. Schlueter 创建的一个`handy for command-line scripts, or other cases where you want to be able to accept shorthands.`(专门为了处理命令行脚本，或者接受速记)

Isaac Z. Schlueter ：npm inventor and CEO. Early contributor and former BDFL of Node.js. Author of many JavaScripts. Been making internets for a pretty long time.
(npm的创造者和CEO)

[BDFL](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life)：Benevolent Dictator For Life (BDFL) is a title given to a small number of open-source software development leaders, typically project founders who retain the final say in disputes or arguments within the community.
是给少数开源软件开发领导者的标题，类似在社区里有说话权的项目创始人

比如以下几位都是nodejs的[贡献者](http://nodeguide.com/community.html)：
    [Ryan Dahl](https://github.com/ry)(跟PAAS Heroku 有关)
    [Isaac Schlueter](https://github.com/isaacs) (在Oakland CA 奥克兰（美国加利福尼亚州西部港市）)
    [Bert Belder](https://github.com/piscisaureus) (为node提供windows支持的主要开发者)
    [TJ Holowaychuk](https://github.com/visionmedia) (大名鼎鼎的TJ,express,jade的作者)
    [Tim Caswell](https://github.com/creationix) (connect的作者)
    [Felix Geisendörfer](https://github.com/felixge) ( works on projects node-formidable, node-mysql 在Berlin, Germany （德国柏林）)
    [Mikeal Rogers](https://github.com/mikeal) (request的作者，在旧金山 San Francisco)
    [Alexis Sellier](https://github.com/cloudhead) ( less.js, vows 在柏林)
    [Jeremy Ashkenas](https://github.com/jashkenas) ( CoffeeScript, underscore, backbone, docco 的作者 在纽约 NYC)
    [Jed Schmidt](https://github.com/jed) (  fab.js )
    [Marak Squires](https://github.com/marak) (mostly known for pumping out dozens of node.js libraries per month)
    [Peteris Krumins](https://github.com/pkrumins) ( browserling 在拉脱维亚)
    [James Halliday](https://github.com/substack) ( dnode, optimist and browserify的作者)

abbrev是在注册命令时为了存储别名时用的一个模块，比如输入`hexo n`的时候hexo会识别出是调用了`new`命令。就像它在readme.md中所说的Just like [ruby's Abbrev](http://apidock.com/ruby/Abbrev).就像ruby中的Abbrev模块。

Usage:

```js
var store = {

	'new' : function(){},
	'deploy' : function(){},
	'publish': function(){}
};

 console.log(Object.keys(store));
//print array 
//[ 'new', 'deploy', 'publish' ]

var alias = abbrev(Object.keys(store));

console.log(alias);

//print 
console.log(alias);

{ d: 'deploy',
  de: 'deploy',
  dep: 'deploy',
  depl: 'deploy',
  deplo: 'deploy',
  deploy: 'deploy',
  n: 'new',
  ne: 'new',
  new: 'new',
  p: 'publish',
  pu: 'publish',
  pub: 'publish',
  publ: 'publish',
  publi: 'publish',
  publis: 'publish',
  publish: 'publish' }
 ```

 ## hexo 命令获取

 通过存储在Console实例中的alias字典对象中获取实际的命令Key(alias字典对象就是通过调用abbrev生成的),然后去实际的store对象中获取命令。

 核心的获取逻辑：

 ```js
 	Console.prototype.get = function(name){
	  name = name.toLowerCase();
	  return this.store[this.alias[name]];
	};
 ```

