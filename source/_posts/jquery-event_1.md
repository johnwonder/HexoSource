title: 'jquery_event[1]'
date: 2016-08-29 21:44:44
tags: jquery
---

##   jquery1.0源码解读

### 添加元素事件

1.0源码:
```js
	new function(){

		var e = ("blur,focus,load,resize,scroll,unload,click,dblclick," +
			"mousedown,mouseup,mousemove,mouseover,mouseout,change,reset,select," +
			"submit,keydown,keypress,keyup,error").split(",");
		//比如$("#element").click(function(){});其实执行的就是bind
		// Go through all the event names, but make sure that
		// it is enclosed properly
	    //适当地封闭
	    //包装在一个function中执行
		for ( var i = 0; i < e.length; i++ ) new function(){

			var o = e[i];

			// Handle event binding
			//处理事件绑定 调用bind方法
			jQuery.fn[o] = function(f){
				//如果没参数f绑定 那么就执行事件。。
				return f ? this.bind(o, f) : this.trigger(o);
			};

			// Handle event unbinding
			jQuery.fn["un"+o] = function(f){ return this.unbind(o, f); };

			// Finally, handle events that only fire once
			//绑定只能触发一次事件
			jQuery.fn["one"+o] = function(f){
				// Attach the event listener
				//遍历元素绑定
				return this.each(function(){

					//巧妙运用闭包
					var count = 0;

					// Add the event
					jQuery.event.add( this, o, function(e){
						// If this function has already been executed, stop
						//只能执行一次
						if ( count++ ) return;

						// And execute the bound function
						return f.apply(this, [e]);
					});
				});
			};

		};

		// If Mozilla is used
		if ( jQuery.browser.mozilla || jQuery.browser.opera ) {
			// Use the handy event callback
			document.addEventListener( "DOMContentLoaded", jQuery.ready, false );

		// If IE is used, use the excellent hack by Matthias Miller
		// http://www.outofhanwell.com/blog/index.php?title=the_window_onload_problem_revisited
		} else if ( jQuery.browser.msie ) {

	    // Only works if you document.write() it
	    //script标签的defer属性，这个defer属性是IE独有的。当它被设为true的时候，表示这段script要等文档加载好了才执行。
			document.write("<scr" + "ipt id=__ie_init defer=true " +
				"src=//:><\/script>");

			// Use the defer script hack
			var script = document.getElementById("__ie_init");
			script.onreadystatechange = function() {
				if ( this.readyState == "complete" )
					jQuery.ready();
			};

			// Clear from memory
			script = null;

		// If Safari  is used
		} else if ( jQuery.browser.safari ) {
			// Continually check to see if the document.readyState is valid
			//用个定时器轮询 1.2.6改为setTimeout( arguments.callee, 0 );
			//1.4.3改为document.addEventListener
			jQuery.safariTimer = setInterval(function(){
				// loaded and complete are both valid states
				if ( document.readyState == "loaded" ||
					document.readyState == "complete" ) {

					// If either one are found, remove the timer
					clearInterval( jQuery.safariTimer );
					jQuery.safariTimer = null;

					// and execute any waiting functions
					jQuery.ready();
				}
			}, 10);
		}

		// A fallback to window.onload, that will always work
		jQuery.event.add( window, "load", jQuery.ready );

	};
```

### jQuery.event.add函数：
```
	// Bind an event to an element
		// Original by Dean Edwards
		add: function(element, type, handler) {
			// For whatever reason, IE has trouble passing the window object
			// around, causing it to be cloned in the process
			if ( jQuery.browser.msie && element.setInterval != undefined )
				element = window;

			// Make sure that the function being executed has a unique ID
			if ( !handler.guid )
				handler.guid = this.guid++;

			// Init the element's event structure
			//初始化元素事件结构
			if (!element.events)
				element.events = {};

			// Get the current list of functions bound to this event
			var handlers = element.events[type];

			// If it hasn't been initialized yet
			if (!handlers) {
				// Init the event handler queue
				handlers = element.events[type] = {};

				// Remember an existing handler, if it's already there
				if (element["on" + type])
					handlers[0] = element["on" + type];
			}

			// Add the function to the element's handler list
			//核心一句 添加此handler函数到handlers 对象
			handlers[handler.guid] = handler;

			// And bind the global event handler to the element
			//最终执行的是jQuery.event.handle方法
			//handle方法内部遍历调用events[type]
			//还有阻止冒泡
			element["on" + type] = this.handle;

			// Remember the function in a global list (for triggering)
			if (!this.global[type])
				this.global[type] = [];
			this.global[type].push( element );
			//放到global对象的type数组中
			//用于trigger函数调用
		}
```

### jQuery.fn.bind函数
```js
		bind: function( type, fn ) {
			//如果fn是字符串
			//比如".css()",那么就执行$(this).css()方法
			if ( fn.constructor == String )
				fn = new Function("e", ( !fn.indexOf(".") ? "$(this)" : "return " ) + fn);
			jQuery.event.add( this, type, fn );
		}
```
