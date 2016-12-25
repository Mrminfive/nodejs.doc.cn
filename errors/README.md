<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Error(错误)](#error%E9%94%99%E8%AF%AF)
  - [错误的冒泡和捕获](#%E9%94%99%E8%AF%AF%E7%9A%84%E5%86%92%E6%B3%A1%E5%92%8C%E6%8D%95%E8%8E%B7)
  - [Node.js 风格的回调](#nodejs-%E9%A3%8E%E6%A0%BC%E7%9A%84%E5%9B%9E%E8%B0%83)
  - [Error类](#error%E7%B1%BB)
    - [new Error(message)](#new-errormessage)
    - [Error.captureStackTrace(targetObject[, constructorOpt])](#errorcapturestacktracetargetobject-constructoropt)
    - [Error.stackTraceLimit](#errorstacktracelimit)
      - [error.message](#errormessage)
      - [error.stack](#errorstack)
  - [Class：RangeError](#classrangeerror)
  - [Class: ReferenceError](#class-referenceerror)
  - [Class: SyntaxError](#class-syntaxerror)
  - [Class: TypeError](#class-typeerror)
  - [异常与错误](#%E5%BC%82%E5%B8%B8%E4%B8%8E%E9%94%99%E8%AF%AF)
  - [系统错误](#%E7%B3%BB%E7%BB%9F%E9%94%99%E8%AF%AF)
    - [Class：System Error](#classsystem-error)
      - [error.code](#errorcode)
      - [error.errno](#errorerrno)
      - [error.syscall](#errorsyscall)
    - [通用的系统错误](#%E9%80%9A%E7%94%A8%E7%9A%84%E7%B3%BB%E7%BB%9F%E9%94%99%E8%AF%AF)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Error(错误)

在 Node.js 中有4中类型的错误：

* 标准的 Javascript 错误，如：
	* <EvalError>：当一个 `eval()` 执行失败时抛出。
	* <SyntaxError>：当检测到错误的 JavaScript 语法格式时抛出。
	* <RangeError>：当一个值不在预期范围内时抛出。
	* <ReferenceError>：当使用未定义的变量时抛出。
	* <TypeError>：当传递错误类型的参数时抛出。
	* <URIError>：当全局 URI 处理函数被错误使用时抛出。
* 由于底层操作系统的限制引发的错误。例如：试图打开一个不存在的文件，试图向一个已关闭的套接字（socket）发送数据等。
* 由底层代码触发的 User-specified（用户指定） 错误。
* 当 Node.js 检测到一个不应该发生的特殊逻辑冲突时，将会触发断言错误这一种特殊的错误。这些错误通常由 `assert` 模块触发。

Node.js 中提供的所有错误都实例化或继承于标准的 [JavaScript Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error) 类，并保证提供**至少**该类上的可用属性。

## 错误的冒泡和捕获

在程序运行时，Node.js 支持几种错误冒泡的处理机制。如果报告和处理这些错误完全取决于错误的类型和被调用的 API 的风格。

所有的 JavaScript 错误都将做异常处理，立即产生，并使用标准的 JavaScript `throw` 机制抛出一个错误。这些错误可以使用 JavaScript 语言所提供的[try / catch 结构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)进行处理。

``` javascript
// Throws with a ReferenceError because az is undefined
try {
	const m =1;
	const n = m + z;
} catch (err) {
	// Handle the error here.
}
```

使用 JavaScript提供的 `throw` 机制将会引发一个异常，*必须*使用 `try/catch`来处理或者 Node.js 立即退出进程。

有少数例外，*同步的API*（任何不接受回调函数的阻塞方法，例如 [fs.readFileSync](#Todo)）会使用 `throw` 报告错误。

异步 API 中的错误可以以多种方式进行报告。

* 大多数异步方法都接受一个 `callback` 函数，并将 `Error` 对象做为该函数的第一个参数传递。如果第一个参数不是 `null`，是一个 `Error` 对象实例，就应该对发生的错误进行处理。

``` javascript
const fs = require('fs');
fs.readFile('a file that does not exist', (err, data) => {
	if (err) {
		console.error('There was an error readubf the file!', err);
		return ;
	}
	// Otherwise handle the data
});
```

* 当 `EventEmitter` 对象调用一个异步方法时，错误会被分发到该对象的 `error` 事件上。

``` javascript
const net = require('net');
const connection = net.connect('localhost');

// Adding an 'error' event handler to a stream:
connection.on('error', (err) => {
	// If the connection is reset by the server, or if it can't
	// connect at all, or on any sort of error encountered by 
	// ther connection, ther error will be sent here.
});

connection.pipe(process.stdout);
```

* 在 Node.js 的异步API中，仍有少数使用 `throw` 机制引发错误，必须使用 `try/catch` 处理。这类方法没有一份完整的列表。请参阅各类方法的文档以确定合适的错误处理机制。

使用 `'error'` 事件机制最常见的是 [stream-based](#Todo) 和 [event emitter-based](#Todo) API。它们本身就代表这随时间的一系列一异步操作（相对与单一操作，可能有效也可能无效）。

对于*所有 `EventEmitter` 对象*而言，如果没有提供 `error` 事件来处理程序错误，错误将会被抛出，导致 Node.js 进程报告一个未处理的异常并随即崩溃，除非适当的使用 [dimain 模块](#Todo) 或注册了处理程序的 [process.on('uncaughtException')](#Todo) 事件。

``` javascript
const EventEmitter = require('events');
const ee = new EventEmitter();

setImmediate(() => {
	// This will crash the process because no 'error' event
	// handler has been added.
	ee.emit('error', new Error('this will crash'));
});
```

在调用代码已经退出后抛出的错误，无法用 `try/catch` 截获。

开发者必须查阅各类方法的文档，以明确在错误发生是的错误冒泡方式。

## Node.js 风格的回调

大多数 Node.js API 暴露出来的异步方法都遵循一种被称为"Node.js风格回调"的惯用模式。通过这种模式，可以将一个回调函数通过参数的方式传入。当操作完成或引发错误时，回掉函数被调用，`Error` 对象（如果有的话）作为第一个参数传递给回调函数，如果没有的话，回调函数第一个参数将为 `null`。

``` javascript
const fs = require('fs');

function nodeStyleCallback(err, data) {
	if (err) {
		console.error('There was an error', err);
		return;
	}
	console.log(data);
}

fs.readFile('/some/file/that/does-not-exist', nodeStyleCallback);
fs.readFile('/some/file/that/does-exist', nodeStyleCallback);
```

JavaScript 的 `try/catch` 机制**不能**用于拦截由异步API生成的错误。对于初学者的一个常见错误为：试图在一个 Node.js 风格回调函数中用 `throw`：

``` javascript
// This will not work:
const fs =require('fs');

try {
	fs.readFile('/some/file/that/does-not-exist', (err, data) => {
		// mistaken assumption: throwing here...
		if (err) {
			throw err;
		}
	});
} catch(err) {
	// This will not catch the throw!
	console.log(err);
}
``` 

这并没有什么用，因为 `fs.readFile()` 是一个异步调用的回调函数。当回调函数被调用时，这些代码（包括 `try { } catch(err) { }` 区域）就已经退出。在大对数案例中，抛出回调函数中的错误会**引起 Node.js 进程崩溃**。如果 [domains](#Todo) 被启用，或已注册了 `process.on('uncaughtException')` 事件，那么这样的错误是可以被拦截的。

## Error类

一个普通的 JavaScript Error 对象，不表述发生错误的具体原因。`error` 对象会捕获“堆栈跟踪”，详细的说明是实例化 `error` 的是在代码中的那个位置，并且允许提供错误的文本描述。

Node.js 中所产生的所有错误，包括系统和 JavaScript 错误，要么是实例对象，要么继承至 `Error` 类。

### new Error(message)

创建一个新的 `Error` 对象，并将 `error.message` 属性设置来提供的文本消息。如果将一个对象传递为 `message`，则通过调用 `message.toString()` 生成文本信息。`error.stack` 属性会表明 `new error` 在代码中的位置。堆栈跟踪是根据[V8的堆栈跟踪API](#todo)生成的。堆栈跟踪仅跟踪到（a）*同步代码执行的开始*，或（b）由属性 `Error.stackTraceLimit` 给出的帧数中的最小值。

### Error.captureStackTrace(targetObject[, constructorOpt])

为 `targetObject` 创建一个 `.stack` 属性，在其被访问时返回 `Error.captureStackTrace()` 在代码中被调用的堆栈跟踪位置。

``` javascript
const myObject = {};
Error.captureStackTrace(myObject);
myObject.stack // similar to `new Error().stack`
// 译者注：
// console.log(myObject.stack);
```

堆栈跟踪的第一行会是前缀被替换为 `ErrorType: message` 的 `targetObject.toString()` 的调用结果。
> 译者注：试验过，与文档所说不符，求各位看官指点一二。

可选的 `constructorOpt` 接受一个函数作为其参数。如果提供了该参数，则 `constructorOpt` 以前包括自身在内的全部栈帧都会被其生成的堆栈跟踪省略。

`constructorOpt` 用在向最终用户隐藏错误生成的具体细节时非常有用。例如：

``` javascript
function MyError() {
	Eror.captureStackTrace(this, MyError);
}

// Without passong MyError to captureStackTrace, the MyError
// frame would show up in the .stack property. By passing
// the constructor, we omit that frame and all frames above it.
new MyError().stack

// 译者注：试验过代码无效，另附有效代码, 求看官指点
// function MyError() {
//    Eror.captureStackTrace(this, function() {});
// }
```

### Error.stackTraceLimit

`Error.stackTraceLimit` 属性用于设置堆栈跟踪（通过 `new Error().stack` 或 `Error.captureStackTrace(obj)` 收集的）的堆栈帧数

默认值为 `10` ，可以设置为任何有效的 JavaScript 数字。配置这个值后会影响到之后的堆栈跟踪。

如果设置为一个非数字值或者设置为一个负数，堆栈跟踪不会捕获任何帧。

译者注：

``` javascript
function MyError() {
	Error.stackTraceLimit = -1;
	Error.captureStackTrace(this);
}

console.log(new MyError().stack); // 将输出一个空的堆栈跟踪

function MyError() {
	Error.stackTraceLimit = {};
	Error.captureStackTrace(this);
}

console.log(new MyError().stack); // 将输出undefined，即不会做任何堆栈跟踪配置。
```

#### error.message

返回由 `new Error(message)` 设置的用来描述错误的信息字符串。该字符串将会在堆栈跟踪的第一行显示，然而，在 `Error` 对象创建完后再去修改这个属性*可能不会改变*堆栈跟踪第一行的提示信息（译者注：暂时没遇到过）。

``` javascript
const err = new Error('The message');
console.log(err.message);
// Prints: The message
```

#### error.stack

返回一个 `Error` 实例在代码中的位置的字符串（即堆栈跟踪）。

例如：

``` javascript
Error: Things keep happening!
	at /home/gbusy/file.js:525:2
	at Frobnicator.refrobulate (/home/gbusey/business-logic.js:424:21)
	at Actor.<anonymous> (/home/gbusey/actors.js:400:8)
	at increaseSynergy (/home/gbusey/actors.js:701:6)
```

第一行会被格式化为 `<error class name>: <error message>` ，并且随后跟着一系列栈帧（每一行都会以 "at " 开头）。每一帧都描述了一个代码中导致错误生成的调用点。V8 引擎会尝试显示每个函数的名称（变量名、函数名或对象的方法名），但偶尔也可能找不到一个合适的名称。如果 V8 引擎没法确定一个函数的名称，在该栈帧中只会显示仅有的位置信息。否则，在位置信息的附近将会显示已确定的函数名。

注意，这对**仅**由 JavaScript 函数产生的栈帧非常有用。例如，在自身是一个 JavaScript 函数的情况下，通过同步执行一个名为 cheetahify 的 C++ 插件时，代表 cheetahify 回调的栈帧将不会出现在当前的堆栈跟踪里：

``` javascript
const cheetahify = require('./native-binding.node');

function makeFaster() {
	// cheetahify *synchronously* calls speedy.
	cheetahify(function speedy() {
		thorw new Error('oh no!');
	});
}

makeFaster(); // will throw:
// /home/gbusey/file.js:6
//     throw new Error('oh no!');
//           ^
// Error: oh no!
//     at speedy (/home/gbusey/file.js:6:11)
//     at makeFaster (/home/gbusey/file.js:5:3)
//     at Object.<anonymous> (/home/gbusey/file.js:10:1)
//     at Module._compile (module.js:456:26)
//     at Object.Module._extensions..js (module.js:474:10)
//     at Module.load (module.js:356:32)
//     at Function.Module._load (module.js:312:12)
//     at Function.Module.runMain (module.js:497:10)
//     at startup (node.js:119:16)
//     at node.js:906:3
```

其中的位置信息会以下形式出现：

* `native`: 如果栈帧产生自 V8 引擎内部（比如，[].forEach）。
* `plain-filename.js:line:column`: 如果栈帧产生自 Node.js 内部。
* `/absolute/path/to/file.js:line:column`: 如果栈帧产生自用户程序或其依赖。

代表堆栈跟踪的字符串是在访问 `error.stack` 属性时才被生成的。

堆栈跟踪捕获的栈帧的数量是由 `Error.stackTraceLimit` 或当前事件循环可用的栈帧数量中的最小值界定。

系统级的错误由已传参的 `Error` 实例产生，详见[此处](#%E7%B3%BB%E7%BB%9F%E9%94%99%E8%AF%AF)。


## Class：RangeError

`Error` 的一个子类，表示提供的参数无法被设置或不在函数可接受范围内，无论是在一个数值范围外还是在函数给定的参数选项集合外，均用此类型错误表示。

例如：

``` javascript
require('net').connect(-1);
// throws RangeError, port should be > 0 && < 65536
```

## Class: ReferenceError

`Error` 的一个子类，表示企图访问一个未定义的变量。这些错误通常表示代码中的错别字或者程序异常。

虽然客户端代码可能会产生和传播这些错误，但在实践中，只有 V8 引擎会这么做。

``` javascript
doesNotExist;
// throws ReferenceError, doesNotExist is not a variable in this program.
```

`ReferenceError` 实例会带有一个 `error.arguments` 属性，它的值是一个包含单个元素的数组：表示没有定义的变量的字符串。

``` javascript
const assert = require('assert');

try {
	doesNotExist;
} catch(err) {
	assert(err.arguments[0], 'doesNotExist');
	// 译者注：试验过err.arguments为undefined，这点无法理解，求指点
}
```

除非一个应用程序是动态生成并运行的代码，否则 `ReferenceError` 实例会始终被视为在代码或其依赖中的错误。

## Class: SyntaxError

`Error` 的一个子类，表示当前程序含有无效的 JavaScript 代码。这些错误只会产生和传播代码的评测结果。代码评测结果可能产生至 `eval`, `Function`, `require` 或[vm](#Todo)。这些错误几乎都表示程序无法运行。

``` javascript
try {
	require('vm').runInThisContext('binary ! isNotOk');
} catch(err) {
	// err will be a SyntaxError
}
```

## Class: TypeError

`Error` 的一个子类，表示提供的参数的类型不被允许。例如，将一个函数传递给一个需要字符串的参数时会产生一个 `TypeError` 。

``` javascript
require('url').parse(() => {});
// throws TypeError, since it expected a string
```

Node.js 会生成并以一种参数验证的形式*立即*抛出 `TypeError` 实例。

## 异常与错误

JavaScript 异常通常是作为无效操作的结果或作为 `throw` 语句的目标抛出的值。虽然不需要这些值是 `Error` 或继承至 `Error` 类的实例，但 Node.js 或 JavaScript 运行时抛出的所有异常都将继承至 `Error`。

一些异常在 JavaScript 层是*无法恢复*的。这种异常总是会导致 Node.js 进程崩溃。例如：`assert()` 检查或在 C++层中调用 `abort()`。

## 系统错误

当程序运行环境中发生异常时会产生系统错误。通常，这些是当应用程序违反操作系统约束（例如尝试读取不存在的文件或用户没有足够的权限）时发生的操作错误。

系统错误通常在系统级别的代码中生成：通过在大多数 Unix 上运行 `man 2 intro` 或 `man 3 errno`，可以获得错误代码及其含义的详细列表；或[在线查看](http://man7.org/linux/man-pages/man3/errno.3.html)

在 Node.js 中，用添加了新属性的 `Error` 扩充对象表示系统错误。

### Class：System Error

#### error.code

返回一个表示错误代码的字符串，总是以 `E` 开头，后跟着大写字母序列，可以在 `man 2 intro` 中直接引用。

#### error.errno

返回与负面错误代码相对应的数字，可以在 `man 2 intro` 中引用该数字。例如：`ENOENT` 错误的 errno 为 `-2`，因为 `ENOENT` 的错误代码为 `2`。

#### error.syscall

返回描述[系统调用](http://man7.org/linux/man-pages/man2/syscall.2.html)失败的字符串。


### 通用的系统错误

此列表并不完整，但列举了编写 Node.js 程序时遇到的许多常见的系统错误。点击[这里](http://man7.org/linux/man-pages/man3/errno.3.html)查看详细列表。

* `EACCES` (权限不足)：试图以权限不足的方式访问文件。
* `EADDRINUSE` (地址被占用)：由于本地系统上的另一个服务器已占用该地址，因此尝试服务器（[net](#Todo)、[http](#Todo)或[https](#Todo)绑定到本地地址失败）。
* `ECONNREFUSED` (连接被拒绝)：由于目标计算机主动拒绝连接，因此无法建立连接。这通常是由于尝试连接到的外部主机上的服务不运行导致。
* `ECONNRESET` (连接被对方重置)：连接的目标强制关闭连接。这通常是由于超时或重新启动而导致远程套接字（socket）上的连接丢失。在 [http](#Todo) 和 [net](#Todo) 模块时会经常遇到。
* `EEXIST` (文件已存在)：一个要求目标不存在的文件操作的目标是一个已存在的文件。
* `EISDIR` (是一个目录)：一个对文件的操作，但给定的路径是一个目录。
* `EMFILE` (在系统中打开了过多的文件)：已达到系统中[文件描述符](https://en.wikipedia.org/wiki/File_descriptor)允许的最大数量，并且另外一个描述符的请求在至少关闭其中一个之前不能被满足。这在一次并行打开多个文件时会遇到，尤其是在那些在进程中限制了一个较低的文件描述符数量的操作系统上（在 particular 和 OS X 中）。为了破除这个限制，请在与运行 Node.js 进程的同一 shell 中运行 `ulimit -n 2048` 。
* `ENOENT` (无此文件或目录)：通常是由[文件操作](#Todo)引起的，这表明指定的路径组合不存在--在给定的路径上无法找到任何实体（文件或目录）。
* `ENOTEMPTY` (目录非空)：一个需要空目录操作的目标目录是一个实体。通常是由 [fs.unlink](#Todo) 引起的。
* `EPERM` (操作不被允许)：试图执行需要提升权限的操作。
* `EPIPE` (管道损坏)：没有进程读取数据时写入 `pipe`、`socket` 或 `FIFO`。在 [net](#Todo) 和 [http](#Todo) 层经常遇到，表明在远端的流（stream）准备写入前被关闭。
* `ETIMEDOUT` (操作超时)：由于连接方在一段时间后并没有做出合适的响应导致的连接或发送的请求失败。在 [http](#Todo) 或 [net](#Todo) 中经常遇到--往往标志着 `socket.end()` 没有被合理的调用。