<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Modules(模块)](#modules%E6%A8%A1%E5%9D%97)
  - [访问主模块](#%E8%AE%BF%E9%97%AE%E4%B8%BB%E6%A8%A1%E5%9D%97)
  - [核心模块（内置模块）](#%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%E5%86%85%E7%BD%AE%E6%A8%A1%E5%9D%97)
  - [循环](#%E5%BE%AA%E7%8E%AF)
  - [文件模块](#%E6%96%87%E4%BB%B6%E6%A8%A1%E5%9D%97)
  - [文件夹做模块](#%E6%96%87%E4%BB%B6%E5%A4%B9%E5%81%9A%E6%A8%A1%E5%9D%97)
  - [从 `node_modules` 文件夹中加载](#%E4%BB%8E-node_modules-%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%AD%E5%8A%A0%E8%BD%BD)
  - [从全局文件夹加载](#%E4%BB%8E%E5%85%A8%E5%B1%80%E6%96%87%E4%BB%B6%E5%A4%B9%E5%8A%A0%E8%BD%BD)
  - [模块封装](#%E6%A8%A1%E5%9D%97%E5%B0%81%E8%A3%85)
  - [缓存](#%E7%BC%93%E5%AD%98)
  - [`module` 对象](#module-%E5%AF%B9%E8%B1%A1)
    - [module.children](#modulechildren)
    - [module.exports](#moduleexports)
      - [exports 快捷方式](#exports-%E5%BF%AB%E6%8D%B7%E6%96%B9%E5%BC%8F)
    - [module.filename](#modulefilename)
    - [module.id](#moduleid)
    - [module.loaded](#moduleloaded)
    - [moudle.parent](#moudleparent)
    - [module.require(ID)](#modulerequireid)
  - [总言之](#%E6%80%BB%E8%A8%80%E4%B9%8B)
  - [附录](#%E9%99%84%E5%BD%95)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



# Modules(模块)

> 稳定性：**3** - 锁定

Node.js 拥有一个简单的模块加载系统。在 Node.js 中，文件与模块是一一对应的（每一个文件都会被视为一个模块）。

示例如下，假设有一个名为 `foo.js` 的文件：

``` javascript
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```

在 `foo.js` 中的第一行代码为加载与 `foo.js` 在相同目录下的 `circle.js` 模块。

`circle.js` 的内容如下：

``` javascript
const PI = Math.PI;

exports.area = (r) => PI * r * r;

exports.circumference = (r) => 2 * PI * r;
```

`circle.js` 模块导出 `area()` 和 `circumference()` 两个函数。为了将函数和对象添加到你的主模块中，你需要将它们添加到特殊的 `exports` 对象中。

模块中的局部变量均为私有，应为模块内容被包裹在由 Node.js 提供的函数中（见[模块封装](#%E6%A8%A1%E5%9D%97%E5%B0%81%E8%A3%85)）。在本例中，变量 `PI` 就是 `circle.js` 私有的。

如果你希望你的模块最终导出的是一个函数（如构造函数）或者一次性导出一个完整的对象，而不需要多次输出属性，可以使用 `module.exports` 代替 `exports`。

下面，我将使用从 `square` 模块中导出的构造函数。

``` javascript
const square = require('./square.js');
let mySquare = square(2);
console.log(`The area of my square is ${mySquare.area()}`);
```

`square.js` 中是这样定义 `square` 模块的：

``` javascript
// assigning to exports will not modify module, must use module.exports
module.exports = (width) => {
	return {
		area: () => width * width
	};
}
```

模块系统在 `require("module")` 中实现。

## 访问主模块

当 Node.js 直接运行一个文件时，`require.main` 就会被设置为该文件的输出 `module`。这意味着，你可以在测试中判断一个文件是否直接运行。

``` javascript
require.main === module
```

对于 `foo.js` 而言，通过 `node foo.js` 运行结果将为 `true`, 通过 `require('./foo.js')` 运行结果将为 `false`。

另外，`module` 提供了一个 `filename` 属性（通常情况下等同于 `__filename`），可以通过 `require.main.filename` 获取当前程序的入口。


## 核心模块（内置模块）

Node.js 中内置了一些用二进制编译好的模块。这些模块在本文档的其它地方由详细的介绍。

核心模块定义在 Node.js 源代码的 `lib/` 目录下。

`require()` 总是会优先加载核心模块。例如，`require('http')` 总是会返回内置的 HTTP 模块，不管是否存在同名的文件。

## 循环

当循环调用 `require()` 时，一个模块可能在返回时并未执行。

考虑这样的情况：

`a.js`:

``` javascript
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

`b.js`

``` javascript
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

`main.js`:

``` javascript
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

首先 `main.js` 加载 `a.js`，接着 `a.js` 又去加载 `b.js`。这个时候，`b.js` 又尝试去加载 `a.js`。为了防止形成无限循环，`a.js` 会返回一个**未完成复制**（译者注：即执行到会形成无限循环的代码之前，之后的代码完全忽略）的对象给 `b.js`。然后 `b.js` 加载完成，并为 `a.js` 提供相应的 `exports` 对象。

这样 `main.js` 就把这两个模块都加载完成了。这段程序的输出结果如下：

``` javascript
$ node main.js
main starting
a starting
b starting
in b, a.done = false
b.done
in a, b.done = true
a.done
in main, a.done = true, b.done = true
``` 

如果你的程序中存在循环依赖模块，请确保它们是按计划执行的。

## 文件模块

如果没有找到相应的文件名，Node.js 会尝试依次添加 `.js`、`.json` 和 `.node` 后缀名去加载。

`.js` 文件被解析为 `Javascript` 纯文本文件，`.json` 文件被解析为 JSON 的文本文件，`.node` 则会被解析为用 `dlopen` 加载的编译的插件模块。

模块以 `'/'` 为前缀，则表示绝对路径。例如：`require(/home/marco/foo.js)` 将加载的是 `/home/marco/foo.js` 这个文件。

模块以 `'./'` 为前缀，则表示路径是相对与调用 `require()` 的文件。也就是说，`circle.js` 必须与 'foo.js' 在同一个目录下， `require('./circle')` 才能找到它。

当没有使用 `'/'` 或 `'./'` 前缀来加载一个文件时，这个模块要么是一个核心模块，要么从 `node_modules` 文件夹中加载。

如果给定的路径不存在，`require()` 将抛出一个 `code` 属性为 `MODULE_NOT_FOUND` 的 `[Error(错误)](#gg)`;

## 文件夹做模块

可以把程序和库放到一个单独的文件夹里，并提供单一入口来指向它。有三种方法，使一个文件夹可以作为 `require()` 的参数来加载。

第一种方式是在文件夹的根目录创建一个叫做 `package.json` 的文件，它需要指定一个 main 模块。下面是一个 `package.json` 文件的示例：

``` javascript
{
	"name": "some-library",
	"main": "./lib/some-library.js"
}
```

示例中的这个文件，如果是放在 `./some-library` 目录下，那么 `requrie('./some-library')` 将会去加载 `./some-library/lib/some-library.js`。

这就是 Node.js 处理 `pageage.json` 文件的方式。

注意：如果在 `pageage.json` 中没法找到 `main` 入口，Node.js 将无法解析该模块，并抛出一个找不到该模块的默认错误：

``` javascript
Error: Cannot find module 'some-library'
``` 

如果目录中不存在 `pageage.json` 文件，Node.js 将会尝试去加载这个目录下的 `index.js` 或 `index.node`。例如：若上面的例子不存在 `pageage.json` 文件，`require('./some-library')` 将会尝试加载以下文件：

* `./some-library/index.js`
* `./some-library/index.node`

## 从 `node_modules` 文件夹中加载

如果传递 `require()` 的标识符不是本地模块，并且不带 `/`、`../` 或 `./` 前缀，Node.js 会从当前模块的父级目录开始，尝试在它的 `/node_modules` 文件夹中加载相应模块。Node.js 不会添加 `node_modules` 到以 `node_modules` 结尾的路径上。

例如：如果在文件 `/home/ry/projects/foo.js` 调用 `require('bar.js')`，那么 Node.js 查找其位置的顺序为：

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

这使得程序可以本地化它们的依赖，避免冲突。

你可以要求特定的文件或子模块分散在一个模块的模块名后所跟着的路径后缀中。例如，`require('example-module/path/to/file') `将被解析为相对于 `example-module` 模块的 `path/to/file` 路径中。后缀路径同样遵循模块路径的解析规则。

## 从全局文件夹加载

如果 `NODE_PATH` 环境变量设置了一个以冒号分隔的绝对路径列表，在其它位置找不到模块时，Node.js 将会从这些路径中搜索。（注：在 Windows 上，`NODE_PATH` 用的是分号，而不是冒号分隔）。

`NODE_PATH` 最初创建用以支持从不同路径加载模块，它不会在当前[模块解析](#%E6%80%BB%E8%A8%80%E4%B9%8B)算法运行之前使用。

`NODE_PATH` 仍然受支持，但在 Node.js 已经建立了一套用于定位依赖模块的机制之后，已经没有太大必要了。很多情况下，依赖于 `NODE_PATH` 路径下的环境部署会让开发者在不清楚 `NODE_PATH` 路径下依赖环境而产生很多奇怪的问题。有时候，模块的依赖关系会发生变化，导致在搜索 `NODE_PATH` 时加载不同的版本（甚至不同的模块）。

此外，Node.js将会在以下路径搜索：

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

其中 `$HOME` 为用户的主目录，`$PREFIX` 为 Node.js 配置的 `node_prefix`

这些大多是历史遗留问题。**强烈建议你将所有的依赖模块安装到本地的 `node_modules` 文件夹中**。这样它们的加载速度会更快，也更可靠。

## 模块封装

模块代码在执行之前，将被 Node.js 用类似与下面的一个函数包装起来：

``` javascript
(function(exports, require, module, __filename, __dirname) {
	// your module code actually lives in here
})
``` 

通过这种方式，Node.js 可以实现：

* 使得顶层变量（通过 `var`，`const` 或 `let`）的作用域为当前模块而不是全局对象。
* 提供一些特定在模块中使用的全局变量：
	* 开发者可以使用 `module` 和 `exports` 对象来输出模块中的值。
	* 便利变量 `__filename` 和 `__dirname`, 表示模块的绝对路径文件名和目录路径。

## 缓存

模块在它们第一次加载后会被缓存起来。这意味着如果解析到同一个文件，那么 `require('foo')` 每次都会返回完全相同的对象。

多次调用 `require(foo)` 未必会导致模块中的代码执行多次。这是一个重要的功能。借助这个功能, 可以返回“部分完成”的对象。这样, 传递依赖也能被加载, 即使它们可能导致循环依赖。

如果你希望一个模块执行多次，那就导出一个函数，然后调用这个函数

## `module` 对象

* {Object}

在每个模块中，内置变量 `module` 为当前模块的一个引用。`module.exports` 可以直接通过内置变量 `exports` 访问。`module` 变量实际上并不是全局的，而是每个模块内部的。

### module.children

* {Array}

模块会通过该数组加载模块。

### module.exports

* {Object}

`module.exports` 是通过模块系统产生的。但这并不是我们想要的，很多人都想他们的模块是某个类的示例。为了实现这一点，你得把要导出的对象复制给 `module.exports`。需要注意的是：如果将需要导出的对象复制给 `exports` 只会简单的绑定到本地变量 `exports` 上，导出结果可能不是正确的。

例如：假如我们有一个名为 `a.js` 的模块

``` javascript
const EventEmitter = require('events');

module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the moudle itself/
setTimeout(() => {
	module.exports.emit('ready');
}, 1000);
```

然后在另一个文件中，我们这样些：

``` javascript
const a = require('./a');
a.on('ready', () => {
	console.log(''module a is ready);
})
```

需要注意的是，给 `module.exports` 的赋值必须立即生效，不能在任何回调中执行，否则降将不生效。

#### exports 快捷方式

`exports` 为模块作用域内可访问的变量，它是初始化 `module.exports` 时的一个引用。

它提供一种快捷方式，以便将 `module.exports.f = ...` 写出更简洁的 `exports.f = ...`。但是，你要知道，对于任何变量而言，如果你为其赋一个新值，它将不再绑定到以前的值。

``` javascript
module.exports.hello = true; // Exported from require of module
exports = { hello: false }; // Not exported, only available in the module
```

当 `module.exports` 属性由一个新的对象完全取代时，常见的用来重新分配 `exports` 的方法如下：

``` javascript
module.exports = exports = function Constructor() {
	// ... etc.
}
```

为了解释这个情况，我们模拟一个 `require()`：

``` javascript
function require(...) {
	let module = { exports: {} };
	((module, exports) => {
		// Your module code here. In this example, define a function.
		function some_func() {};
		exports = some_func;
		// At this point, exports is no longer a shortcut to module.exports, and
		// this module will still export an empty default object.
		module.exports = some_func;
		// At this point, this module will now export some_func, instead of the
		// default object.
	}) (module, module.exports);
	return module.exports;
}
```

### module.filename

* {String}

模块完全解析后的文件名。

### module.id

* {String}

模块表示符，通常是模块完全解析后的文件名

### module.loaded

* {Boolean}

是否已完全加载完成，或者正在加载过程中。

### moudle.parent

* {Object}

加载该模块的父级模块。

### module.require(ID)

* id {String}
* return {Object} 已解析的模块的 `module.exports`

`module.require` 方法提供了一种像 `require()` 从最初模块加载另一个模块的方法。

需要注意的是，为了做到这一点，你必须获取一个 `module` 对象的引用。 `require()` 返回 `module.exports`，并且 `module` 是一个典型的只能在特定模块作用域内有效的变量，如果想要使用它，就必须明确的导出。

## 总言之

为了获取调用 require() 加载的确切的文件名，请使用 require.resolve() 函数（译者注：`console.log(require.resolve('foo'))`）。

综上所述，下面用伪代码的高级算法形式表达 require.resolve 是如何工作的：

```
require(X) from module at path Y
1. If X is a core module,
   a. return the core module
   b. STOP
2. If X begins with './' or '/' or '../'
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
3. LOAD_NODE_MODULES(X, dirname(Y))
4. THROW "not found"

LOAD_AS_FILE(X)
1. If X is a file, load X as JavaScript text.  STOP
2. If X.js is a file, load X.js as JavaScript text.  STOP
3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
4. If X.node is a file, load X.node as binary addon.  STOP

LOAD_AS_DIRECTORY(X)
1. If X/package.json is a file,
   a. Parse X/package.json, and look for "main" field.
   b. let M = X + (json main field)
   c. LOAD_AS_FILE(M)
2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
4. If X/index.node is a file, load X/index.node as binary addon.  STOP

LOAD_NODE_MODULES(X, START)
1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   c. DIR = path join(PARTS[0 .. I] + "node_modules")
   b. DIRS = DIRS + DIR
   c. let I = I - 1
5. return DIRS
```

## 附录

Node.js 的 `require()` 函数的语义被设计的足够通用化，可以支持各种常规目录结构。包管理程序如 `dpkg`、`rpm` 和 `npm` 将不用修改就能够从 Node.js 模块构建本地包。

接下来我们将给你一个可行的目录结构建议：

假设我们希望将一个包的指定版本放在 `/usr/lib/node/<some-package>/<some-version>` 目录中。

包可以依赖于其他包。为了安装包 `foo` ，可能需要安装包 `bar` 的一个指定版本。 包 `bar` 也可能有依赖关系，在某些情况下依赖关系可能发生冲突或形成循环。

因为 Node.js 会查找它所加载的模块的真实路径（也就是说会解析符号链接），然后按照[上文描述的方式](#%E4%BB%8E-node_modules-%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%AD%E5%8A%A0%E8%BD%BD)在 `node_modules` 目录中查询依赖关系，这种情形跟以下体系结构非常相似：

* `/usr/lib/node/foo/1.2.3/` - foo 包 1.2.3 版本的内容
* `/usr/lib/node/bar/4.3.2/` - foo 包所依赖的 bar 包的内容
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - 指向 `/usr/lib/node/bar/4.3.2/` 的符号链接
* `/usr/lib/node/bar/4.3.2/node_modules/*` - 指向 `bar` 包所依赖的包的符号链接

因此，即便存在循环依赖或依赖冲突，每个模块还是可以获得它所依赖的包的一个可用版本。

当 `foo` 包中的代码调用 `require('bar')` ，将获得符号链接 `/usr/lib/node/foo/1.2.3/node_modules/bar` 指向的版本。 然后，当 `bar` 包中的代码调用 `require('queue')` ，将会获得符号链接 `/usr/lib/node/bar/4.3.2/node_modules/quux` 指向的版本。

此外，为了进一步优化模块搜索过程，不要将包直接放在 `/usr/lib/node` 目录中，而是将它们放在 `/usr/lib/node_modules/<name>/<version>` 目录中。 这样在找不到依赖包的情况下，Node.js 就不会在 `/usr/node_modules` 或 `/node_modules` 目录中查找了。

为了使模块在 Node.js 的 **REPL** 中可用，你可能需要将 `/usr/lib/node_modules` 目录加入到 `$NODE_PATH` 环境变量中。由于在 `node_modules` 目录中搜索模块使用的是相对路径，使得调用 `require()` 获得的是基于真实路径的文件，因此包本身可以放在任何位置。