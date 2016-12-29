<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Events](#events)
  - [传递参数以及监听器的this](#%E4%BC%A0%E9%80%92%E5%8F%82%E6%95%B0%E4%BB%A5%E5%8F%8A%E7%9B%91%E5%90%AC%E5%99%A8%E7%9A%84this)
  - [异步和同步](#%E5%BC%82%E6%AD%A5%E5%92%8C%E5%90%8C%E6%AD%A5)
  - [一次性事件](#%E4%B8%80%E6%AC%A1%E6%80%A7%E4%BA%8B%E4%BB%B6)
  - [error 事件](#error-%E4%BA%8B%E4%BB%B6)
  - [Class: EventEmitter](#class-eventemitter)
    - [Event：newListener](#eventnewlistener)
    - [Event：removeListener](#eventremovelistener)
    - [EventEmitter.listenerCount(emitter, eventName)](#eventemitterlistenercountemitter-eventname)
    - [EventEmitter.defaultMaxListeners](#eventemitterdefaultmaxlisteners)
    - [emitter.addListener(eventName, listener)](#emitteraddlistenereventname-listener)
    - [emitter.emit(eventName[, ...args])](#emitteremiteventname-args)
    - [emitter.eventNames()](#emittereventnames)
    - [emitter.getMaxListeners()](#emittergetmaxlisteners)
    - [emitter.setMaxListeners(n)](#emittersetmaxlistenersn)
    - [emitter.listenerCount(eventName)](#emitterlistenercounteventname)
    - [emitter.listeners(eventName)](#emitterlistenerseventname)
    - [emitter.on(eventName, listener)](#emitteroneventname-listener)
    - [emitter.once(eventName, listener)](#emitteronceeventname-listener)
    - [emitter.prependListener(eventName, listener)](#emitterprependlistenereventname-listener)
    - [emitter.perpendOnceListener(eventName, listener)](#emitterperpendoncelistenereventname-listener)
    - [emitter.removeAllListeners([eventName])](#emitterremovealllistenerseventname)
    - [emitter.removeListener(eventName, listener)](#emitterremovelistenereventname-listener)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Events

> 稳定性：**2** - 稳定

大多数的 Node.js 核心API都是采用惯用的异步事件驱动架构构建的，通过某些对象（称为“emitters：发射器”）周期性的触发事件去调用函数对象（称为“listeners：侦听器”）。

例如：一个 [net.Server](#Todo) 对象在每次有新连接时都会触发一个事件；一个 [fs.ReadStream](#Todo) 对象会在文件被打开时触发一个事件；一个 [stream](#Todo) 对象会在每次数据可读时触发一个事件。

所有能触发事件的对象都是 `EventEmitter` 类的实例。这些对象都暴露出一个 `eventEmitter.on()` 方法，用于绑定一个或多个函数到该对象触发的命名事件上。通常情况下，事件名称都是以驼峰命名（camel-cased）的字符串，但也可以使用任何有效的 JavaScript 属性键值。

当 `EventEmitter` 对象发出一个事件时，所有连接到该特定事件的函数都是*同步*调用的。所有监听器返回的值都会被*忽略并丢弃*。

下面的例子展示了一个只有单个监听器的简单的 `EventEmitter`。`eventEmitter.on()` 用于注册监听器，`eventEmitter.emit()` 用于触发事件。

``` javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {};

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
	console.log('an event occurred!');
});

myEmitter.emit('event');
``` 

## 传递参数以及监听器的this

`eventEmitter.emit()` 方法允许传递任意参数给监听器函数。需要牢记一点，当一个普通的监听器函数被 `EventEmitter` 时，`this` 指向将被设置指向为该 `EventEmitter` 实例。

``` javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', function(a, b) {
	console.log(a, b, this);
	//	Prints:
	//		a b MyEmitter {
	//			damain: null,
	//			_events: { event: [Function] },
	// 			_eventsCount: 1,
	//			_maxListeners: undefined }
});
myEmitter.emit('event', 'a', 'b');
```

同样可以使用ES6箭头函数作为监听器，但是，这样做的话，监听器的 `this` 指向将不再指向 `EventEmitter` 实例。

``` javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
	console.log(a, b, this);
	// Prints: a b {}
});
myEmitter.emit('event', 'a', 'b');
```

## 异步和同步

`EventListener` 会按照监听器的注册顺序同步调用它的所有监听器。这在保证事件的正确时序上很重要，避免了条件竞态或逻辑错误的出现。在适当的时候，监听器函数可以通过 `setImmediate()` 或 `process.nextTick()` 方法实现异步操作模式。

``` javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
	setImmediate(() => {
		console.log('this happens asynchronously');
	});
});
myEmitter.emit('event', 'a', 'b');
```

## 一次性事件

当使用 `eventEmitter.on()` 方法注册一个监听器时，该监听器在*每次*事件触发时都会被调用。

``` javascript
const myEmitter = new MyEmitter();

var m = 0;
myEmitter.on('event', () => {
	console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
// Prints: 2
```

当使用 `eventEmitter.once()` 方法注册一个监听器时，这个监听器最多被调用*一次*，一旦调用就会立即取消注册。

``` javascript
const myEmitter = new MyEmitter();
var m = 0;
myEmitter.once('event', () => {
	console.log(++m);
});
myEmitter.emit('event');
// Print: 1
myEmitter.emit('event');
// Ignored
``` 

## error 事件

当一个 `EventEmitter` 实例内部发生错误时，一个比较典型的操作是去触发一个 `error` 事件。这在 Node.js 中是一个特例。

如果 `EventEmitter` 实例没有注册过至少一个 `error` 监听器，当 `error` 事件被触发时，将会抛出这个错误，打印堆栈跟踪，并退出 Node.js 进程。

``` javascript
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));
// Throws and crashes Node.js
```

为了防止 Node.js 进程崩溃，可以在 [process对象的uncaughtException事件] 上注册监听器或使用 [domain] 模块。（请注意：`domain` 模块*已弃用*）

``` javascript
const myEmitter = new MyEmitter();

process.on('uncaughtException', (err) => {
	console.log('whoops! there was an error');
});

myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```

作为最佳实践，应该总是在 `error` 事件上注册监听器。

``` javascript
const myEmitter = new MyEmitter();
myEmitter.on('error', (err) => {
	console.log('whoops! there was an error');
});
myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```

## Class: EventEmitter

`EventEmitter` 类由 `events` 模块定义和暴露：

``` javascript
const EventEmitter = requre('events');
```

所有的事件触发器都会在新的监听器被添加时触发 `'newListener'` 事件；在一个监听器被移除时触发 `'removeListener'` 事件。（译者注：`EventEmitter` 遵循先处理事件再调用监听器的原则）

### Event：newListener

* `eventName` <String>|<Symbol> 监听的事件名称
* `listener` <Function> 事件处理函数

`EventEmitter` 实例在一个监听器添加到内部的监听器数组*前*会触发自身的 `newListener` 事件。

注册了 `newListener` 事件的监听器将接收到注册的事件名和监听器的引用两个参数。

事实上，在事件触发前添加监听器会有一个微妙但重要的副作用：任何注册过监听器的事件在再注册另外的监听器前都会触发内部的 'newListener' 回调。

``` javascript
const myEmitter = new MyEmitter();
// Only do this once so we don't loop forever
myEmitter.once('newListener', (event, listener) => {
	if (event === 'event') {
		// Insert a new listener in front
		myEmitter.on('event', () => {
			console.log('B');
		})
	}
});
myEmitter.on('event', () => {
	console.log('A');
});
myEmitter.emit('event');
// Prints:
//		B
//		A
```

### Event：removeListener

* `eventName` <String>|<Symbol> 事件名称
* `listener` <Function> 事件处理函数

`removeListener` 事件将在一个 `linstener` 被移除后*触发*。

### EventEmitter.listenerCount(emitter, eventName)

> 稳定性：**0** - 已弃用 请使用 [emitter.listenerCount()](#emitterlistenercounteventname) 替代

``` javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', () => {});
myEmitter.on('event', () => {});
console.log(EventEmitter.listennerCount(myEmitter, 'event'));
// Prints: 2
```

### EventEmitter.defaultMaxListeners

默认任何单一事件最多可以注册 10 个监听器。每个 `EventEmitter` 实例都可以使用 [emitter.setMaxListeners(n)](#emittersetmaxlistenersn) 方法来解除这个限制。可以使用 `EventEmitter.defaultMaxListeners` 属性改变所有 `EventEmitter` 实例的默认设置。

请谨慎设置 `EventEmitter.defaultMaxListeners` 属性，因为这个改变会影响到所有的 `EventEmitter` 实例，包括那些之前创建的实例。但是，调用 `emitter.setMaxListeners(n)` 的优先级高于设置 `EventEmitter.defaultMaxListeners` 。

``` javascript
// 译者注：
const EventEmitter = require('events');

const myEvent = new EventEmitter;

let fun = function(data) {
	console.log(data);
}

myEvent.setMaxListeners(2);

myEvent.on('data', fun);
myEvent.on('data', fun);

EventEmitter.defaultMaxListeners = 3;

myEvent.on('data', fun);
console.log(EventEmitter.listenerCount(myEvent, 'data'));
```

请注意，这不是一个硬性限制。`EventEmitter` 实例允许添加更多的监听器但会输出一个 `stderr` 的跟踪警告：检测到一个 `possible EventEmitter memory leak` 错误。对于任何 `EventEmitter` 实例单独使用 `emitter.getMaxListeners()` 和 `emitter.setMaxListeners()` 方法可以暂时避免此警告：

``` javascript
emitter.setMaxListeners(emitter.getMaxListeners() + 1);
emitter.once('event', () => {
	// do stuff
	emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
})
```

使用 [--trace-warnings](../cli/#--trace-warnings) 命令行标识可以显示警告的堆栈跟踪

所有发出的警告对象都可以被 [process.on('warning')](#Todo) 获取，附加的 `emitter`, `type` 和 `count` 属性分别指的是 `EventEmitter` 实例、该事件的名称和绑定的监听器的数量。它的 `name` 属性被设置为 `MaxListenersExceededWarning`。

### emitter.addListener(eventName, listener)

`emitter.on(eventName, listener)` 的别名。

### emitter.emit(eventName[, ...args])

按照监听器的注册顺序同步调用每个以 `eventName` 注册的监听器，并将额外的参数传递给它们。

如果事件有监听器存在就返回 `true` ，否则返回 `false` 。

### emitter.eventNames()

返回一个代表该 `EventEmitter` 实例中已注册了监听器的事件名的数组。数组中的值将为字符串（String）或符号（Symbol）。

``` javascript
const EventEmitter = require('events');
const myEE = new EventEmitter();
myEE.on('foo', () => {});
myEE.on('bar', () => {});

const sym = Symbol('symbol');
myEE.on(sym, () => {});

console.log(myEE.eventNames());
// Prints: ['foo', 'bar', Symbol(symbol)]
```

### emitter.getMaxListeners()

返回当前 `EventEmitter` 实例的最大监听器数量，该值可能是通过 [emitter.setMaxListeners(n)](#emittersetmaxlistenersn) 设置的值或 [EventEmitter.defaultMaxListeners](#eventemitterdefaultmaxlisteners) 设置的默认值。

### emitter.setMaxListeners(n)

在默认情况下，EventEmitter 会在多于 10 个监听器监听某个事件的时候出现警告，此限制在寻找内存泄露时非常有用。很显然，并不是所有的事件都要被仅限为 `10` 个。`emitter.setMaxListeners()` 方法允许修改特定的 `EventEmitter` 实例的限制数量。如果想要不限制监听器的数量，可以将这个值设置为 `Infinity` （或 0）。

返回一个当前 `EventEmitter` 的引用以便链式调用。

### emitter.listenerCount(eventName)

* `eventName` <String> 被监听的事件名

返回 `eventName` 事件的监听器数量。

### emitter.listeners(eventName)

返回 `eventName` 的事件的监听器数组的副本。

``` javascript
server.on('connection', (stream) => {
	console.log('someone connected!');
});
console.log(util.inspect(server.listeners('connection')));
// Prints: [ [Function] ]
```

### emitter.on(eventName, listener)

* `eventName` <String>|<Symbol> 事件名称
* `listener` <Function> 事件处理函数

在监听器数组末尾添加名为 `eventName` 的 `listener` 函数，不会检测 `listener` 是否已经被添加。通过重复传递相同的 `eventName` 和 `listener` 的组合会导致 `listener` 被添加和调用多次。

``` javascript
server.on('connection', (stream) => {
	console.log('someone connected!');
});
```

返回一个当前 `EventEmitter` 的引用以便链式调用。

默认情况下，事件监听器将按它们添加的顺序调用。`emitter.prependListener()` 方法可以将事件监听器添加到监听器队列的开头。

``` javascript
const myEE = new EventEmitter();
myEE.on('foo', () => console.log('a'));
myEE.prependListener('foo', () => console.log('b'));
myEE.emit('foo');
// Prints:
//		b
//		a
```

### emitter.once(eventName, listener)

* `eventName` <String>|<Symbol> 事件名称
* `listener` <Function> 事件处理函数

给名为 `eventName` 的事件添加一个一次性的 `listener` 函数。在下一次 `eventName` 事件触发时，这个定时器将被先解除绑定后再调用。

``` javascript
server.once('connection', (stream) => {
	console.log('Ah, we hava our first user!');
});
```

返回一个当前 `EventEmitter` 的引用以便链式调用。

默认情况下，事件监听器将按它们添加的顺序调用。`emitter.perpendOnceListener()` 方法可以将事件监听器添加到监听器队列的开头。

``` javascript
const myEE = new EventEmitter();
myEE.once('foo', () => console.log('a'));
myEE.perpendOnceListener('foo', () => console.log('b'));
myEE.emit('foo');
// Prints:
//		b
//		a
```

### emitter.prependListener(eventName, listener)

* `eventName` <String>|<Symbol> 事件名称
* `listener` <Function> 事件处理函数

将 `listener` 函数添加到 `eventName` 事件的监听器队列的*开头*。不会检测 `listener` 是否已经被添加。通过重复传递相同的 `eventName` 和 `listener` 的组合会导致 `listener` 被添加和调用多次。

``` javascript
server.perpendListener('connection', (stream) => {
	console.log('someone connected!');
});
```

返回一个当前 `EventEmitter` 的引用以便链式调用。

### emitter.perpendOnceListener(eventName, listener)

* `eventName` <String>|<Symbol> 事件名称
* `listener` <Function> 事件处理函数

将一个一次性的 `listener` 函数添加到 `eventName` 事件的监听器队列的*开头*。在下一次 `eventName` 事件触发时，这个定时器将被先解除绑定后再调用。

``` javascript
server.perpendOnceListener('connection', (stream) => {
	console.log('Ah! we have our first user!');
});
```

返回一个当前 `EventEmitter` 的引用以便链式调用。

### emitter.removeAllListeners([eventName])

移除全部或某些特定 `eventName` 的监听器。

请注意，在代码中移除其他地方添加的监听器是一个不好的做法，尤其是由其他组件或模块（如，sockets(套接字)或streams(文件流)）创建的 `EventEmitter` 实例。

返回一个当前 `EventEmitter` 的引用以便链式调用。

### emitter.removeListener(eventName, listener)

从监听器队列中移除名为 `eventName` 的特定的 `listener` 。

``` javascript
var callback = (stream) => {
	console.log('someone connected!');	
};
server.on('connection', callback);
// ...
server.removeListener('connection', callback);
```

`removeListener` 最多只会从当前的监听器队列里移除一个监听器实例。如果任何单一的监听器多次添加特定的 `eventName` 到监听器数组中，必须多次调用 `removeListener` 才能移除每个实例。

请注意，一旦一个事件被触发，所有关联到它的监听器都能在那时依次触发。这也意味着，任何的 `removeListener()` 或 `removeAllListeners()` 在调用触发后和最后一个监听器执行完毕前不会在 `emit()` 过程中移除它们。随后的事件会像预期的那样发生。

``` javascript
const myEmitter = new MyEmitter();

var callbackA = () => {
    console.log('A');
    myEmitter.removeListener('event', callbackB);
};

var callbackB = () => {
    console.log('B');
};

myEmitter.on('event', callbackA);

myEmitter.on('event', callbackB);

// callbackA removes listener callbackB but it will still be called.
// Internal listener array at time of emit [callbackA, callbackB]
myEmitter.emit('event');
// Prints:
//   A
//   B

// callbackB is now removed.
// Internal listener array [callbackA]
myEmitter.emit('event');
// Prints:
//   A
```

因为监听器通过一个内部数组进行管理，调用该方法会在监听器被移除后改变其中任何已注册的监听器的索引值。虽然这不会影响监听器的调用顺序，但也意味着由 `emitter.listeners()` 方法返回的任何监听器副本都将需要重新获取。

返回一个当前 `EventEmitter` 的引用以便链式调用。