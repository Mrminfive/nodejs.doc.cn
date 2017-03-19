<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Stream（流）](#stream%E6%B5%81)
  - [本文档结构](#%E6%9C%AC%E6%96%87%E6%A1%A3%E7%BB%93%E6%9E%84)
  - [流的类型](#%E6%B5%81%E7%9A%84%E7%B1%BB%E5%9E%8B)
    - [对象模式](#%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%BC%8F)
    - [缓冲](#%E7%BC%93%E5%86%B2)
  - [面向流消费者的API](#%E9%9D%A2%E5%90%91%E6%B5%81%E6%B6%88%E8%B4%B9%E8%80%85%E7%9A%84api)
    - [可写流（Writable）](#%E5%8F%AF%E5%86%99%E6%B5%81writable)
      - [stream.Writable类](#streamwritable%E7%B1%BB)
        - ['close'事件](#close%E4%BA%8B%E4%BB%B6)
        - ['drain'事件](#drain%E4%BA%8B%E4%BB%B6)
        - ['error'事件](#error%E4%BA%8B%E4%BB%B6)
        - ['finish'事件](#finish%E4%BA%8B%E4%BB%B6)
        - ['pipe'事件](#pipe%E4%BA%8B%E4%BB%B6)
        - ['unpipe'事件](#unpipe%E4%BA%8B%E4%BB%B6)
        - [writable.cork()](#writablecork)
        - [writable.uncork()](#writableuncork)
        - [writable.end([chunk][,encoding][,callback])](#writableendchunkencodingcallback)
        - [writable.setDefaultEncoding(encoding)](#writablesetdefaultencodingencoding)
        - [writable.write(chunk[,encoding][,callback])](#writablewritechunkencodingcallback)
    - [可读流（Readable）](#%E5%8F%AF%E8%AF%BB%E6%B5%81readable)
      - [两种模式](#%E4%B8%A4%E7%A7%8D%E6%A8%A1%E5%BC%8F)
      - [三种状态](#%E4%B8%89%E7%A7%8D%E7%8A%B6%E6%80%81)
      - [只选一种](#%E5%8F%AA%E9%80%89%E4%B8%80%E7%A7%8D)
      - [stream.Readable 类](#streamreadable-%E7%B1%BB)
        - ['close' 事件](#close-%E4%BA%8B%E4%BB%B6)
        - ['data' 事件](#data-%E4%BA%8B%E4%BB%B6)
        - ['end' 事件](#end-%E4%BA%8B%E4%BB%B6)
        - ['error' 事件](#error-%E4%BA%8B%E4%BB%B6)
        - ['readable' 事件](#readable-%E4%BA%8B%E4%BB%B6)
        - [readable.isPaused()](#readableispaused)
        - [readable.pause()](#readablepause)
        - [readable.pipe(destination[,options])](#readablepipedestinationoptions)
        - [readable.unpipe([destination])](#readableunpipedestination)
        - [readable.read([size])](#readablereadsize)
        - [readable.resume()](#readableresume)
        - [readable.setEncoding(encoding)](#readablesetencodingencoding)
        - [readable.unshift()](#readableunshift)
        - [readable.wrap(stream)](#readablewrapstream)
    - [Duplex（双工）流和 Transform（变换）流](#duplex%E5%8F%8C%E5%B7%A5%E6%B5%81%E5%92%8C-transform%E5%8F%98%E6%8D%A2%E6%B5%81)
      - [stream.Duplex类](#streamduplex%E7%B1%BB)
      - [stream.Transform类](#streamtransform%E7%B1%BB)
  - [面向流实现者的API](#%E9%9D%A2%E5%90%91%E6%B5%81%E5%AE%9E%E7%8E%B0%E8%80%85%E7%9A%84api)
    - [简化的构造方式](#%E7%AE%80%E5%8C%96%E7%9A%84%E6%9E%84%E9%80%A0%E6%96%B9%E5%BC%8F)
    - [实现可写流](#%E5%AE%9E%E7%8E%B0%E5%8F%AF%E5%86%99%E6%B5%81)
      - [构造器：new stream.Writable([options])](#%E6%9E%84%E9%80%A0%E5%99%A8new-streamwritableoptions)
      - [writable._write(chunk, encoding, callback)](#writable_writechunk-encoding-callback)
      - [writable._writev(chunks, callback)](#writable_writevchunks-callback)
      - [写入时的错误](#%E5%86%99%E5%85%A5%E6%97%B6%E7%9A%84%E9%94%99%E8%AF%AF)
      - [一个可写流例子](#%E4%B8%80%E4%B8%AA%E5%8F%AF%E5%86%99%E6%B5%81%E4%BE%8B%E5%AD%90)
    - [实现可读流](#%E5%AE%9E%E7%8E%B0%E5%8F%AF%E8%AF%BB%E6%B5%81)
      - [new stream.Readable([options])](#new-streamreadableoptions)
      - [readable._read(size)](#readable_readsize)
      - [readable.push(chunk[, encoding])](#readablepushchunk-encoding)
      - [读取时的错误](#%E8%AF%BB%E5%8F%96%E6%97%B6%E7%9A%84%E9%94%99%E8%AF%AF)
      - [计数流示例](#%E8%AE%A1%E6%95%B0%E6%B5%81%E7%A4%BA%E4%BE%8B)
    - [实现双工流](#%E5%AE%9E%E7%8E%B0%E5%8F%8C%E5%B7%A5%E6%B5%81)
      - [new Stream.Duplex(options)](#new-streamduplexoptions)
      - [一个双工流的例子](#%E4%B8%80%E4%B8%AA%E5%8F%8C%E5%B7%A5%E6%B5%81%E7%9A%84%E4%BE%8B%E5%AD%90)
      - [对象模式的双工流](#%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E5%8F%8C%E5%B7%A5%E6%B5%81)
    - [实现转换流](#%E5%AE%9E%E7%8E%B0%E8%BD%AC%E6%8D%A2%E6%B5%81)
      - [new stream.Transform([options])](#new-streamtransformoptions)
      - ['finish' 和 'end' 事件](#finish-%E5%92%8C-end-%E4%BA%8B%E4%BB%B6)
      - [transform._flush(callback)](#transform_flushcallback)
      - [transfrom._transfrom(chunk, encoding, callback)](#transfrom_transfromchunk-encoding-callback)
      - [Class: stream.PassThrough](#class-streampassthrough)
    - [补充内容](#%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Stream（流）

> 稳定性：**2** - 稳定

流是用于在 Node.js 中处理数据流的抽象接口。`Stream` 模块提供了一个基本的API，用于快速构建实现了流接口的对象。

Node.js 中提供了许多内置的流对象。例如，[对一个HTTP服务器的请求](#TODE)和[process.stdout](#TODE)都是流实例。

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

* [可读](#streamreadable-%E7%B1%BB) - 可以读取数据的流（例如：[fs.createReadStream()](#Tode)）。
* [可写](#%E5%8F%AF%E5%86%99%E6%B5%81writable) - 可以写入数据的流（例如：[fs.createWriteStream()](#Tode)）。
* [双工](#streamduplex%E7%B1%BB) - 可读可写的流（例如：[net.Socket](#Tode)）
* [变换](#streamtransform%E7%B1%BB) - 可以在写入和读写数据时修改或变换数据的双工流（例如：[zlib.createDeflate()](#Tode)）

### 对象模式

所有由 Node.js API 创建的流都只运行在字符串和 `Buffer` 对象上。但是，流是可以在其它 JavaScript 类型（除了 `null`，应为它在流中有特殊的用途）上运行的，这样的流被认为是在“对象模式”中操作的流。

创建流时，流实例将使用 `objectMode` 选项去切换到对象模式。不应该将现有的流切换到对象模式。


### 缓冲

无论是 [可写流](#streamwritable%E7%B1%BB) 还是 [可读流](#streamreadable-%E7%B1%BB)，它们都会在内部对象中缓存数据，这两个内部对象分别可以从 `writable._writableState.getBuffer()` 或 `readable._readableState.buffer` 中获取。

缓冲的数据量大小取决于传递到流构造函数种的 `highWaterMark` 选项。对于正常的流，`highWaterMark` 选项指定的是总字节数。对于在对象模式下操作的流，`higtWaterMark` 选项指定的是对象的总数。

实现调用 [steam.push(chunk)](#readablepushchunk-encoding) 时，数据将被缓冲当在可读流中。如果流的消费者没有调用 `steam.read()`，数据将一直保留在内部队列中，直到它被消费。

一旦内部用于读的缓冲区的总大小达到了由 `highWaterMark` 设定的阈值，则流将临时停止从底层资源读取数据，直到可以使用当前缓冲的数据（即流将停止调用内部的 `readable._read()` 方法去填充用于读的缓冲区）。

当重复调用 `writable.write(chunk)` 方法时，数据将被缓冲在可写流中。当内部用于写的缓冲区的总大小低于 `higWaterMark` 设定的阈值时，对 `writable.write()` 的调用将返回 `true`。一旦内部缓冲区的大小达到或超过 `highWaterMark` 阈值时，将返回 `false`。

流提供的 API，特别是 `stream.pipe()` 方法的主要目的是为了将数据的缓存控制在可接受的范围内，使得不同速度的资源和目标不会占满内存。

由于[双工流](#streamduplex%E7%B1%BB)和[变换流](#streamtransform%E7%B1%BB)都是可读和可写的，每种流都保持两个单独的内部缓冲区用于读取和写入，并允许一侧独立于另一个操作，同时保证数据流的正确及有效。例如，[net.Socket](#Tode)实例是一个[双工流](#streamduplex%E7%B1%BB)，其 Readable(可读) 端允许消耗从 socket(套接字) 接收的数据，并且其 Writable(可写) 端允许向 socket(套接字) 写数据。因为可能需要以比接收数据更快或更慢的速率将数据写入 socket(套接字) ，所以每一测操作（和缓冲）独立于另一侧是很重要的。


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

[可写流](#streamwritable%E7%B1%BB)（如实例中的 `res`）暴露一些方法，例如 `write()` 与 `end()` 可用于将数据写入到该流中。

[可读流](#streamreadable-%E7%B1%BB)使用[EventEmitter](../events/#class-eventemitter)的 API 在数据可用时通知应用程序代码读取流中的数据。可以通过多种方式从流中读取可用数据。

[可写流](#streamwritable%E7%B1%BB)和[可读流](#streamreadable-%E7%B1%BB)都以各种方式使用[EventEmitter](../events/#class-eventemitter)的API来传达流的当前状态。

[双工流](#streamduplex%E7%B1%BB)和[变换流](#streamtransform%E7%B1%BB)都是[可读](#streamreadable-%E7%B1%BB)和[可写](#streamwritable%E7%B1%BB)的；

向流写入数据或从流消费数据的应用不需要直接实现流接口，并且一般情况下并不需要调用到 `requrie('stream')`。

如果你正在自己的程序中实现流接口，请参阅[面向实现者的API](#%E9%9D%A2%E5%90%91%E6%B5%81%E5%AE%9E%E7%8E%B0%E8%80%85%E7%9A%84api)。

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

注意：这些实例中的一些实际上是[双工流](#streamduplex%E7%B1%BB)只是表现为[可写流](#%E5%8F%AF%E5%86%99%E6%B5%81writable)的形式。

所有的[可写流](#%E5%8F%AF%E5%86%99%E6%B5%81writable)都基于 `stream.Writable` 类实现。

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

如果调用 [stream.write(chunk)](#writablewritechunkencodingcallback) 返回 `false` (译者注：即内部数据缓冲区的大小超过阈值，数据无法写入)，`drain` 事件将在流允许写入更多的数据的时候触发。

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

`finish` 事件将会在调用 [stream.end()](#writableendchunkencodingcallback) 方法，并且缓冲区的所有数据都更新到底层系统中后触发。

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

* `src` <[stream.Readable](#streamreadable-%E7%B1%BB)> 被导流到该可写流的可读流。

当在可读流上调用 [stream.pipe()](#readablepipedestinationoptions) 方法并将当前可写流作为目标时，将会触发 `pipe` 事件。

``` javascript
const writer = getWritableStreamSomehow();
const reader = getReadableStreamSomehow();
writer.on('pipe', (src) => {
	console.error('something is piping into the writer');
	assert.equal(src, reader);
});
reader.pipe(writer);
```

##### 'unpipe'事件

* `src` <[stream.Readable](#streamreadable-%E7%B1%BB)> 被解除导流到该可写流的可读流。

当在可读流上调用 [stream.unpipe()](#readableunpipedestination) 方法并将当前可写流从目标集中解除时，将会触发 `unpipe` 事件。

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

`writable.cork()` 将强制要写入的数据进入缓存。当调用 [stream.uncork()](#writableuncork) 或 [stream.end()](#writableendchunkencodingcallback) 方法时才会从缓存中吐出并开始写入。

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

在 [stream.end()](#writableendchunkencodingcallback) 方法之后调用 [stream.write()](#writablewritechunkencodingcallback) 方法将抛出一个错误

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

`writable.setDefaultEncoding()` 方法用于设置一个[可写流](#streamwritable%E7%B1%BB)的默认 `encoding`。

##### writable.write(chunk[,encoding][,callback])

* `chunk` <String>|<Buffer> 写入的数据
* `encoding` <String> 编码，如果 `chunk` 是字符串将使用该编码格式
* `callback` <Function> 可选的当流完成时的回调函数
* Returns: <Bollean> 如果返回 `false` 表示流希望在等待 `drain` 事件发出后再继续调用代码去写入附加数据，否则返回 `true`。

`writable.write()` 方法将一些数据写入流中，并在数据完全处理后调用提供的回调函数。如果发生错误，则回调*可能会也可能不会*将错误作为第一个参数调用。为了检测写入错误，请监听 'error' 事件。

如果允许数据写入后内部缓冲区小于创建时配置的 `highWaterMark`，则返回 `true`，如果返回 `false`，则表示内部缓冲区暂时无法支持数据的继续写入，应该停止向流中写入数据的操作，直到发出 `drain` 事件。

当一个流不消耗时，对 `write()` 的调用将缓存 `chunk`，并返回 `false`。一旦所有当前缓冲的 `chunk` 被排空（由操作系统接收传递），将发出 `drain` 事件。建议一旦 `write()` 方法返回 `false`，在 'drani' 事件发出之前不再写入任何 `chunk`。当对未排空的流调用 `write()` 方法时，Node.js 将缓存所有写入的 `chunk`，直到最大内存使用率，此时它将被强制中止。甚至在被中止前，内存占满将导致垃圾收集器的性能变的糟糕和 RSS（通常不会释放会系统，即使在不再需要内存之后）。由于TCP套接字可能永远不会耗尽，如果远程对等体不读取数据，编写未排空的套接字可能导致可被远程利用的漏洞产生。

对于 [双工流](#streamduplex%E7%B1%BB) 来说在流不排出数据时写入数据尤其有问题，因为 `双工流` 被默认暂停，直到它们被管道化或者添加了 `data` 或 `readable` 事件后才会打开。

如果要写入的数据可以根据需要生成或提取，建议将配置为 [可读流](#streamreadable-%E7%B1%BB) 并使用 [stream.pipe()](#readablepipedestinationoptions) 方法。但是，如果优先使用 `write()` 的话，则可以使用 `drain` 事件来避免内存问题。

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

所有的[可读流](#streamreadable-%E7%B1%BB)都基于 `stream.Readable` 类实现。

#### 两种模式

可读流有两种有效的运行模式： 流动(flowing)和暂停(paused)。

当处在流动模式时，数据从底层系统自动读取，并通过 [EventEmitter](../events/#class-eventemitter) 接口使用事件尽可能快的提供给应用程序。

在暂停模式下，必须显示调用 [stream.read()](#readablereadsize) 方法以从流中读取数据块。

所有可读流都以暂停模式开始，但可以通过以下方式切换到流模式：

* 添加一个 ['data'](#data-%E4%BA%8B%E4%BB%B6) 事件处理器。
* 调用 [stream.resume()](#readableresume) 方法来开启数据流
* 调用 [stream.pipe()](#readablepipedestinationoptions) 方法将数据发送给一个[可写流](#streamwritable%E7%B1%BB)。

可读流可以使用以下方法切换为暂停模式：

* 如果没有管道目标，则可以通过调用 [stream.pause()](#readablepause) 方法切换。
* 如果有管道目标，则可以通过删除所有 ['data'](#data-%E4%BA%8B%E4%BB%B6) 事件处理程序，并通过调用 [stream.unpipe()](#readableunpipedestination) 方法删除所有管道目标。

要记住一个重要概念是，在没有提供消费或忽略该数据的机制之前，可读流是不会生成数据的。如果消费机制被禁用或移除，可读流将*尝试*停止生成数据。

*注意*：出于向后兼容的考虑，删除 ['data'](#data-%E4%BA%8B%E4%BB%B6) 事件处理程序**并不会**自动暂停流。此外，当有导流目标时，调用 [stream.pause()](#readablepause) 并不能保证流在那些目标排空并请求更多数据时维持暂停状态。

*注意*：如果[可读流](#streamreadable-%E7%B1%BB)切换到流动模式但没有可用的数据处理机制，该数据将丢失。例如，当在没有设置 `data` 事件处理程序的情况下调用 [readable.resume()](#readableresume) 时或者当从流中移除 `data` 事件处理程序时，就会出现数据丢失。

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

并不是所有的 [可读流](#streamreadable-%E7%B1%BB) 都会触发 `close` 事件。

##### 'data' 事件

* `chunk` <[Buffer](#TODE)> | <String> | <any> 数据块。对于不再对象模式下操作的流，块将是字符串或 Buffer。对于出于对象模式的流，块可以是处理 `null` 之外的任何 JavaScript 值。

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

实际上，`'readable'` 事件表示流的状态发生了改变：有新数据可用或这已经到达流的末端。在前一种情况下，调用 [stream.read()](#readablereadsize) 方法将返回可用的数据。在后一种情况下，调用 [stream.read()](#readablereadsize) 将返回 `null`。例如，下例中， foo.txt 是一个空文件：

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

* `destination` <[stream.Writable](#streamwritable%E7%B1%BB)> 写入数据的目标
* `options` <Object> 管道参数
	* `end` <Boolean> 当读取结束时终止写入，默认为 `true`

`readable.pipe()` 方法将 [可写流](#streamwritable%E7%B1%BB) 附加到当前的可读流上，使其自动切换到流动模式，并将其所有数据推送到附加的 [可写流](#streamwritable%E7%B1%BB) 中。该方法能自动控制流量以避免目标被快速读取的可读流所淹没。

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

默认情况下，当源数据可读流发出 ['end'](#end-%E4%BA%8B%E4%BB%B6) 事件时，目标可写流的 [stream.end()](#writableendchunkencodingcallback) 方法会被调用，以便目标可写流不再可写。要禁用此默认行为，可以将 `end` 选项设置为 `false`（译者注：{ end: false }），以保持目标可写流的开启状态。如下例所示：

``` javascript
reader.pipe(writer, { end: false });
reader.on('end', () => {
	writer.end('Goodbye\n');
});
```

警告：如果可读流再处理期间发生了错误，则目标可写流是不会自动关闭的。如果发生错误，需要手动关闭每个流，以防止内存泄漏。

注意：[process.stderr](#TODE) 和 [process.stdout](#TODE) 在进程结束前是不会关闭的，无论是否指定选项。

##### readable.unpipe([destination])

* `destination` <[stream.Writable](#streamwritable%E7%B1%BB)> 可选，指定解除导流的流。

`readable.unpipe()` 方法用于移除使用 [readable.pipe()](#readablepipedestinationoptions) 方法附加的可写流。

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
* Return <String> | <[Buffer](#TODE)> | <Null>

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

注意，在 ['end'](#end-%E4%BA%8B%E4%BB%B6) 事件触发后调用 [stream.read([size])](#readablereadsize) 将会返回 `null`，并且不会产生错误警告。

##### readable.resume()

* Returns: `this`

`readable.resume()` 方法将暂停状态的流切换到流动模式，使其继续发出 ['data'](#data-%E4%BA%8B%E4%BB%B6) 事件。

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

* `chunk` <[Buffer](#TODE)> | <String> 回读队列开头的数据块

该方法在某些情况下很有用，比如一个流正在被一个解析器消费，解析器需要“逆消费”某些刚从源中拉取出来的数据，以便流可以传递给其它消费者。

注意，`stream.unshift(chunk)` 不能在 ['end'](#end-%E4%BA%8B%E4%BB%B6) 事件触发后调用，否则将产生一个运行时错误。

如果你发现你必须在你的程序中频繁调用 [stream.unshift(chunk)](#TODE) ，请考虑实现一个变换流（Transform）作为替代。（详见[面向流实现者的 API](#%E9%9D%A2%E5%90%91%E6%B5%81%E5%AE%9E%E7%8E%B0%E8%80%85%E7%9A%84api)）

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

请注意，不像 [stream.push(chunk)](#readablepushchunk-encoding) 那样，`stream.unshift(chunk)` 不会通过重置流的内部读取状态结束读取过程。如果在读取过程（比如，在一个 [stream._read()](#readable_readsize) 内部实现一个自定义流）中调用 `readable.unshift()` 将导致意想不到的结果。在调用 `readable.unshift()` 后立即调用 [stream.push('')](#readablepushchunk-encoding) 会适当地重置读取状态，然而最好简单地避免在执行一个读出过程中调用 unshift() 。

##### readable.wrap(stream)

* `stream` <[Stream]>(#stream%E6%B5%81) 一个“旧式”可读流

Node.js v0.10 版本之前的流并未实现现今所有流 API。（更多信息详见[“兼容性”章节](#%E8%A1%A5%E5%85%85%E5%86%85%E5%AE%B9)。）

如果你正在使用一个早期版本的 Node.js 库，它会触发 ['data'](#data-%E4%BA%8B%E4%BB%B6) 事件并且有一个仅作查询用途的 [stream.pause()](#readablepause) 方法，那么你可以使用 `readable.wrap()` 方法来创建一个使用旧式流作为数据源的[可读流](#streamreadable-%E7%B1%BB)。

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

### Duplex（双工）流和 Transform（变换）流

#### stream.Duplex类

Duplex 流是同时实现了 [Readable](#streamreadable-%E7%B1%BB) 和 [Writable](#streamwritable%E7%B1%BB) 接口的流。

Duplex 流的实例包括：

* [TCP sockets](#TODE)
* [zilb 流](#TODE)
* [crypto 流](#TODE)

#### stream.Transform类

变换流（Transform streams） 是一种 [Duplex](#streamduplex%E7%B1%BB) 流。它的输出与输入是通过某种方式关联的。和所有 Duplex 流一样，变换流同时实现了 [Readable](#streamreadable-%E7%B1%BB) 和 [Writable](#streamwritable%E7%B1%BB) 接口。

Transfrom 流的实例包括：

* [zlib 流](#TODE)
* [crypto 流](#TODE)


## 面向流实现者的API

流模块API的设计使得我们可以很容易得通过 Javascript 原型继承去实现流。

首先，流实现者要先声明一个新的 Javascript 类，并继承四个基本流类之一（stream.Readable，stream.Writable，stream.Duplex，stream.Transform），并确保调用了相应的父类构造函数。

``` javascript
const Writable = require('stream').Writable;

class MyWritable extends Writable {
	constructor(options) {
		super(options);
	}
}
```

然后，新流类必须实现一个或多个特定方法，具体取决于要创建的流的类型，如下表所示：

用例 | 类 | 必须的方法
---- | -- | ----------
只读 | [Readable](#streamreadable-%E7%B1%BB) | [_read](#readable_readsize)
只写 | [Writable](#streamwritable%E7%B1%BB) | [_write](#writable_writechunk-encoding-callback), [_writev](#writable_writevchunks-callback)
读写 | [Duplex](#streamduplex%E7%B1%BB) | [_read](#readable_readsize), [_write](#writable_writechunk-encoding-callback), [_writev](#writable_writevchunks-callback)
对写入数据进行操作，然后读取结果 | [Transfrom](#streamtransform%E7%B1%BB) | [_transfrom](#transfrom_transfromchunk-encoding-callback), [_flush](#transform_flushcallback)

*注意：在你的流实现代码中，绝对不要调用 [面向消费者的API](#%E9%9D%A2%E5%90%91%E6%B5%81%E6%B6%88%E8%B4%B9%E8%80%85%E7%9A%84api) 中描述的方法。否则，可能会导致在使用你实现的流接口的过程中产生不良的副作用。*

### 简化的构造方式

在比较简单的情况下，可以不依赖继承来实现一个新的流。可以通过直接创建 `stream.Writable`、`stream.Readable`、`stream.Duplex`、`stream.Transfrom` 对象的实例，并传递适当的方法作为构造函数的选项来实现。

例如：

``` javascript
const Writable = require('stream').Writable;

const myWritable = new Writable({
	write(chunk, encoding, callback) {
		// ...
	}
});
```

### 实现可写流

扩展 `stream.Writable` 类去实现 [Writable](#streamwritable%E7%B1%BB) 流。

自定义可写流必须调用新的 `stream.Writable([options])`构造函数并实现 `writable._write()` 方法。 也可以实现 `writable._writev()` 方法。

#### 构造器：new stream.Writable([options])

* `options` <Object>
	* `highWaterMark` <Number> [stream.write()](#writablewritechunkencodingcallback) 开始返回 `false` 时的 Buffer 大小。 默认为16384（16 kb），或对于支持对象模式的流为16。
	* `decodeStrings` <Boolean> 在传递给 [stream._write()](#writable_writechunk-encoding-callback) 前是否解码字符串为 Buffer。默认为 `true`。
	* `objectMode` <Boolean> [stream.wirte(anyObj)](#writablewritechunkencodingcallback) 是否是有效操作（译者注：即是否支持对象模式），如果设置，则可以写入任意 Javascript 值，而不是限定为 `Buffer` / `String` 数据。默认为 `false`.
	* `write` <Function> 用于实现 [stream._write()](#writable_writechunk-encoding-callback) 方法。
	* `writev` <Function> 用于实现 [stream._writev()](#writable_writevchunks-callback) 方法。

例如：

``` javascript
const Writable = require('stream').Writable;

class MyWritable extends Writable {
	constructor(options) {
		// Calls the strean.Writable() constructor
		super(options);
	}
}
```

或者，使用pre-ES6风格的构造函数：

``` javascript
const Writable = require('stream').Writable;
const util = require('util');

function MyWritable(options) {
	if (!(this instanceof MyWritable)) {
		return new MyWritable(options);
	}
	Writable.call(this, options);
}
util.inherits(MyWritable, Writable);
```

或者，使用简化的构造方式：

``` javascript
const Writable = require('stream').Writable;

const myWritable = new Wirtable({
	write(chunk, encoding, callback) {
		// ...
	}
	writev(chunk, callback) {
		// ...
	}
});
```

#### writable._write(chunk, encoding, callback)

* `chunk` <Buffer> | <String> 要写入的块，*默认*为 Buffer，除非 `decodeStrings` 设置为 `false`。
* `encoding` <String> 如果块是一个字符串，则 `encoding` 是该字符串的字符编码。 如果块是 Buffer，或者如果流以对象模式操作，则会忽略该编码。
* `callback` <Function> 当为所提供的块完成处理时，调用此函数（可选择使用错误参数）。

所有可写流实现必须提供一个 [writable._write()](#writable_writechunk-encoding-callback) 方法来将数据发送到底层资源。

*注意：*Transform（变换）流已自行提供了 [writable._write()](#writable_writechunk-encoding-callback) 方法

*注意：*此函数**不能直接由应用程序代码调用**。 它应该由子类实现，并且仅由内部 Writable 类方法调用。

当写入操作成功或失败并抛出错误时会调用 `callback` 函数，如果操作失败，传递给 `callback` 的第一个参数将是 `Error` 对象，如果操作成功，第一个参数将是 `null`。

需要特別注意的一点是：在 `writable._write()` 被调用到 `callback` 被调用的这段时间内调用 `writable.write()` 方法，数据将会被缓存起来。一旦回调被调用，流将发出 ["drain"](#drain%E4%BA%8B%E4%BB%B6)事件。 如果实现的流需要能够一次处理多个数据块，那么应该实现 `writable._writev()` 方法。

如果构造函数选项中设定了 `decodeStrings` 标志，则 `chunk` 可能会是字符串而不是 Buffer，并且 `encoding` 表明了字符串的格式。这种设计是为了支持对某些字符串数据编码提供优化处理的实现。如果您没有明确地将 `decodeStrings` 选项设定为 `false`，那么您可以安全地忽略 `encoding` 参数，并假定 `chunk` 总是一个 Buffer。

这是一个带有下划线前缀的方法，因为它是在类内部定义的，并且不应该由用户程序直接调用。


#### writable._writev(chunks, callback)

* `chunks` <Array> 要写入的所有块。每个块具有以下的格式：{ chunk: ..., encoding: ... }
* `callback` <Function> 当为所提供的块完成处理时，调用此函数（可选择使用错误参数）。

*注意：*此函数**不能直接由应用程序代码调用**。 它应该由子类实现，并且仅由内部 Writable 类方法调用。

需要实现一次处理多个数据块的流中，除了实现必要的 `writable._write()` 方法，还可以选择实现 `writable._writev()` 方法。该方法将使用当前写入缓存队列中的所有数据来调用。

这是一个带有下划线前缀的方法，因为它是在类内部定义的，并且不应该由用户程序直接调用。

#### 写入时的错误

建议在处理 `writable._write()` 和 `writable._writev()` 方法期间发生的错误通过调用回调并将错误作为第一个参数进行报告。 这将导致 Writable 发出一个 `'error'` 事件。 从 `writable._write()` 内部抛出错误可能会导致意外和不一致的行为，具体取决于如何使用流。 使用回调可确保一致和可预测的错误处理。

``` javascript
const Writable = require('stream').Writable;

const myWritable = new Writable({
	write(chunk, encoding, callback) {
		if (chunk.toString().indexOf('a') >= 0) {
			callback(new Error('chunk is invalid'));
		} else {
			callback();
		}
	}
});
```

#### 一个可写流例子

下面展示了一个相当简单的（有点无意义）自定义可写流实现。 虽然这个特定的可写流实例没有任何实际的特殊用处，但该示例说明了自定义可写流实例的每个必需元素：

``` javascript
const Writable = require('stream').Writable;

class MyWritable extends Writable {
	constructor(options) {
		super(options);
	}

	_write(chunk, encoding, callback) {
		if (chunk.toString().indexOf('a') >= 0) {
			callback(new Error('chunk is invalid'));
		} else {
			callback();
		}
	}
}
```


### 实现可读流

扩展 `stream.Readable` 类去实现 [Readable](#streamreadable-%E7%B1%BB) 流。

自定义可写流必须调用新的 `stream.Readable([options])`构造函数并实现 `writable._read()` 方法。

#### new stream.Readable([options])

* `options` <Object>
	* `highWaterMark` <Number> 停止从底层资源读取能够存储在内部缓冲区的最大字节数。 默认为16384（16 kb），或对于支持对象模式的流为16。
	* `encoding` <String> 如果指定，则缓冲区将使用指定的编码字符串解码。默认为 `null`。
	* `objectMode` <Boolean> 否支持对象模式，如果设置，则意味着 [stream.read(n)](#readablereadsize) 返回单个值而不是一个大小为 n 的 Buffer。默认为 `false`。
	* `read` <Function> 用于实现 [stream._read()](#readable_readsize) 方法。

例如：

``` javascript
const Readable = require('stream').Readable;

class MyReadable extends Readable {
	constructor(options) {
		// Calls the stream.Readable(options) constructor
		super(options);
	}
}
```

或者，使用pre-ES6风格的构造函数：

``` javascript
const Readable = require('stream').Readable;
const util = require('util');

function MyReadable(options) {
	if (!(this instanceof MyReadable)) {
		return new MyReadable(options);
	}
	Readable.call(this, options);
}
util.inherits(MyReadable, Readable);
```

或者，使用简化的构造方式：

``` javascript
const Readable = require('stream').Readable;

const myReadable = new Readable({
	read(size) {
		// ...
	}
});
```

#### readable._read(size)

* `size` <Number> 异步读取的字节数

*注意：*此函数**不能直接由应用程序代码调用**。 它应该由子类实现，并且仅由内部 Readable 类方法调用。

所有可读流实现必须提供一个 `readable._read()` 方法的实现，以从底层资源获取数据。

当调用 `readable._read()` 时，如果从资源获取的数据可用，实现（译者注：_read方法的实现）应该通过调用 [this.push(dataChunk)](#readablepushchunk-encoding) 方法将数据推入到读取队列中。`_read()` 应该继续从资源读取并推送数据，直到 `readable.push()` 返回 `false`。

*注意：*一旦调用 `readable._read()` 方法，直到 [readable.push()](#readablepushchunk-encoding) 方法被调用之前将不能被再次调用。

`size` 参数仅供参考。对于只有 `read` 返回数据的操作的实现（译者注：指仅有 `_read` 方法的流实现），可以使用 `size` 参数来确定要提取多少数据。其它的流实现可以忽略这个参数，只要能提供数据，能调用。不需要“等待” `size` 大小的数据可用才去调用 [readable.push(chunk)](#readablepushchunk-encoding) 方法。

这是一个带有下划线前缀的方法，因为它是在类内部定义的，并且不应该由用户程序直接调用。

#### readable.push(chunk[, encoding])

* `chunk` <Buffer> | <null> | <String> 要推入读取队列的数据块
* `encoding` <String> String 类型的数据块的编码。必须是有效的 Buffer 编码，例如：'urf8'或'ascii'。
* Returns <Boolean> 返回是否可以继续推送数据块

当 `chunk` 是 Buffer 或字符串时，数据块将被添加到内部队列以供流的用户使用。 将块传递为 `null` 表示流的结束（EOF），之后不能写入更多的数据。

当 Readable 流在暂停状态下操作时，在发出 ['readable'](#readable-%E4%BA%8B%E4%BB%B6) 事件时，通过调用 [readable.read()](#readablereadsize) 方法可以读取通过 `readable.push()` 方法添加的数据。

当 Readable 流在流动状态下时，通过发出 `data` 事件来传递通过 `readable.push()` 添加的数据。

`readable.push()` 被设计成尽可能地灵活。例如，你可以包裹有某种暂停/恢复机制的低级资源和一个数据回调。在这种情况下，你可以通过这样做来包裹低级资源对象：

``` javascript
// source is an object with readStop() and readStart() methods,
// and an `ondata` member that gets called when it has data, and
// an `onend` member that gets called when the data is over.

class SourceWrapper extends Readable {
	constructor(options) {
		super(options);

		this._source = getLowlevelSourceObject();

		// Every time there's data, push it into the internal buffer.
		this._source.ondata = (chunk) => {
			// if push() returns false, then stop reading from source
			if (this.push(chunk))
				this._source.readStop();
		};

		// When the source ends, push the EOF-signaling `null` chunk
		this._source.onend(() => {
			this.push(null);
		});
	}

	// _read will be called when the stream wants to pull more data in
	// the advisory size argument is ignored in this case.
	_read(size) {
		this._source.readStart();
	}
}
```

*注意：* `readable.push()` 方法只能由实现的 Readable
 程序调用，并且只能从 `readable._read()` 方法中调用。


#### 读取时的错误

建议在处理 `readable._read()` 方法期间发生的错误使用 `'error'` 事件发出，而不是抛出。 从 `readable._read()` 内部抛出错误可能会导致意外和不一致的行为，具体取决于流是以流模式还是暂停模式运行。 使用 `'error'` 事件可确保一致和可预测的错误处理。

``` javascript
const Readable = require('stream').Readable;

const myReadable = new Readable({
	read(size) {
		if (let err = checkSomeErrorCondition()) {
			process.nextTick(() => this.emit('error', err));
			return;
		}

		// do some work
	}
});
```

#### 计数流示例

以下是可读流的基本示例，其以升序排列从1到1,000,000的数字，然后结束。

``` javascript
const Readable = require('stream').Readable;

class Counter extends Readable {
	constructor(opt) {
		super(opt);
		this._max = 1000000;
		this._index = 1;
	}

	_read() {
		var i = this._index++;
		if (i > thie._max)
			this.push(null);
		else {
			var str = '' + i;
			var buf = Buffer.form(str, 'ascii');
			this.push(buf);
		}
	}
}
```


### 实现双工流

双工流是基于 [Readable](#streamreadable-%E7%B1%BB) 和 [Writable](#streamwritable%E7%B1%BB) 实现的流，例如 TCP 套接字连接。

因为 Javascript 不具备多原型继承能力，所以 Duplex 流的实现是基于 `stream.Duplex`类（而不是基于 `stream.Readable`类 和 `stream.Writable` 类）。

*注意：*`stream.Duplex` 类原型继承自 `stream.Readable`和 `stream.Writable` 组合寄生类，但 instanceof 将正常工作为两个基类，因为在 `stream.Writable` 中重写 [Symbol.hasInstance](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance)。

自定义双工流必须调用`new stream.Duplex([options])` 构造函数并实现 `readable._read()` 和 `writable._write()` 方法。

#### new Stream.Duplex(options)

* `options` <Object> 传递给 `Readable` 和 `Writable` 构造函数。同时还有以下字段：
	* `allowHalfOpen` <Boolean> 默认为 `true`。如果设置为 `false`，那么当写入结束后流将自动结束读取，反之亦然。
	* `readableObjectMode` <Boolean> 默认为 `false`。用于设置流读取端的 `objectMode` 。如果读取端 `objectMode` 设置为 `true`，该字段将无效。
	* `writableObjectMode` <Boolean> 默认为 `false`。用于设置流写入端的 `objectMode` 。如果写入端 `objectMode` 设置为 `true`，该字段将无效。

例子如下：

``` javascript
const Duplex = require('stream').Duplex;

class MyDuple extends Duplex {
	constructor(options) {
		super(options);
	}
}
```

或者，使用pre-ES6风格的构造函数：

``` javascript
const Duplex = require('stream').Duplex;
const util = require('util');

function MyDuplex(options) {
	if (!(this instanceof MyDuple)) {
		return new MyDuplex(options);
	}
	Duplex.call(this, options);
}
util.inherits(MyDuplex, Duplex);
```

或者，使用简化的构造方式：

``` javascript
const Duplex = require('stream').Duplex;

const myDuplex = new Duplex({
	read(size) {
		// ...
	},
	write(chunk, encoding, callback) {
		// ...
	}
});
```

#### 一个双工流的例子

下面举例说明了一个简单的例子，它包含一个假想的低层源对象的双工流，数据可以被写入其中，并且可以从中读取数据，尽管使用的是与 Node.js 流不兼容的API。 下面举例说明了Duplex流的一个简单示例，它通过可读接口缓冲传入的写入数据，并通过 [Readable](#streamreadable-%E7%B1%BB) 接口读回。

``` javascript
const Duplex = require('stream').Duplex;
const kSource = Symbol('source');

class MyDuplex extends Duplex {
	constructor(source, options) {
		super(options);
		this[kSource] = source;
	}

	_write(chunk, encoding, callback) {
		// The underlying source only deals with strings
		if (Buffer.isBuffer(chunk))
			chunk = chunk.toString();
		this[kSource].wirteSomeData(chunk);
		callback();
	}

	_read(size) {
		this[kSource].fetchSomeData(size, (data, encoding) => {
			this.push(Buffer.form(data, encoding));
		});
	}
}
```

双工流的最重要的方面是可读和可写端独立于彼此操作，尽管在单个对象实例内共存。

#### 对象模式的双工流

对于双工流，`objectMode` 可以分别使用 `readableObjectMode` 和 `writableObjectMode` 选项为 Readable 或 Writable 专门设置。

在下面的例子中，创建了一个新的转换流（一种双工流的类型），它具有对象模式的可写端，在读取端接收 Javascript 值并将其转换为十六进制的字符串。

``` javascript
const Transform = require('stream').Transform;

// All Transform streams are also Duplex Streams
const myTransform = new Transform({
	writableObjectMode: true,

	transform(chunk, encoding, callback) {
		// Coerce the chunk to a number if necessary
		chunk |= 0;

		// Transform the chunk into something else.
		const data = chunk.toString(16);

		// Push the data onto the readable queue.
		callback(null, '0'.repeat(data.length % 2) + data);
	}
});

myTransform.setEncoding('ascii');
myTransform.on('data', (chunk) => console.log(chunk));

myTransform.write(1);
// Prints: 01
myTransform.write(10);
// Prints: 0a
myTransform.write(100);
// Prints: 64
```


### 实现转换流

[Transform](#streamtransform%E7%B1%BB) 流是一种将输入以某种方式计算后输出的 [Duplex](#streamduplex%E7%B1%BB) 流。比如用于压缩的 [zlib](#TODE) 流或用于加解密的 [crypto](#TODE) 流。

*注意：*它并不要求输入和输出的数据块需要相同大小、相同块数或同时到达。例如：一个 Hash 流只会在输入结束时产生一个数据块的输出；一个 `zlib` 流会产生比输入小得多或大得多的输出。

转换流是基于 `stream.Transform` 扩展实现的流。

`stream.Transform` 类在原型上继承自 `stream.Duplex` 并实现其自己版本的 `writable._write()` 和 `readable._read()` 方法。 自定义变换流必须实现 [transform._transform()](#transfrom_transfromchunk-encoding-callback) 方法，并且还可以实现 [transform._flush()](#transform_flushcallback) 方法。

注意：使用转换流时必须小心，因为写入流的数据可能导致流的可写端在可读端的输出未被消耗时暂停。

#### new stream.Transform([options])

* `options` <Object> 传递给 `Readable` 和 `Writable` 构造函数。同时还有以下字段：
	* `transfrom` <Function> 用于实现 [stream._transform()](#transfrom_transfromchunk-encoding-callback) 方法
	* `flush` <Function> 用于实现 [stream._flush()](#transform_flushcallback) 方法

例子如下：

``` javascript
const Transform = require('stream').Transfrom;

class MyTransfrom extends Transform {
	constructor(options) {
		super(options);
	}
}
```

或者，使用pre-ES6风格的构造函数：

``` javascript
const Transform = require('stream').Transform;
const util = require('util');

function MyTransform(options) {
	if (!(this instanceof MyDuple)) {
		return new MyTransform(options);
	}
	Transform.call(this, options);
}
util.inherits(MyTransform, Transform);
```

或者，使用简化的构造方式：

``` javascript
const Transform = require('stream').Transform;

const myTransform = new Transform({
	transform(chunk, encoding, callback) {
		// ...
	}
});
```

#### 'finish' 和 'end' 事件

['finish'](#finish%E4%BA%8B%E4%BB%B6) 和 ['end'](#end-%E4%BA%8B%E4%BB%B6) 事件分别来自 `stream.Writable` 和 `stream.Readable` 类。 在调用 [stream.end()](#writableendchunkencodingcallback) 后，通过 [stream._transform()](#transfrom_transfromchunk-encoding-callback) 处理所有块，发出 `'finish'` 事件。 `'end'` 事件在所有数据输出后发出，这在 [transform._flush()](#transform_flushcallback) 的回调被调用之后发生。

#### transform._flush(callback)

* `callback` <Function> 刷新剩余数据时调用的回调函数（可选地包含错误参数和数据）。

*注意：*此函数**不能直接由应用程序代码调用**。 它应该由子类实现，并且仅由内部 Readable 类方法调用。

在一些情况下，变换操作可能需要在流的末端附加额外的数据。例如：`zlib` 压缩流在末端存储一定量用于压缩输出的内部状态。但是，在流结束时，需要刷新附加数据，使得压缩数据得以完整。

自定义 [Transfrom](#streamtransform%E7%B1%BB) 流可以实现 `transform._flush()` 方法。 当没有更多的要被消费的写入数据时，在发出 `'end'` 事件之前发出信号通知可读流的结束时，将调用该方法。

在 `transfrom._flush()` 方法中，可以调用任意调用 `readable.push()` 方法。但当数据刷新完成时，必须调用 `callback` 回调。

`transform._flush()` 方法以下划线为前缀，因为它是定义它的类的内部，并且不应该由用户程序直接调用。

#### transfrom._transfrom(chunk, encoding, callback)

* `chunk` <Buffer> | <String> 要进行转换的数据块。除非 `decodeStrings` 选项设置为 `false`。否则将一直是 Buffer。
* `encoding` <String> 如果块是字符串，那么这是编码类型。 如果 `chunk` 是一个 Buffer，将忽略它。
* `callback` <Function> 在处理提供的块之后调用的回调函数（可选地具有错误参数和数据）。

*注意：* **此函数不能直接由应用程序代码调用。** 它应该由子类实现，并且仅由内部可读类方法调用。

所有转换流实现都必须提供一个 `_transform()` 方法来接受输入并产生输出。 `transform._transform()` 用于处理正在写入的数据并计算输出，然后使用 `readable.push()` 方法将输出传递到可读区域。

`transform.push()` 方法可以用于输出数据，调用的次数取决于作为块的结果要输出多少。

可能没有从任何给定的输入数据块产生输出。

只有当当前块被完全消耗时，才必须调用回调函数。 如果处理输入时出错，则传递给回调的第一个参数必须是一个 `Error` 对象，否则为空。 如果第二个参数传递给回调，它将被转发到 `readable.push()` 方法。 换句话说，以下是等价的：

``` javascript
transfrom.prototype._transform = function(data, encoding, callback) {
	this.push(data);
	callback();
};

transfrom.prototype._transfrom = function(data, encoding, callback) {
	callback(null, data);
};
```

`transform._transform()` 方法以下划线为前缀，因为它是定义它的类的内部，并且不应该由用户程序直接调用。

#### Class: stream.PassThrough

`stream.PassThrough` 类是一个简单实现的转换流，简单地将输入字节传递到输出。 它的目的主要是用于示例和测试，但有一些用例中将 `stream.PassThrough` 作为一种构建块的新型流。


### 补充内容

对旧版本 Node.js 的兼容

> 译者注：兼容这一块的东西不考虑翻译，有兴趣的可以查看 [官方文档](https://nodejs.org/dist/latest-v7.x/docs/api/stream.html#stream_additional_notes)