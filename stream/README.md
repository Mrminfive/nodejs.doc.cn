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

实现调用 [steam.push(chunk)](#Tode) 时，数据将被缓冲当在可读流中。如果流的消费者没有调用 `steam.read()`，数据将一直保留在内部队列中，直到它被消费。

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

### 可写流（Writable）

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

##### writable.write(chunk[,encoding][,callback])

* `chunk` <String>|<Buffer> 写入的数据
* `encoding` <String> 编码，如果 `chunk` 是字符串将使用该编码格式
* `callback` <Function> 可选的当流完成时的回调函数
* Returns: <Bollean> 如果返回 `false` 表示流希望在等待 `drain` 事件发出后再继续调用代码去写入附加数据，否则返回 `true`。

`writable.write()` 方法将一些数据写入流中，并在数据完全处理后调用提供的回调函数。如果发生错误，则回调*可能会也可能不会*将错误作为第一个参数调用。为了检测写入错误，请监听 'error' 事件。

如果允许数据写入后内部缓冲区小于创建时配置的 `highWaterMark`，则返回 `true`，如果返回 `false`，则表示内部缓冲区暂时无法支持数据的继续写入，应该停止向流中写入数据的操作，直到发出 `drain` 事件。

当一个流不消耗时，对 `write()` 的调用将缓存 `chunk`，并返回 `false`。一旦所有当前缓冲的 `chunk` 被排空（由操作系统接收传递），将发出 `drain` 事件。建议一旦 `write()` 方法返回 `false`，在 'drani' 事件发出之前不再写入任何 `chunk`。当对未排空的流调用 `write()` 方法时，Node.js 将缓存所有写入的 `chunk`，直到最大内存使用率，此时它将被强制中止。甚至在被中止前，内存占满将导致垃圾收集器的性能变的糟糕和 RSS（通常不会释放会系统，即使在不再需要内存之后）。由于TCP套接字可能永远不会耗尽，如果远程对等体不读取数据，编写未排空的套接字可能导致可被远程利用的漏洞产生。

对于 [双工流](#TODE) 来说在流不排出数据时写入数据尤其有问题，因为 `双工流` 被默认暂停，直到它们被管道化或者添加了 `data` 或 `readable` 事件后才会打开。

如果要写入的数据可以根据需要生成或提取，建议将配置为 [可读流](#TODE) 并使用 [stream.pipe()](#TODE) 方法。但是，如果优先使用 `write()` 的话，则可以使用 `drain` 事件来避免内存问题。

``` javascript
function write(data, cb) {
	if (!stream.write(data)) {
		stream.once('drain', cb);
	} else {
		process.nextTick(cb);
	}
}

// Wait for cb to be called before doing any other write.
write('hello', () => {
	console.log('write completed, do more writes now');
});
```

对象模式中的可写流将始终忽略 `encoding` 参数。


### 可读流（Readable）

可读流是一个你正在读取的数据*源*的抽象。

可读流的例子包括：

* [客户端上的 HTTP 请求](#TODE)
* [服务器上的 HTTP 请求](#TODE)
* [fs 读取流](#TODE)
* [zlib 流](#TODE)
* [crypto 流](#TODE)
* [TCP 套接字](#TODE)
* [子进程的 stdout 和 stderr](#TODE)
* [process.stdin](#TODE)

所有的[可读流](#TODE)都基于 `stream.Readable` 类实现。

#### 两种模式

可读流有两种有效的运行模式： 流动(flowing)和暂停(paused)。

当处在流动模式时，数据从底层系统自动读取，并通过 [EventEmitter](#TODE) 接口使用事件尽可能快的提供给应用程序。

在暂停模式下，必须显示调用 [stream.read()](#TODE) 方法以从流中读取数据块。

所有可读流都以暂停模式开始，但可以通过以下方式切换到流模式：

* 添加一个 ['data'](#TODE) 事件处理器。
* 调用 [stream.resume()](#TODE) 方法来开启数据流
* 调用 [stream.pipe()](#TODE) 方法将数据发送给一个[可写流](#TODE)。

可读流可以使用以下方法切换为暂停模式：

* 如果没有管道目标，则可以通过调用 [stream.pause()](#TODE) 方法切换。
* 如果有管道目标，则可以通过删除所有 ['data'](#TODE) 事件处理程序，并通过调用 [stream.unpipe()](#TODE) 方法删除所有管道目标。

要记住一个重要概念是，在没有提供消费或忽略该数据的机制之前，可读流是不会生成数据的。如果消费机制被禁用或移除，可读流将*尝试*停止生成数据。

*注意*：出于向后兼容的考虑，删除 ['data'](#TODE) 事件处理程序**并不会**自动暂停流。此外，当有导流目标时，调用 [stream.pause()](#TODE) 并不能保证流在那些目标排空并请求更多数据时维持暂停状态。

*注意*：如果[可读流](#TODE)切换到流动模式但没有可用的数据处理机制，该数据将丢失。例如，当在没有设置 `data` 事件处理程序的情况下调用 [readable.resume()](#TODE) 时或者当从流中移除 `data` 事件处理程序时，就会出现数据丢失。

#### 三种状态

可读流对“两种模式”的操作是在可读流中实现中发生的更复杂的内部状态管理的简单抽象。

具体来说，在任何时候，任何可读；流都处在以下三种状态之一：

* `readable._readableState.flowing = null`
* `readable._readableState.flowing = false`
* `readable._readableState.flowing = true`

当 `readable._readavkeState.flowing` 为 `null` 时，表示没有提供消耗流数据的机制，因此流不会生成其数据。

为流添加 `'data'` 事件监听器、调用 `readable.pipe()` 方法或调用 `readable.resume()` 方法会将 `readable._readableState.flowing` 切换为 `true`,  然后可读流才会在数据生成时主动触发事件。

调用 `readable.pause()`、`readable.unpipe()` 或接收“back pressure” 将会导致 `readable._readableState.flowing` 被设置为 `false`，并临时停止事件的传播，但不会停止数据的生成。

当 `readable._readableState.flowing` 为 `false` 时，数据可能会在流内部缓存区中积累。

#### 只选一种

可读流API随着 Node.js 版本的不断演进，提供了多种消耗流数据的方法。一般来说，开发人员应该只选择一种方法去使用数据而不是使用多种方法来从单个流中消耗数据。

建议大家使用 `readable.pipe()` 方法，因为它提供最简单的消耗流数据的方法。当然，需要对数据传输和生成进行更细粒度控制的开发人员可以使用 [EventEmitter](../events/#class-eventemitter) 和 `readable.pause()` / `readable.resume()` API 对流进行自定义控制。

#### stream.Readable 类

##### 'close' 事件

当流与底层数据源（例如：文件描述符）的联系被关闭时，发出 `close` 事件。该事件标识将不再触发其它事件，并且不会进行进一步的计算。

并不是所有的 [可读流](#TODE) 都会触发 `close` 事件。

##### 'data' 事件

* `chunk` [<Buffer>](#TODE)|<String>|<any> 数据块。对于不再对象模式下操作的流，块将是字符串或 Buffer。对于出于对象模式的流，块可以是处理 `null` 之外的任何 JavaScript 值。

在流准备好抛出数据块时，就会发出 `‘data’` 事件。每当通过调用 `readable.pipe()`、`readable.resume()`或将监视器回调函数绑定到 `'data'` 事件上等方式将流切换为流动模式时，将会发生这种情况。每当调用 `readable.read()` 方法并返回一个数据块时，`data` 事件也会被触发。（译者注：不理解的同学可以移步 [readable.read()] 方法）

为尚显示暂停的流绑定 `'data'` 事件将会将流切换为流模式。一旦数据可用，数据就会被传递。

如果使用 `readable.setEncoding()` 方法为流指定了默认编码，则监听器回调将用字符串传递数据块；否则将用 Buffer 传递数据。

``` javascript
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
	console.log(`Received ${chunk.length} bytes of data.`);
});
```

##### 'end' 事件

当流中无更多可被消费的数据时，触发 `end` 事件。

注意：在数据没有被完全消耗完之前，`end` 事件是**不会被触发**。可以通过将流切换为流模式或通过反复调用 [stream.read()] 方法直到所有数据都被抽空。

``` javascript
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
	console.log(`Received ${chunk.length} bytes of data.`);
});
readable.on('end', () => {
	console.log('There will be no more data.');
});
```

##### 'error' 事件

* [Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)

`error` 事件可以在可读流实现的任何时间发出。通常，如果底层的流由于底层内部故障而不能生成数据或者当流尝试推送无效的数据块时，将会触发。

监听器回调函数将接收到一个 `Error` 对象上。但是，流是可以在其它

##### 'readable' 事件

当数据可以从流中读取时触发 `'readable'` 事件。在某些情况下，假如未准备好，监听一个 `'readable'` 事件会使一些数据从底层系统被读出到内部缓冲区中。

``` javascript
const readable = getReadableStreamSomehow();
readable.on('readable', () => {
	// there is some data to read now
});
```

一旦到达数据流的末端，在没有触发 `end` 事件之前，将会触发一次 `readable` 事件。

实际上，`'readable'` 事件表示流的状态发生了改变：有新数据可用或这已经到达流的末端。在前一种情况下，调用 [stream.read()](#TODE) 方法将返回可用的数据。在后一种情况下，调用 [stream.read()](#TODE) 将返回 `null`。例如，下例中， foo.txt 是一个空文件：

``` javascript
const fs = require('fs');
const rr = fs.createReadStream('foo.txt');
rr.on('readable', () => {
	console.log('readable: ', rr.read());
});
rr.on('end', () => {
	console.log('end');
});
```

运行此脚本的输出：
``` javascript
$ node test.js
readable: null
end
```

注意：一般来说，更推荐使用 `readable.pipe()` 和 `'data'` 事件机制，而不是使用 `'readable'` 事件。


##### readable.isPaused()

* Returns: <Boolean>

`readable.isPaused()` 方法返回可读流的当前操作状态。该方法的实现机制基于 `readable.pipe()` 方法。在大多数情况下，我们并不需要直接使用该方法。

``` javascript
const readable = new stream.Readable;

readable.isPaused() // === false
readable.pause()
readable.isPaused() // === true
readable.resume()
readable.isPaused() // === false
```

##### readable.pause()

* Returns: `this`

`readable.pause()` 方法将使流动状态的流停止发出 `'data'` 事件，切换出流动模式。任何可用的数据将保留在内部缓存区中。

``` javascript
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
	console.log(`Received ${chunk.length} bytes of data.`);
	readable.pause();
	console.log('The will be no additional data for 1 second.');
	setTimeout(() => {
		console.log('Now data will start flowing again.');
		readable.resume();
	}, 1000);
});
```

##### readable.pipe(destination[,options])

* `destination` [<stream.Writable>](#TODE) 写入数据的目标
* `options` <Object> 管道参数
	* `end` <Boolean> 当读取结束时终止写入，默认为 `true`

`readable.pipe()` 方法将 [可写流](#TODE) 附加到当前的可读流上，使其自动切换到流动模式，并将其所有数据推送到附加的 [可写流](#TODE) 中。该方法能自动控制流量以避免目标被快速读取的可读流所淹没。

以下示例将所有可读流的数据导入名为 `file.txt` 的文件中：

``` javascript
const readable = getReadableStreamSomehow();
const writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt'
readable.pipe(writable);
```

可以将多个可写流附加到单个可读流上。

`readable.pipe()` 方法返回对目标流的应用，因此可以链式调用：

``` javascript
const r = fs.createReadStream('file.txt');
const z = zlib.createGzip();
const w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```

默认情况下，当源数据可读流发出 ['end'](#TODE) 事件时，目标可写流的 [stream.end()](#TODE) 方法会被调用，以便目标可写流不再可写。要禁用此默认行为，可以将 `end` 选项设置为 `false`（译者注：{ end: false }），以保持目标可写流的开启状态。如下例所示：

``` javascript
reader.pipe(writer, { end: false });
reader.on('end', () => {
	writer.end('Goodbye\n');
});
```

警告：如果可读流再处理期间发生了错误，则目标可写流是不会自动关闭的。如果发生错误，需要手动关闭每个流，以防止内存泄漏。

注意：[process.stderr](#TODE) 和 [process.stdout](#TODE) 在进程结束前是不会关闭的，无论是否指定选项。

##### readable.unpipe([destination])

* `destination` [<stream.Writable>](#TODE) 可选，指定解除导流的流。

`readable.unpipe()` 方法用于移除使用 [readable.pipe()](#TODE) 方法附加的可写流。

如果没有指定目标，将*移除所有*管道。

如果指定了目标，但是没有与之建立联系，则不做任何处理。

``` javascript
const readable = getReadableStreamSomehow();
const writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt',
// but only for the first second
readable.pipe(writable);
setTimeout(() => {
	console.log('Stop writing to file.txt');
	readable.unpipe(writable);
	console.log('Manually close the file stream');
	wirtable.end();
}, 1000);
```

##### readable.read([size])

* `size` <Number> 可选参数，指定要读取的数据量
* Return <String> | [<Buffer>](#TODE) | <Null>

`readable.read()` 方法将一些数据从内部缓冲区中取出并返回。如果没有可读的数据，则返回 `null`。默认情况下，数据将用 Buffer 对象返回，除非使用 `readable.setEncoding()` 方法指定了编码或者流在对象模式下运行。

如果你传了一个 `size` 参数，那么它就会返回多少字节的数据。如果 `size` 字节不可用，那么它将返回 `null`，除非已经到了数据末端，在这种情况下，它将返回保留在缓冲区中的数据（即使超过 `size` 字节）。

如果你没有指定 `size` 参数，那么它就会返回内部缓冲区中的所有数据。

该方法仅在暂停状态的流上才能调用。在流动模式下，`readable.read()` 方法将被自动调用，直到内部缓冲区完全排空。

``` javascript
const readable = getReadableStreamSomehow();
readable.on('readable', () => {
	var chunk;
	while(null !== (chunk = readable.read())) {
		console.log(`Received ${chunk.length} bytes of data.`);
	}
});
```

一般情况下，开发人员应避免使用 `readable` 事件和 `readable.read()` 方法，更建议使用 `readable.pipe()` 或 'data' 事件。

对象模式下的可读流调用 `readable.read(size)` 将总是返回单个项，忽略 `size` 参数的值。

注意：如果该方法返回了一个数据块，那么它也会触发 `'data'` 事件。

注意，在 ['end'](#TODE) 事件触发后调用 [stream.read([size])](#TODE) 将会返回 `null`，并且不会产生错误警告。

##### readable.resume()

* Returns: `this`

`readable.resume()` 方法将暂停状态的流切换到流动模式，使其继续发出 ['data'](#TODE) 事件。

`readable.resume()` 方法可以用来完全消耗流中的数据，而不会实际处理任何数据，如下所示：

``` javascript
getReadableStreamSomehow()
	.resume()
	.on('end', () => {
		console.log('Reached the end, but did not read anything');
	});
```

##### readable.setEncoding(encoding)

* `encoding` <String> 使用的编码
* Returns: `this`

`readable.setEncoding()` 方法用于设置从可读流中读取的数据的默认字符编码。

设置编码会导致流数据以指定编码的字符串返回而不是 Buffer 对象。例如，调用 `readable.setEncoding('utf8')` 将导致输出数据被解释为 UTF8 格式，并做字符串传递。调用 `readable.setEncoding('hex')` 将导致数据以十六进制编码的字符串格式传递。

该方法能妥善处理多字节字符，如果你直接取出 Buffer 并对它们调用 [buf.toString(encoding)](#TODE)，很可能会导致字节错位。如果你想要以字符串形式读取数据，请始终使用该方法。

你还可以使用 `readable.setEncoding(null)` 完全禁用任何编码。如果你在处理二进制数据或将大型的多字节字符串分成多块时，这种做法将非常有用。

``` javascript
const readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', (chunk) => {
	assert.equal(typeof chunk, 'string');
	console.log('got %d charactres of string data', chunk.length);
});
```

##### readable.unshift()

* `chunk` [<Buffer>](#TODE) | <String> 回读队列开头的数据块

该方法在某些情况下很有用，比如一个流正在被一个解析器消费，解析器需要“逆消费”某些刚从源中拉取出来的数据，以便流可以传递给其它消费者。

注意，`stream.unshift(chunk)` 不能在 ['end'](#TODE) 事件触发后调用，否则将产生一个运行时错误。

如果你发现你必须在你的程序中频繁调用 [stream.unshift(chunk)](#TODE) ，请考虑实现一个转换流（Transform）作为替代。（详见[面向流实现者的 API](#TODE)）

``` javascript
// Pull off a header delimited by \n\n
// use unshift() if we get too much
// Call the callback with (error, header, stream)
const StringDecoder = require('string_decoder').StringDecoder;

function parseHeader(stream, callback) {
	stream.on('error', callback);
	stream.on('readable', onReadable);

	const decoder = new StringDecoder('utf8');
	var header = '';

	function onReadable() {
		var chunk;
		while(null !== (chunk = stream.read())) {
			var str = decoder.write(chunk);
			if (str.match('/n/n')) {
				// found the header boundary
				var split = str.split(/\n\n/);
				header += split.shift();
				const remaining = split.join('\n\n');
				const bug = Buffer.from(remaining, 'utf8');
				stream.removeListener('error', callback);
				// set the readable listener before unshifting
				stream.removeListener('readable', onReadable);
				if (buf.length)
					stream.unshift(buf);
				// now the body of the message can be read form the stream.
				callback(null, header, stream);
			} else {
				// still reading the header.
				ehader += str;
			}
		}
	}
}
```

请注意，不像 [stream.push(chunk)](#TODE) 那样，`stream.unshift(chunk)` 不会通过重置流的内部读取状态结束读取过程。如果在读取过程（比如，在一个 [stream._read()](#TODE) 内部实现一个自定义流）中调用 `readable.unshift()` 将导致意想不到的结果。在调用 `readable.unshift()` 后立即调用 [stream.push('')](#TODE) 会适当地重置读取状态，然而最好简单地避免在执行一个读出过程中调用 unshift() 。

##### readable.wrap(stream)

* `stream` [<Stream>](#TODE) 一个“旧式”可读流

Node.js v0.10 版本之前的流并未实现现今所有流 API。（更多信息详见[“兼容性”章节](#TODE)。）

如果你正在使用一个早期版本的 Node.js 库，它会触发 ['data'](#TODE) 事件并且有一个仅作查询用途的 [stream.pause()](#TODE) 方法，那么你可以使用 `readable.wrap()` 方法来创建一个使用旧式流作为数据源的[可读流](#TODE)。

例如：

``` javascript
const OldReader = require(./old-api-module.js).OldReader;
const Readanle = require('stream.Readable');
const oreader = new OldReader;
const myReader = new Readable().wrap(oleader);

myReader.on('readable', () => {
	myReader.read(); // etc.
});
```


