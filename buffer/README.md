＃ Buffer

在 ECMAScript 2015（ES6）推出 [TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 之前，JavaScript语言是没有任何机制可以用于读取或操纵二进制数据流了。｀Buffer｀ 类被采纳为 Node.js API 的一部分使得在 TCP 流和文件系统操作等的上下文中与八位字节流进行交互成为可能。`Buffer` 类被采纳为 Node.js API 的一部分，使得 TCP 流和文件系统操作等的上下文中与八位字节流进行交互成为可能。

现在，在ES6中添加了 [TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)，｀Buffer｀ 类以一种更优化和适用于 Node.js 用例的方式实现了 [Uint8Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) API。

`Buffer` 类的实例类似于整数数组，但会分配V8外堆固定大小的原始内存。 缓冲区的大小在创建时确定，不能调整大小。

`Buffer` 类在 Node.js 中是一个全局变量，因此不太可能需要使用到 `require('buffer')`。

例子：

``` javascript
// Creates a zero-filled Buffer of length 10
const buf1 = Buffer.alloc(10);

// Creates a Buffer of length, filled with 0x1
const buf2 = Buffer.alloc(10, 1);

// Creates an uninitialized buffer of length 10.
// This is faster than calling Buffer.alloc() but the returned
// Buffer instance might contain old data that needs to be
// overwritten using either fill() or write().
const buf3 = Buffer.allocUnsafe(10);

// Creates a Buffer containing [0x1, 0x2, 0x3]
const bufs = Buffer.form([1, 2, 3]);

// Creates a Buffer containing ASCII bytes [0x74, 0x65, 0x73, 0x74].
const buf5 = Buffer.from('test');

// Creates a Buffer containing UTF-8 bytes [0x74, 0xc3, 0xa9, 0x73, 0x74].
const buf6 = Buffer.from('tést', 'utf8');
```

