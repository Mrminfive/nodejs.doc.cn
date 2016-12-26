<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [util(工具集)](#util%E5%B7%A5%E5%85%B7%E9%9B%86)
  - [util.debuglog(section)](#utildebuglogsection)
  - [util.deprecate(function, string)](#utildeprecatefunction-string)
  - [util.format(format[, ...args])](#utilformatformat-args)
  - [util.inherits(constructor, superConstructor)](#utilinheritsconstructor-superconstructor)
  - [util.inspect(object[, options])](#utilinspectobject-options)
    - [定制 `util.inspect` 颜色](#%E5%AE%9A%E5%88%B6-utilinspect-%E9%A2%9C%E8%89%B2)
    - [在对象上定制 `inspect()` 函数](#%E5%9C%A8%E5%AF%B9%E8%B1%A1%E4%B8%8A%E5%AE%9A%E5%88%B6-inspect-%E5%87%BD%E6%95%B0)
    - [util.inspect.defaultOptions](#utilinspectdefaultoptions)
    - [util.inspect.custom](#utilinspectcustom)
  - [已废弃的API](#%E5%B7%B2%E5%BA%9F%E5%BC%83%E7%9A%84api)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# util(工具集)

> 稳定性：**2** - 稳定

util模块主要是用于 Node.js 内部需求的API。在程序或模块开发中同样也可以使用：

``` javascript
const util = require('util');
```

## util.debuglog(section)

* `section` <String> 标识为其创建调试日志功能的应用程序部分的字符串。
* 返回 <Function> 日志函数

`util.debuglog()` 方法用于创建一个函数，根据 NODE_DEBUG 环境变量，有条件地将调试消息写入 `stderr`。如果 `section` 名称出现在该环境变量的值中，则返回的函数的操作类似与 [console.error()](../console/#consoleerrordataargs)。如果没有，那么返回的函数是一个空函数。

例如：

``` javascript
const util = require('util');
const debuglog = util.debuglog('foo');

debuglog('hello from foo [%d]', 123);
```

如果程序在环境中运行时带有 `NODE_DEBUG=foo`，那么会有这样的输出：

``` javascript
FOO 3245: hello from foo [123]
```

这里的 `3245` 是进程ID。如果不是与环境变量设置一起运行，它是不会打印出任何东西的。

可以在 NODE_DEBUG 环境变量中设置多个 `section` 名称，用逗号分隔。例如：`NODE_DEBUG = fs, net, tls`。

## util.deprecate(function, string)

`util.deprecate()` 方法可以为给定的 `fucntion` 或类进行包装，标记为废弃状态（译者注：但仍可以使用）。

``` javascript
const util = require('util');

exports.puts = util.deprecate(function() {
	for(var i = 0, len = arguments.length; i < len; i++) {
		process.stdout.write(arguments[i] + '\n');
	}
}, 'util.puts: Use console.log instead');
```

当调用 `util.deprecate()` 时，将返回一个函数，它将使用 `process.on('warning')` 事件发出 `DeprecationWarning`（事件将在当前事件队列的末尾调用）。默认情况下，在第一次调用时，此警告将发出且只打印一次到 `stderr`。发出警告后，调用包装函数。

如果使用了 `--no-deprecation` 或 `--no-warnings` 命令行标记，或者在第一个弃用警告之前将 `process.noDeprecation` 属性设置为 `true`。`util.deprecate()` 方法将不会执行任何操作。

如果设置了 `--trace-deprecation` 或 `--trace-warnings` 命令行标记，或者 `process.traceDeprecation` 属性设置为 `true`, 则在第一次调用弃用的函数时，会向 `stderr` 打印警告和堆栈跟踪。

如果设置了 `--throw-depercation` 命令行标记，或者将 `process.throwDeprecation` 属性设置为 `true`，那么在调用已弃用的函数时将抛出异常。

`--throw-deprecation` 和 `process.throwDeprecation` 优先级高于 `--trace-deprecation` 和 `process.traceDeprecation`。

## util.format(format[, ...args])

* `format` <String> 类似于 `printf` 格式的字符串。

`util.format()` 方法使用第一个参数作为类似 `printf` 的格式返回一个格式化的字符串。

第一个参数是一个包含零个或多个占位符标记的字符串。每隔占位符令牌都将替换为相应参数的转换值。支持的占位符包括：

* `%s` - 字符串。
* `%d` - 数值（包括整型和浮点数）
* `%j` - JSON如果参数包含循环引用，则将其替换为字符串 `'[Circular]'`。
* `%%` - 百分号（'%'）。这不消耗参数。

如果占位符没有相应的参数，占位符不被替换。

``` javascript
util.format('%s:%s', 'foo');
// Returns: 'foo:%s'
```

如果传递给 `util.format()` 方法的参数多余占位符的数量，则额外的参数被强制转换为字符串（对于对象和符号，使用 [util.inspect()](#utilinspectobject-options) 处理），然后使用空格分隔后连接到返回的字符串中。

``` javascript
uti;l.format('%s:%s', 'foo', 'bar', 'baz'); // 'foo:bar baz'
```

如果第一个参数不是格式化字符串，那么 `util.format()` 方法将返回一个字符串，它是用空格分隔的所有参数的串联。每个参数使用 `util.inspect()` 转换为字符串。

``` javascript
util.format(1, 2, 3); // '1 2 3'
```

## util.inherits(constructor, superConstructor)

注意：不建议使用 `util.inherits()`。请使用 ES6 的类并使用 `extend` 关键字获得语言级继承支持。还有注意，两种方式在[语义上是不兼容](https://github.com/nodejs/node/issues/4179)的。

* `constructor` <Function>
* `superConstructor` <Function>

将原型的方法从一个[构造函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor)继承到另一个构造函数。`constructor` 的原型将被设置为从 `superConstructor` 创建的一个新对象。

为了方便，`superConstructor` 可以通过 `constructor.super_` 属性访问。

``` javascript
const util = require('util');
const EventEmitter = require('events');

function MyStream() {
	EventEmitter.call(this);
}

util.inherits(MyStream, EventEmitter);

MyStream.prototype.write = function(data) {
	this.emit('data', data);
};

const stream = new MyStream();

console.log(stream instanceof EventEmitter); // true
console.log(MyStream.super_ === EventEmitter); // true

stream.on('data', (data) => {
	console.log(`Received data: ${data}`);
});

stream.write('It works!'); // Received data: "It works!"
```

使用 ES6 `class` 和 `extends` 的例子：

``` javascript
const EventEmitter = require('events');

class MyStream extends EventEmitter {
	constructor() {
		super();
	}

	write(data) {
		this.emit('data', data);
	}
}

const stream = new MyStream();

stream.on('data', (data) => {
	console.log(`Received data: ${data}`);
})

stream.write('With ES6');
``` 

## util.inspect(object[, options])

* `object` <any> 任何 JavaScript 原始值或对象。
* `options` <Object>
	* `showHidden` <boolean> 如果设置为 `true` ，`object` 的不可枚举和 symbol 属性都将展示出来。默认为 `false`。
	* `depth` <number> 指定格式化对象时递归的次数。这对于检查大型复杂对象很有用。默认为 `2` 。设置为 `null` 可无限递归。
	* `colors` <boolean> 如果设置为 `true`，将使用 ANSI 颜色代码样式输出。默认为 `false`。颜色允许自定义，请参阅[定制 util.inspect 颜色](#%E5%AE%9A%E5%88%B6-utilinspect-%E9%A2%9C%E8%89%B2)。
	* `customInspect` <boolean> 如果设置为 `false`，`object` 上的自定义函数 `inspect(depth, opts)` 被检查时不会被调用。默认为 `true`。
	* `showProxy` <boolean> 如果设置为 `true`，作为 [Proxy代理对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 的对象和函数将自动检查并显示其目标和处理程序对象。默认为 `false`。
	* `maxArrayLength` <number> 指定要格式化的数组和 `TypedArray（类数组）` 元素的最大数目。默认为 `100`，设置为 `null` 显示所有数组元素。设置为 `0` 或负值，不显示数组元素。
	* `breakLength` <number> 对象的键在多行间分隔的长度。设置为无穷大可以将对象格式化为单行。为了兼容旧版本，默认设置为 `60`。

`util.inspect()` 方法返回表示 `object` 的字符串，主要用于调试。可以传递额外的配置选项来改变格式化字符串的方式。

检查 util 对象的所有属性的示例：

``` javascript
const util = require('util');

console.log(util.inspect(util, { showHidden: true, depth: null }));
```

值可以提供自己的自定义 inspect(depth, opts) 函数，当调用这些函数时，在递归检查中接收当前深度，以及传递给 util.inspect() 的 options 对象。

``` javascript
// 译者注：
const util = require('util');

let obj = {
	a: 1,
	inspect(depth, opts) {
		console.log(depth, opts);
		return 'lalal'
	}
};

console.log(util.inspect(obj));
```

### 定制 `util.inspect` 颜色

通过 `util.inspect.styles` 和 `util.inspect.colors` 对象来全局定制 `util.inspect` 的颜色输出（如果启用）。

`util.inspect.styles` 每一种风格都会映射为 `util.inspect.colors` 的一种颜色。

默认风格及代表的高亮样式：

* `number` - `yellow` 黄色
* `boolean` - `yellow` 黄色
* `string` - `green` 绿色
* `date` - `magenta` 品红
* `regexp` - `red` 红色
* `null` - `bold` 粗体
* `undefined` - `grey` 灰色
* `special` - `cyan` 青色（仅应用于当前函数）
* `name` - （没有样式）

预定义的颜色代码是：`white`、`grey`、`black`、`blue`、`cyan`、`green`、`magenta`、`red` 和 `yellow`。同样有 `bold`、`italic`、`underline` 和 `inverse` 代码。

### 在对象上定制 `inspect()` 函数

对象也可以定义自己的 `[util.inspect.custom](depth, otps)` (或者等价的 `inspect(depth, opts)` 函数)，`util.inspect()` 方法将调用并使用它检查对象给出的结果。

``` javascript
const util = require('util');

class Box {
	constructor(value) {
		this.value = value;
	}

	inspect(depth, options) {
		if (depth < 0) {
			return options.stylize('[Box]', 'special');
		}

		const newOptions = Object.assign({}, options, {
			depth: options.depth === null ? null : options.depth - 1
		});

		// Five space padding because that's the size of "Box< ".
		const padding = ' '.repeat(5);
		const inner = util.inspect(this.value, newOptions).replace(/\n/g, '\n' + padding);
		return options.stylize('Box', 'special') + '<' + inner + '>';
	}
}

const box = new Box(true);

util.inspect(box);
// Returns: "Box< true >"
```

自定义 `[util.inspect.custom](depth, opts)` 函数通常返回一个字符串，但也可以返回任何类型的值，交由 `util.inspect()` 继续格式化。

``` javascript
const util = require('util');

const obj = { foo: 'this will not show up in the inspect() output' };
obj[util.inspect.custom] = function(depth) {
	return { bar: 'baz' };
};

util.inspect(obj);
// Returns: "{ bar: 'baz' }"
```

可以通过在对象上设置 `inspect(depth, opts)` 方法来提供定制检查方法：

``` javascript
const util = require('util');

const obj = { foo: 'this will not show up in the inspect() output' };
obj.inspect = function(depth) {
	return { bar: 'baz' };
};

util.inspect(obj);
// Returns: "{ bar: 'baz' }"
```

### util.inspect.defaultOptions

可以通过 `defaultOptions` 值定制 `util.inspect` 使用的默认选项。这对像 `console.log` 或 `util.format` 这种隐式调用 `util.inspect()` 的函数是非常有用的。它应该设置为一个或多个有效 `util.inspect()` 选项的对象。同时还支持直接设置选项属性。

``` javascript
const util = require('util');
const arr = Array(101);

console.log(arr); // logs the truncated array
util.inspect.defaultOptions.maxArrayLength = null;
console.log(arr); // logs the full array
```

### util.inspect.custom

可用于声明自定义检查的 symbol ，请参阅 [在对象上定制 `inspect()` 函数](#%E5%9C%A8%E5%AF%B9%E8%B1%A1%E4%B8%8A%E5%AE%9A%E5%88%B6-inspect-%E5%87%BD%E6%95%B0)。

## 已废弃的API

> 已废弃的API，在这就不翻译下去了，有兴趣的可以直接查看[官方文档](https://nodejs.org/dist/latest-v7.x/docs/api/util.html#util_deprecated_apis)。