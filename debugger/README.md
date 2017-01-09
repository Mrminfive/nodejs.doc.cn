<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Debugger](#debugger)
  - [监视器](#%E7%9B%91%E8%A7%86%E5%99%A8)
  - [命令参考](#%E5%91%BD%E4%BB%A4%E5%8F%82%E8%80%83)
    - [步进](#%E6%AD%A5%E8%BF%9B)
    - [断点](#%E6%96%AD%E7%82%B9)
    - [信息](#%E4%BF%A1%E6%81%AF)
    - [执行控制](#%E6%89%A7%E8%A1%8C%E6%8E%A7%E5%88%B6)
    - [杂项](#%E6%9D%82%E9%A1%B9)
  - [高级用法](#%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95)
  - [集成了 V8 lnspector 的 Node.js](#%E9%9B%86%E6%88%90%E4%BA%86-v8-lnspector-%E7%9A%84-nodejs)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Debugger

> 稳定性：**2** - 稳定

Node.js 包含一个通过[基于TCP协议](https://github.com/v8/v8/wiki/Debugging-Protocol)访问的进程外的调试工具，并内置调试客户端。通过 Node.js 参数 `debug` 加上调试脚本的路径启动可以使用它；启动后将显示一个提示，指示调试器已启动：

``` javascript
$ node debug myscript.js
< Debugger listening on 127.0.0.1:5858
connecting to 127.0.0.1:5858 ... ok
break in D:\demo\node\my-blog\aa\debugger\myscript.js:1
> 1 x = 5;
  2 setTimeout(() => {
  		debugger;
debug>
```

Node.js 的调试器客户端不支持所有功能，但简单的步骤（断点）和检查是支持的。

在脚本的源代码中插入 `debugger` 将在代码中该位置上进行断点。

``` javascript
x = 5;
setTimeout(() => {
  debugger;
  console.log('world');
}, 1000);
console.log('hello');
```

一旦调试器运行，断点将出现在第4行：

``` javascript
$ node debug myscript.js
< debugger listening on port 5858
connecting... ok
break in /home/indutny/Code/git/indutny/myscript.js:1
  1 x = 5;
  2 setTimeout(() => {
  3   debugger;
debug> cont
< hello
break in /home/indutny/Code/git/indutny/myscript.js:3
  1 x = 5;
  2 setTimeout(() => {
  3   debugger;
  4   console.log('world');
  5 }, 1000);
debug> next
break in /home/indutny/Code/git/indutny/myscript.js:4
  2 setTimeout(() => {
  3   debugger;
  4   console.log('world');
  5 }, 1000);
  6 console.log('hello');
debug> repl
Press Ctrl + C to leave debug repl
> x
5
> 2+2
4
debug> next
< world
break in /home/indutny/Code/git/indutny/myscript.js:5
  3   debugger;
  4   console.log('world');
  5 }, 1000);
  6 console.log('hello');
  7
debug> quit
```

`repl` 允许代码远程评估命令。`next` 步骤的下一行命令。键入 `help` 看看其他可用命令。

按 `enter` 未键入命令将重复以前的调试命令。

## 监视器

可以在调试时查看表达式和变量值。每个断点都会在当前上下文中评估观察列表中的每个表达式，并在列举断点的源代码前立即被显示。

通过键入 `watch('my_expression')` 来监视一个表达式， `watchers` 命令会将其打印在激活的监视器中。通过键入 `unwatch('my_expression')` 来移除一个监视器。

## 命令参考

### 步进

* `cont`，`c` - 继续执行
* `next``n` - 下一步
* `step``s` - 介入
* `out``o` - 退出介入
* `pause` - 暂停运行代码（类似开发着工具中的暂停按钮）

### 断点

* `setBreakpoint()`, `sb()` - 在当前行设置断点
* `setBreakpoint(line)`, `sb(line)` - 在特定行设置断点
* `setBreakpoint('fn()')`, `sb(...)` - 设置断点在函数体的第一条语句
* `setBreakpoint('script.js', 1)`, `sb(...)` - 对 script.js 的第一行设置断点
* `clearBreakpoint('script.js', 2)`, `cb(...)` - 清楚 script.js 第一行的断点

也可以在未加载的文件（模块）中设置断点：

``` javascript
$ ./node debug test/fixtures/break-in-module/main.js
< debugger listening on port 5858
connecting to port 5858... ok
break in test/fixtures/break-in-module/main.js:1
  1 var mod = require('./mod.js');
  2 mod.hello();
  3 mod.hello();
debug> setBreakpoint('mod.js', 23)
Warning: script 'mod.js' was not loaded yet.
  1 var mod = require('./mod.js');
  2 mod.hello();
  3 mod.hello();
debug> c
break in test/fixtures/break-in-module/mod.js:23
 21
 22 exports.hello = () => {
 23   return 'hello from module';
 24 };
 25
debug>
```

### 信息

* `backtrace`, `bt` - 当前执行帧的打印回溯
* `list(5)` - 列出脚本源代码的 5 行上下文（前 5 行）
* `watch(expr)` - 将表达式添加到观察列表中
* `unwatch(expr)` - 从观察列表中删除表达式
* `watchers` - 列出所有观察者及其值（自动列在每个断点上）
* `repl` - 在调试脚本的上下文中打开调试器的 repl 来评估
* `exec expr` - 在调试脚本的上下文中执行表达式

### 执行控制

* `run` - 运行脚本（在调试器启动时自动运行）
* `restart` - 重新启动脚本
* `kill` - 终止脚本

### 杂项

* `scripts` - 列出所有加载的脚本
* `version` - 显示 V8 引擎的版本


## 高级用法

启用和访问调试器的另一种方式是在启动 Node.js 时添加 `--debug` 命令行标志，或向已存在的 Node.js 进程发送 `SIGUSR1` 信号。

一个进程一旦以这种方式进入了调试模式，它就可以被 Node.js 调试器连接使用，通过连接已运行的进程的 `pid` 或访问这个正在监听的调试器的 `URI`：

* `node debug -p <pid>` - 通过 pid 连接进程
* `node debug <URI>` - 通过类似 `localhost:5858` 的 URI 连接进程

## 集成了 V8 lnspector 的 Node.js

*注意：这是一个实验功能*

V8 lnspector 集成允许使用 Chrome DevTools（开发者工具） 附加到 Node.js 实例以进行调试和分析。

V8 lnspector 可以通过 `--inspect` 启动标志启动 Node.js 应用程序以启用它。它也可以定制端口，例如 `--inspect=9222` 将接受在端口9222 DevTools连接。

提供 `--debug-brk` 标志可以打破应用程序代码的第一行， `--inspect`除外。

``` javascript
$ node --inspect index.js
Debugger listening on port 9229.
Warning: This is an experimental feature and could change at any time.
To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/remote/serve_file/@60cd6e859b9f557d2312f5bf532f6aec5f284980/inspector.html?experiments=true&v8only=true&ws=localhost:9229/node
```