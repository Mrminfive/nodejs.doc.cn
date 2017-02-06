# Stream（流）

> 稳定性：**2** - 稳定

流是用于在 Node.js 中处理数据流的抽象接口。`Stream` 模块提供了一个基本的API，用于快速构建实现了流接口的对象。

Node.js 中提供了许多内置的流对象。例如，[对一个HTTP服务器的请求](#Tode)和[process.stdout](#Tode)都是流实例。

流可以是可读、可写或者两者兼具的。所有的流都是[EventEmitter](../events/#class-eventemitter)实例。

`stream` 模块可以这样访问：

``` javascript
const stream = require('stream');
```

尽管 `stream` 模块可以让开发人员很容易去创建新类型的流实例，但每个 Node.js 使用者都应该明白流是如何工作的。

## 本文档结构

本文档分为两个主要部分，第三部分用于附加注释。第一部分介绍了在应用程序中使用流所需要的 API 。第二部分介绍了实现一个新的流对象需要的 API。

## 流的类型

Node.js 中有四种基本的流类型：

* [可读](#Tode) - 可以读取数据的流（例如：[fs.createReadStream()](#Tode)）。
* [可写](#Tode) - 可以写入数据的流（例如：[fs.createWriteStream()](#Tode)）。
* [双工](#Tode) - 可读可写的流（例如：[net.Socket](#Tode)）
* [转换](#Tode) - 可以在写入和读写数据时修改或转换数据的双工流（例如：[zlib.createDeflate()](#Tode)）

### 对象模式

所有由 Node.js API 创建的流都只运行在字符串和 `Buffer` 对象上。但是，流是可以在其它 JavaScript 类型（除了 `null`，应为它在流中有特殊的用途）上运行的，这样的流被认为是在“对象模式”中操作的流。

创建流时，流实例将使用 `objectMode` 选项去切换到对象模式。不应该将现有的流切换到对象模式。


### 缓冲

无论是 [可写流](#Tode) 还是 [可读流](#Tode)，它们都会在内部对象中缓存数据，这两个内部对象分别可以从 `writable._writeableState.getBuffer()` 或 `readable._readableState.buffer` 中获取。

缓冲的数据量大小取决于传递到流构造函数种的 `highWaterMark` 选项。对于正常的流，`highWaterMark` 选项指定的是总字节数。对于在对象模式下操作的流，`higtWaterMark` 选项指定的是对象的总数。

实现调用[steam.push(chunk)](#Tode)时，数据将被缓冲当在可读流中。如果流的消费者没有调用 `steam.read()`，数据将一直保留在内部队列中，直到它被消费。

一旦内部用于读的缓冲区的总大小达到了由 `highWaterMark` 设定的阈值，则流将临时停止从底层资源读取数据，直到可以使用当前缓冲的数据（即流将停止调用内部的 `readable._read()` 方法去填充用于读的缓冲区）。

当重复调用 `writable.write(chunk)` 方法时，数据将被缓冲在可写流中。当内部用于写的缓冲区的总大小低于 `higWaterMark` 设定的阈值时，对 `writable.write()` 的调用将返回 `true`。一旦内部缓冲区的大小达到或超过 `highWaterMark` 阈值时，将返回 `false`。

流提供的 API，特别是 `stream.pipe()` 方法的主要目的是为了将数据的缓存控制在可接受的范围内，使得不同速度的资源和目标不会占满内存。

由于[双工](#Tode)和[变换流](#Tode)都是可读和可写的，每种流都保持两个单独的内部缓冲区用于读取和写入，并允许一侧独立于另一个操作，同时保证数据流的正确及有效。例如，[net.Socket](#Tode)实例是一个[双工流](#Tode)，其 Readable(可读) 端允许消耗从 socket(套接字) 接收的数据，并且其 Writable(可写) 端允许向 socket(套接字) 写数据。因为可能需要以比接收数据更快或更慢的速率将数据写入 socket(套接字) ，所以每一测操作（和缓冲）独立于另一侧是很重要的。


## 面向流消费者的API

几乎所有的 Node.js 应用程序，无论多么简单，都以某种方式使用流。以下是在 Node.js 应用程序中使用流实现 HTTP 服务器的实例：

``` javascript
const http  = require('http');

const server = http.createServer((req, res) => {
	// req is an http.IncomingMessage, which is a Readable Stream
	// res is an http.ServerResponse, whice is a Writable Stream

	let body = '';
	// Get the data as utf8 strings.
	// If an encoding is not set, Buffer objects will be received.
	req.setEncoding('utf8');

	// Readable streams emit 'data' events once a listener is added
	req.on('data', (chunk) => {
		body += chunk;
	});

	// the end event indicates that the entire body has been received
	req.on('end', () => {
		try {
			const data = JSON.parse(body);
			// write back something interesting to the user:
			res.write(typeof data);
			res.end();
		} catch (er) {
			// uh oh! bad json!
			res.statusCode = 400;
			return res.end(`error: ${er.message}`);
		}
	});
});

server.listen(1337);

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

[可写流](#Tode)（如实例中的 `res`）暴露一些方法，例如 `write()` 与 `end()` 可用于将数据写入到该流中。

[可读流](#Tode)使用[EventEmitter](../events/#class-eventemitter)的 API 在数据可用时通知应用程序代码读取流中的数据。可以通过多种方式从流中读取可用数据。

[可写流](#Tode)和[可读流](#Tode)都以各种方式使用[EventEmitter](../events/#class-eventemitter)的API来传达流的当前状态。

[双工流](#Tode)和[变换流](#Tode)都是[可读](#Tode)和[可写](#Tode)的；

向流写入数据或从流消费数据的应用不需要直接实现流接口，并且一般情况下并不需要调用到 `requrie('stream')`。

如果你正在自己的程序中实现流接口，请参阅[面向实现者的API](#Tode)。

### 可写流

可写流是数据被写入的*目的地*的抽象。

可写流的例子包括：

* [客户端上的HTTP请求](#TODE)
* [服务端上的HTTP响应](#TODE)
* [fs可写流](#TODE)
* [zlib流](#TODE)
* [crypto流](#TODE)
* [TCP sockets(套接字)](#TODE)
* [子进程的stdin](#TODE)
* [process.stdout, process.stderr](#TODE)

注意：这些实例中的一些实际上是[双工流](#TODE)只是表现为[可写流](#TODE)的形式。

所有的[可写流](#TODE)都基于 `stream.Writable` 类实现。

虽然可写流的特定实例的实现方式可能不一致，但是所有的可写流都遵循以下实例中所示相同的基本使用模式：

``` javascript
const myStream = getWritableStreamSomehow();
myStream.write('some data');
myStream.write('some more data');
myStream.end('done writing data');
```

#### stream.Writable类

##### 'close'事件

当流与底层数据源（例如：文件描述符）的联系被关闭时，发出 `close` 事件。该事件标识将不再触发其它事件，并且不会进行进一步的计算。

并不是所有的流都会触发 `close` 事件。

##### 'drain'事件

如果调用 [stream.write(chunk)](#TODE) 返回 `false` (译者注：即内部数据缓冲区的大小超过阈值，数据无法写入)，`drain` 事件将在流允许写入更多的数据的时候触发。

``` javascript
// Write the data to the supplied writable stream one million times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
	let i = 1000000;
	write();

	function write() {
		var ok = true;
		do {
			i--;
			if (i === 0) {
				// last time!
				writer.write(data, encoding, callback);
			} else {
				// see if we should continue, or wait
				// don't pass the callback, because we're not done yet.
				ok = writer.write(data, encoding);
			}
		} while (i > 0 && ok);

		if (i > 0) {
			// had to stop early!
			// write some more once it drains
			writer.once('drain', write);
		}
	}
}
```

##### 'error'事件

`error` 事件会在写入或导流数据发生错误时触发。在被调用侦听器回调函数时会传递一个错误参数。

注意：只有在流未关闭的状态下 `error` 事件才会触发。

##### 'finish'事件

`finish` 事件将会在调用 [stream.end()](#TODE) 方法，并且缓冲区的所有数据都更新到底层系统中后触发。

``` javascript
const writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i++) {
	writer.write(`hello, #${i}!\n`);
}
writer.end('This is the end\n');
writer.on('finish', () => {
	console.error('All writes are now complete.');
})
```

##### 'pipe'事件

* `src` [<stream.Readable>](#TODE) 被导流到该可写流的可读流。

当在可读流上调用 [stream.pipe()](#TODE) 方法并将当前可写流作为目标时，将会触发 `pipe` 事件。

``` javascript
const writer = getWriteableStreamSomehow();
const reader = getReadableStreamSomehow();
writer.on('pipe', (src) => {
	console.error('something is piping into the writer');
	assert.equal(src, reader);
});
reader.pipe(writer);
```

##### 'unpipe'事件

* `src` [<stream.Readable>](#TODE) 被解除导流到该可写流的可读流。

当在可读流上调用 [stream.unpipe()](#TODE) 方法并将当前可写流从目标集中解除时，将会触发 `unpipe` 事件。

``` javascript
const writer = getWritableStreamSomehow();
const reader = getReadableStreamSomehow();
writer.on('unpipe', (src) => {
	console.error('Something has stopped piping into the writer.');
	assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

##### writable.cork()

`writable.cork()` 将强制要写入的数据进入缓存。当调用 [stream.uncork()](#TODE) 或 [stream.end()](#TODE) 方法时才会从缓存中吐出并开始写入。

使用 `writable.cork()` 方法的主要目的时为了避免在将多个小块数据写入流的情况下，在内部缓冲区中产生对性能有影响的备份。在这种情况下，可以使用 `writable._writev()` 方法，以更加优化的方式执行缓冲和写入。

##### writable.uncork()
将由于调用 `stream.cork()` 方法缓存起来的数据吐出并进行写入。

当使用 `writable.cork()` 和 `writable.uncork()` 来对流的写入缓存进行管理时，建议使用 `process.nextTick()` 来延迟对 `writable.uncork()` 的调用。这样做可以一次性处理在给定的 Node.js 事件循环阶段发生的所有 `writable.write()` 调用。

``` javascript
stream.cork();
stream.write('some ');
stream.write('data ');
process.nextTick(() => stream.unrock());
```

如果在流上多次调用 `writable.cork()` 方法，则必须调用相同数量的 `writable.uncork()` 方法来吐出缓存的数据（译者注：在所有阀门未打开前，数据不会写入）。

``` javascript
stream.cork();
stream.write('some ');
stream.cord();
stream.write('data ');
process.nextTick(() => {
	stream.uncork();
	// 在第二次调用 uncork() 之前不会写入数据
	stream.uncork();
});
```

##### writable.end([chunk][,encoding][,callback])

* `chunk` <String>|<Buffer>|<any> 要写入的可选数据。对于不以对象模式操作的流，`chunk` 必须是字符串或 [Buffer](#TODE)。对于对象模式的流，`chunk` 可以使除 `null` 外的任何 JavaScript 值。
* `encoding` <String> 编码，如果 `chunk` 是字符串将使用该编码格式
* `callback` <Function> 可选的当流完成时的回调函数

调用 `writable.end()` 方法表示不再有数据会被写入可写流中。传递可选的 `chunk` 和 `encoding` 参数将在关闭流之前立即写入一个最后的附加数据块。如果提供可选的回调参数，则回调函数将附加到 `finish` 事件的监听器上。

在 [stream.end()][#TODE] 方法之后调用 [stream.write()][#TODE] 方法将抛出一个错误

``` javascript
// write 'hello, ' and then end with 'world!'
const file = fs.createWriteStream('example.txt');
file.write('hello, ');
file.end('world!');
// writing more now is not allowed!
```

##### writable.setDefaultEncoding(encoding)

* `encoding` <String> 新的默认编码
* Returns: `this`

`writable.setDefaultEncoding()` 方法用于设置一个[可写](#TODE)流的默认 `encoding`。

