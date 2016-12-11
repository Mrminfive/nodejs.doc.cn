<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [定时器](#%E5%AE%9A%E6%97%B6%E5%99%A8)
  - [Class：Immediate](#classimmediate)
  - [Class: Timeout](#class-timeout)
    - [timeout.ref()](#timeoutref)
    - [timeout.unref()](#timeoutunref)
  - [预定定时器](#%E9%A2%84%E5%AE%9A%E5%AE%9A%E6%97%B6%E5%99%A8)
    - [setImmediate(callback[, ...args])](#setimmediatecallback-args)
    - [setTimeout(callback, delay[, ...args])](#settimeoutcallback-delay-args)
    - [setInterval(callback, delay[, ...args])](#setintervalcallback-delay-args)
  - [取消定时器](#%E5%8F%96%E6%B6%88%E5%AE%9A%E6%97%B6%E5%99%A8)
    - [clearImmediate(immediate)](#clearimmediateimmediate)
    - [clearTimeout(timeout)](#cleartimeouttimeout)
    - [clearInterval(timeout)](#clearintervaltimeout)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 定时器

> 稳定性：**3** - 已锁定

`timer` 模块暴露了一个全局 API 用于在将来的某个时间段调用函数。因为定时器函数是全局的，所以不需要使用 `require('timers')` 就能使用该 API。

Node.js 中的定时器函数实现了一个跟 Web 浏览器提供的定时器类似的 API。但内部实现原理是基于[Node.js的事件循环](https://github.com/nodejs/node/blob/master/doc/topics/event-loop-timers-and-nexttick.md)。

## Class：Immediate

这个类由[setImmediate()](#setimmediatecallback-args)函数在内部生成并返回。它可以传递给[clearImmediate()](#clearimmediateimmediate)函数以取消计划的操作。

## Class: Timeout

这个类由[setTimeout()](#settimeoutcallback-delay-args)和[setInterval()](#setintervalcallback-delay-args)函数在内部生成并返回。它可以分别传递给[clearTimeout()](#cleartimeouttimeout)或[clearInterval()](#clearintervaltimeout)已取消计划的操作。

默认情况下，当时用[setTimeout()](#settimeoutcallback-delay-args)或[setInterval()](#setintervalcallback-delay-args)调用定时器时，只要定时器仍处于活动状态，Node.js 事件循环将继续运行。这些函数返回的每个 `Timeout` 对象都会暴露可用于控制次默认行为的 `timeout.ref()` 和 `timeout.unref()` 函数。

### timeout.ref()

被调用时，只要 `Timeout` 对象仍处于活动状态，就会重新将 `Timeout` 对象插入到 Node.js 事件循环队列中。多次调用 `timeout.ref()` 不会有有其它效果。

注意：默认情况下，所有的 `Timeout` 对象都处于 `'ref'd` 状态，通常不需要调用 `timeout.ref()`，除非之前调用了 `timeout.unref()` 。

返回一个对 `Timeout` 的引用。

### timeout.unref()

被调用时，只要 `Timeout` 对象仍处于活动状态，将会将 `Timeout` 对象推出 Node.js 事件循环队列。如果没有其它操作将其重新插入事件循环队列中，那么该进程将会在 `Timeout` 对象的回调被调用之前退出。多次调用 `timeout.unref()` 不会有其它效果。


## 预定定时器

Node.js 中的计时器是一种会在一段时间后调用给定的函数的内部构造。定时器函数会在何时被调用，取决于用来创建定时器的方法以及 Node.js 事件循环是否正在做其他工作。

### setImmediate(callback[, ...args])

* `callback` {Function} 在 Node.js 事件循环回合结束时调用的函数。
* `...args` {Any} 在 `callback` 被调用时传递的可选参数。

预定 “immediate” 执行 callback，它是在 I/O 事件的回调之后并在使用[setTimeout()](#settimeoutcallback-delay-args) 和[setInterval()](#setintervalcallback-delay-args) 创建的计时器之前被触发。返回一个用于[clearImmediate()](#clearimmediateimmediate)的 `Immediate` 对象。

当多次调用 `setImmediate()` 时，回调函数会按照它们的创建顺序依次执行。每个事件循环迭代都会处理整个回调队列。如果立即定时器正在执行回调中排队，那么该定时器直到下一个事件循环迭代之前将不会被触发。

如果 `callback` 不是一个函数，将会抛出一个[TypeError](#error)。

### setTimeout(callback, delay[, ...args])

* `callback` {Function} 当定时器到点时回调的函数。
* `delay` {Number} 在调用 `callback` 之前等待的毫秒数
* `...args` {Any} 在 `callback` 被调用时传递的可选参数。

在 `delay` 毫秒之后执行一次 `callback`。返回一个可用于 [clearTimeout()](#cleartimeouttimeout) 的 `Timeout` 对象。

`callback` 可能不会精确地在 `delay` 毫秒被调用。Node.js 不能保证回调被触发的确切时间，也不能保证它们的顺序。回调会在尽可能接近所指定的时间上调用。

*注意：当 `delay` 大于 `2147483647` 或小于 `1` 时，`delay` 会被设置为 `1`。*

如果 `callback` 不是一个函数，将会抛出一个[TypeError](#error)。

### setInterval(callback, delay[, ...args])

* `callback` {Function} 当定时器到点时回调的函数。
* `delay` {Number} 在调用 `callback` 之前等待的毫秒数
* `...args` {Any} 在 `callback` 被调用时传递的可选参数。

预定每隔 `delay` 毫秒重复执行 `callback`。返回一个可用于 [clearInterval()](#clearintervaltimeout) 的 `Timeout` 对象。

`callback` 可能不会精确地在 `delay` 毫秒被调用。Node.js 不能保证回调被触发的确切时间，也不能保证它们的顺序。回调会在尽可能接近所指定的时间上调用。

*注意：当 `delay` 大于 `2147483647` 或小于 `1` 时，`delay` 会被设置为 `1`。*

如果 `callback` 不是一个函数，将会抛出一个[TypeError](#error)。


## 取消定时器

每个[setImmediate()](#setimmediatecallback-args)、[setTimeout()](#settimeoutcallback-delay-args) 和 [setInterval()](#setintervalcallback-delay-args)方法都会返回一个表示预定定时器的对象。这些对象可以用来取消定时器已防止其触发。

### clearImmediate(immediate)

* `immediate` {Immediate} 由 [setImmediate()](#setimmediatecallback-args) 返回的 `Immediate` 对象。

取消由 [setImmediate()](#setimmediatecallback-args) 返回的 `Immediate` 对象。

### clearTimeout(timeout)

* `timeout` {Timeout} 由 [setTimeout()](#settimeoutcallback-delay-args) 返回的 `Timeout` 对象。

取消由 [setTimeout()](#settimeoutcallback-delay-args) 返回的 `Timeout` 对象。

### clearInterval(timeout)

* `timeout` {Timeout} 由 [setInterval()](#setintervalcallback-delay-args) 返回的 `Timeout` 对象。

取消由 [setInterval()](#setintervalcallback-delay-args) 返回的 `Timeout` 对象。