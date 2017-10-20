title: jquery_pushstack函数分析
date: 2016-09-04 22:10:07
tags: jquery
---

##   jquery1.0源码解读

### pushStack函数

pushStack是个对象函数，不是静态方法，用jQuery.pushStack调用会报错，我们来看代码：
```javascript
	//实例方法保证a是数组。。。
	pushStack: function(a,args) {
		//获取args中的最后一个参数
		var fn = args && args[args.length-1];

		if ( !fn || fn.constructor != Function ) {
			内部维护一个stack数组。用来pop 或者push
			if ( !this.stack ) this.stack = [];
			this.stack.push( this.get() );//get()把当前对象压入堆栈
			//jQuery.map( this, function(a){ return a } ) :
			this.get( a );
		} else {
			var old = this.get();//里面调用map function ,返回当前jquery对象
			this.get( a );//貌似是为了执行fn
			if ( fn.constructor == Function )
				return this.each( fn );
			this.get( old );//重新返回old元素数组
		}

		return this;
	}
```

我们看他关联的end函数：
```javascript
	end: function() {
		return this.get( this.stack.pop() );//stack 保存了元素数组,get把当前元素数组 push 到新数组，1.0还没有prevObject
	}
```
