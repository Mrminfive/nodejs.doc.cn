### 目录

[TOC]

+ [assert(value[,message])](#assertvaluemessage)
+ [assert.deepEqual(actual, expected[,message])](#assertdeepequalactual-expectedmessage)
+ [assert.deepStrictEqual(actual, expected[,message])](#)
+ [assert.doesNotThrow(block[,error][,message])](#)
+ [assert.equal(actual, expected[,message])](#)
+ [assert.fail(actual, expected , message, operator)](#)
+ [assert.ifError(value)](#)
+ [assert.notDeepEqual(actual, expected[,message])](#)
+ [assert.notDeepStrictEqual(actual, expected[,message])](#)
+ [assert.notEqual(actual, expected[,message])](#)
+ [assert.notStrictEqual(actual, expected[,message])](#)
+ [assert.ok(value[,message])](#assert.ok(value[,message]))
+ [assert.strictEqual(actual, expected[,message])](#)
+ [assert.throws(block[,error][,message])](#)


## Assert(断言)

> 稳定度 **3** - 已锁定

`assert` 模块提供了一组简单的方法，可用于做断言测试。该模块为 Node.js 内置模块，可以通过 `request('assert')` 引用并在应用代码中使用。然而， `assert` 并不是一个测试框架，也不会往通用断言库的方向去发展。

## assert(value[,message])

[assert.ok()](#assert.ok(value[,message]) 方法的别名

```js
const assert = require('assert');

assert(true);
// OK
assert(1);
// OK
assert(false);
// throws "AssertionError: false == true"
assert(0);
// throws "AssertionError: 0 == true"
assert(false, 'it\'s false);
// throws "AssertionError: it's false"
```

## assert.deepEqual(actual, expected[,message])
