<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [console（控制台）](#console%E6%8E%A7%E5%88%B6%E5%8F%B0)
  - [异步与同步的控制台](#%E5%BC%82%E6%AD%A5%E4%B8%8E%E5%90%8C%E6%AD%A5%E7%9A%84%E6%8E%A7%E5%88%B6%E5%8F%B0)
  - [Console类](#console%E7%B1%BB)
    - [new Console(stdout[, stderr])](#new-consolestdout-stderr)
    - [console.assert(value[,message][,...args])](#consoleassertvaluemessageargs)
    - [console.dir(obj[,options])](#consoledirobjoptions)
    - [console.warn([data][,...args])](#consolewarndataargs)
    - [console.error([data][,...args])](#consoleerrordataargs)
    - [console.info([data][,...args])](#consoleinfodataargs)
    - [consolo.log([data][,...args])](#consolologdataargs)
    - [console.time(label)](#consoletimelabel)
    - [console.timeEnd(label)](#consoletimeendlabel)
    - [console.trace(message[,...args])](#consoletracemessageargs)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# console（控制台）

> 稳定性：**2** - 稳定

该 `console` 模块提供了类似于 Web 浏览器提供的 JavaScript 控制台机制的一个简单调试控制台

该模块导出两个特定组件：

+ 一个 `Console` 类，包含 `console.log()`, `console.error`, `console.warn()` 等方法，并且可以写入任何 Node.js 流中
+ 一个是全局性的 `console` 实例，用于写入 `stdout` 和 `stderr`。应为这个对象是全局性的，所以它可以在不使用 `require('console')` 的情况下直接调用。

例如：全局 `console` 的使用

``` javascript
console.log('hello world');
// Prints: hello world, to stdout
console.log('hello %s', 'world');
// Prints: hello world, to stdout
console.error(new Error('Whoops, something bad happened'));
// Prints: [Error: Whoops, something bad happened], to stderr

const name = 'Will Robinson';
console.warn(`Danger ${name}! Danger!`);
// Prints: Danger Will Robinson! Danger!, to stderr
```

``` javascript
const out = getStreamSomehow();
const err = getStreamSomehow();
// 译者注：getStreamSomehow方法获取 Node.js 流
const out = process.stdout;
const err = process.stderr;
const myConsole = new console.Console(out, err);

myConsole.log('hello world');
// Prints: hello world, to out
myConsole.log('hello $s', 'world');
// Prints: hello world, to out
myConsole.error(new Error('Whoops, something bad happened'));
// Prints: [Error: Whoops, something bad happened], to err

const name = 'Will Robinson';
myConsole.warn(`Danger ${name}! Danger!`)
// Prints: Danger Will Robinson! Danger!, to err
```

虽然 `Console` 类的 API 是根据浏览器的 `console` 对象设计的，但 Node.js 中的 `Console` 并没有完全复制浏览器中的功能。

## 异步与同步的控制台

`console` 函数通常是异步的，除非目标对象是一个文件。磁盘读写速度块并且操作系统通常使用回写缓存，写入阻塞应该是一个非常罕见的情况，但它还是有可能发生的。

此外，作为对 MAC OS X 中极小的（1kb）的缓存大小限制的一种解决方案，在 MAC OS X 上输出到 TTY（终端）时，控制台功能会遭到阻塞。这是为了防止 `stdout` 和 `stderr` 交叉在一起。

## Console类

`Console` 类可用于创建具有可配置的输出流的简单记录器，而且可以使用 `require('console').Console` 或 `console.Console` 中的任何一种方式访问。

``` javascript
const Console = require('console').Console;
const Console = console.Console;
```

### new Console(stdout[, stderr])

基于一个或两个可写流实例创建一个新的 `Console` 对象。`stdout` 是一个用来打印日志或输出信息的可写流。`stderr` 是一个用来输出警告或错误的可些流。如果 `stderr` 无法正常输出（或者未定义），警告和错误将会被输出到 `stdout` 中。

``` javascript
const output = fs.createWriteStream('./stdout.log');
const errorOutput = fs.createWriteStream('./stderr.log');
// custom simple logger
const logger = new Console(output, errorOutput);
// use it like console
var count = 5;
logger.log('count: %d', count);
// in stdout.log: count 5
```

全局 `console` 是一个特殊的 `Console`，它的输出被发送到 [process.stdout](#) 和 [process.stderr](#)。它等同于调用:

``` javascript
new Console(process.stdout, process.stderr);
```

### console.assert(value[,message][,...args])

一个简单的断言测试用来验证 `value` 是否为真。如果不是，将抛出一个 `AssertionError` 错误。如果提供了错误信息 `message` 参数，则使用 [util.format()](#) 方法格式化后作为错误信息输出。

``` javascript
console.assert(true, 'does nothing');
// OK
console.assert(false, 'Whoops %s', 'didn\'t work');
// AssertionError: Whoops didn't work
```

*注：该 `console.assert()` 方法在 Node.js 中的实现和[浏览器中可用](https://developer.mozilla.org/zh-CN/docs/Web/API/console/assert)的 `console.assert()` 方法实现是不同的。*

说明白点，在浏览器中当 `console.assert()` 方法接受到一个值为假断言（assertion）的时候，会向控制台输出传入的内容，但是并不会中断代码的执行，而在 Node.js 中一个值为假的断言将会导致一个 `AssertionError` 被抛出，使得代码执行被打断。

可以通过扩展 Node.js 的 `console` 并覆盖 `console.assert()` 方法来实现浏览器中实现的类似功能。

以下例子中，会创建一个简单的模块，该模块扩展了 Node.js 中的 `console` 并覆盖了 `console` 中的默认行为。

``` javascript
'use strict';

// Creates a simple extension of console with a
// new impl for assert without monkey-patching
const myConsole = Object.setPrototypeOf({
    assert(assertion, message, ...args) {
        try {
            console.assert(assertion, message, ...args);
        } catch (err) {
            console.error(err.stack);
        }
    }
}, console);

module.exports = myConsole;
```

这个模块可以用来直接替换内置控制台

``` javascript
const console = require('./myConsole');
console.assert(false, 'this message will print, but no error thrown');
console.log('this will also print');
```

### console.dir(obj[,options])

对 `obj` 参数使用 `util.inspect()` 方法并将结果字符串打印到 `stdout` 流中。此方法会绕过任何在 `obj` 上定义的 `inspect()` 函数。可选参数 `options` 对象可以传递一些用于格式化字符串的内容。

* `showHidden` - 如果为 `true`，那么对象的不可枚举和 `Symbol` 属性也将会被显示。默认为 `false`。
* `depth` - 指定 [util.inspect()](#) 函数格式化对象是递归多少次。这对遍历大型复杂的对象很有用。默认设置为 `2`。可以设置为 `null` 实现无限递归。
* `colors` - 如果为 `true`，那么输出结果将使用 ANSI 颜色风格。默认为 `false`。颜色可定制，详情请参照 [定制util.inspect()颜色](#)。

### console.warn([data][,...args])

`console.warn()` 函数是 [console.error()](#consoleerrordataargs) 函数的一个别名

### console.error([data][,...args])

使用换行符将信息打印到 `stderr` 流中，允许传递多个参数，第一个参数作为主要信息，其余参数作为类似与 [print(3)](http://man7.org/linux/man-pages/man3/printf.3.html) 中的替换值（这些参数都会传递给 [util.format()](#1) 做处理）。

``` javascript
const code = 5;
console.error('error #%d', code);
// Prints: error #5, to stderr
console.error('error', code);
// Prints: error 5, to stderr
```

如果在第一个字符串参数中没有找到占位符（如：%d），那么 [util.inspect()](#1) 会在所有参数上调用，并将结果拼合在一起。详情请见 [util.format()](#1) 。

### console.info([data][,...args])

`console.info()` 函数是 [console.log()](#consolologdataargs) 函数的一个别名。

### consolo.log([data][,...args])

使用换行符将信息打印到 `stdout` 流中，允许传递多个参数，第一个参数作为主要信息，其余参数作为类似与 [print(3)](http://man7.org/linux/man-pages/man3/printf.3.html) 中的替换值（这些参数都会传递给 [util.format()](#1) 做处理）。

``` javascript
const code = 5;
console.log('count: %d', code);
// Prints: count: 5, to stdout
console.log('count', code);
// Prints: count 5, to stdout
```

### console.time(label)

启动一个计时器用于计算一个操作的持续时间。`label` 为计时器的唯一标识。可以使用同样的 `label` 来调用 [console.timeEnd()](#) 方法来停止计时器，并将以毫秒为单位的持续时间输出到 `stdout` 流中。计时器的持续时间精确到亚毫秒。

### console.timeEnd(label)

停止之前用 [console.time](#) 启动的计时器并将结果打印到 `stdout` 流中。

``` javascript
console.time('100-elements');
for(var i = 0; i < 100; i++) {
    ;
}
// console.itme('100-elements');
// 译者注：重复启动相同标识的计时器将会覆盖前一个。
console.timeEnd('100-elements');
// Prints: 100-elements: 225.438ms
// 译者注：持续时间以实际机器为主
```

*注意：从 Node.js v6.0.0 开始，`console.timeEnd()` 会将计时器删除，以避免内存泄漏。在旧版本上，计时器会保留，这会允许对同一个标识的计时器多次调用 `console.timeEnd()` 方法。此功能并不是预期的，不再受到支持。*

### console.trace(message[,...args])

通过 `stderr` 流打印字符串 `Trace: `，以及后面用 [util.format()](#) 格式化的消息和代码中当前位置的堆栈跟踪。

``` javascript
console.trace('Show me');
// Prints: (stack trace will vary based on where trace is called)
//  Trace: Show me
//    at repl:2:9
//    at REPLServer.defaultEval (repl.js:248:27)
//    at bound (domain.js:287:14)
//    at REPLServer.runBound [as eval] (domain.js:300:12)
//    at REPLServer.<anonymous> (repl.js:412:12)
//    at emitOne (events.js:82:20)
//    at REPLServer.emit (events.js:169:7)
//    at REPLServer.Interface._onLine (readline.js:210:10)
//    at REPLServer.Interface._line (readline.js:549:8)
//    at REPLServer.Interface._ttyWrite (readline.js:826:14)
```