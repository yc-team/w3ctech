node-events-info
================


参考：

[源码地址](https://github.com/joyent/node/blob/master/lib/events.js)



## Class: events.EventEmitter

通过require('events').EventEmitter来访问EventEmitter类


### emitter.addListener(event,listener) 和 emitter.on(event,listener)


源码：

```javascript

EventEmitter.prototype._events = underfined;
EventEmitter.prototype._maxListeners = underfined;



EventEmitter.prototype.on = EventEmitter.prototype.addListener;


EventEmitter.prototype.addListener = function addListener(type, listener){
	var m;

	//check is function
	if (!util.isFunction(listener)) {
		throw TypeError('listener must be a function');
	}

	if (!this._events) {
		this._events = {};
	}


	//check if bind newListener
	//if true, emit it
	if (this._events.newListener) {
		//util.isFunction(listener.listener)  -- really someone use this way?
		this.emit('newListener', type, util.isFunction(listener.listener) ? listener.listener : listener);
	}

	if (!this._events[type]) {
		//has no key type
		//put {type: listener} in this._events
		this._events[type] = listener;
	} else if (util.isObject(this._events[type])){
		//if key type is already in this._events
		//like array just push
		this._events[type].push(listener);
	} else {

		this._events[type] = [this._events[type], listener];
	}


	if (util.isObject(this._events[type]) && !this._events[type].warned) {
		var m;
		if (!util.isUndefined(this._maxListeners)) {
			m = this._maxListeners;
		} else {
			m = EventEmitter.defaultMaxListeners;
		}
	

		if (m && m > 0 && this._events[type].length > m) {
			//set type.warned = true here
			this._events[type].warned = true;

			console.error('(node) warning: possible EventEmitter memory ' +
	                    'leak detected. %d %s listeners added. ' +
	                    'Use emitter.setMaxListeners() to increase limit.',
	                    this._events[type].length, type);

	        //trace
	        console.trace();

		}

	}

	return this;

};

```


添加一个listener到给指定的event的listeners数组的最后一个。

```javascript
server.on('connection', function(stream){
	console.log('someone connected');
});
```

返回emitter，方便链式调用。



### emitter.once(event,listener)

给event添加指定一次性的事件，这个事件只会在下一次事件发生时候被触发一次，触发完成后就被删除了

```javascript
server.once('connection', function(stream){
	console.log('one time by zhangyaochun');
});
```

返回emitter，方便链式调用。



### emitter.removeListener(event,listener)

从指定的event的listener数组里面删除一个listener。

> 注意：这样会改变所有在这个listener后面的下标。

```javascript
var callback = function(){
	console.log('hello by zhangyaochun');
};

server.on('connection', callback);

server.removeListener('connection', callback);
```

返回emitter，方便链式调用。



### emitter.removeAllListeners([event])

删除所有的listeners，或者这些指定的event。


返回emitter，方便链式调用。


### emitter.setMaxListeners(n)

默认情况下，EventEmitters会在给特定的event绑定多于10个listeners的时候出现警告。
在发现内存泄露时候很有用。当然也不是所有的Emitters都需要限制到10。可以通过这个方法也增加。设置0的化，就没有限制。


源码展示：

```javascript
EventEmitter.prototype.setMaxListeners = function setMaxListeners(n) {
	//check n type
	if (!util.isNumber(n) || n < 0 || isNaN(n)) {
		throw TypeError('n must be a positive number');
	}	

	//set _maxListeners as n
	this._maxListeners = n;

	// for chained use
	return this;
};
```




### emitter.listeners(event)

返回指定event的listeners数组。

```javascript
server.on('connection', function(stream){
	console.log('hello by zhangyaochun');
});

console.log(util.inspect(server.listeners('conection'))); //[ [Function] ]
```


### 类方法：EventEmitter.listenerCount(emitter, event)

返回指定event的listeners的个数。



### emitter.emit(event, [arg1],[arg2],[...])

如果event有listeners，返回true，否则就是false。




### 事件：newListener

* event    {String}
* listener {Function}

在添加一个新的listener的时候触发。不确定这个listener参数是否在emitter.listeners(event)返回的list里面。


### 事件：removeListener

* event    {String}
* listener {Function}

在移除一个listener的时候触发。不确定这个listener参数是否在emitter.listeners(event)返回的list里面。

