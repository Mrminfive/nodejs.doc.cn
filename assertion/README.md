<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Assert(断言)](#assert%E6%96%AD%E8%A8%80)
    - [assert(value[,message])](#assertvaluemessage)
    - [assert.equal(actual, expected[, message])](#assertequalactual-expected-message)
    - [assert.notEqual(actual, expected[, message])](#assertnotequalactual-expected-message)
    - [assert.strictEqual(actual, expected[, message])](#assertstrictequalactual-expected-message)
    - [assert.notStrictEqual(actual, expected[, message])](#assertnotstrictequalactual-expected-message)
    - [assert.deepEqual(actual, expected[, message])](#assertdeepequalactual-expected-message)
    - [assert.notDeepEqual(actual, expected[, message])](#assertnotdeepequalactual-expected-message)
    - [assert.deepStrictEqual(actual, expected[, message])](#assertdeepstrictequalactual-expected-message)
    - [assert.notDeepStrictEqual(actual, expected[, message])](#assertnotdeepstrictequalactual-expected-message)
    - [assert.ok(value[, message])](#assertokvalue-message)
    - [assert.fail(actual, expected, message, operator)](#assertfailactual-expected-message-operator)
    - [assert.ifError(value)](#assertiferrorvalue)
    - [assert.throws(block[, error][, message])](#assertthrowsblock-error-message)
    - [assert.doesNotThrow(block[, error][, message])](#assertdoesnotthrowblock-error-message)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Assert(断言)

> 稳定度 **3** - 已锁定

`assert` 模块提供了一组简单的方法，可用于做断言测试。该模块为 Node.js 内置模块，可以通过 `request('assert')` 引用并在应用代码中使用。然而， `assert` 并不是一个测试框架，也不会往通用断言库的方向去发展。

---

### assert(value[,message])

[assert.ok()](#assertokvalue-message) 方法的别名

```javascript
const assert = require('assert');

assert(true);
// OK
assert(1);
// OK
assert(false);
// throws "AssertionError: false == true"
assert(0);
// throws "AssertionError: 0 == true"
assert(false, 'it\'s false');
// throws "AssertionError: it's false"
```

---

### assert.equal(actual, expected[, message])

浅测试，使用等于运算符（==）对 `actual` 和 `expected` 进行比较

``` javascript
const assert = require('assert');

assert.equal(1, 1);
// OK, 1 == 1
assert.equal(1, '1');
// OK, 1 == '1'

assert.equal(1, 2);
// AssertionError: 1 == 2
assert.equal({ a: { b: 1 } }, { a: { b: 1 } });
// AssertionError: { a: { b: 1 } } == { a: { b: 1 } }

// 译者注：
assert.equal({}, {});
// 抛出AssertionError: {} == {}
```

如果这两个值不相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.notEqual(actual, expected[, message])

浅测试，使用不等于运算符（!=）对 `actual` 和 `expected` 进行比较

``` javascript
const assert = require('assert');

assert.noEequal(1, 2);
// OK
assert.notEqual(1, 1);
// AssertionError: 1 != 1

assert.notEqual(1, '1');
// AssertionError: 1 != '1'
```

如果这两个值相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.strictEqual(actual, expected[, message])

使用全等运算符（===）进行测试。

``` javascript
const assert = require('assert');

assert.strictEqual(1, 2);
// AssertionError: 1 === 2

assert.strictEqual(1, 1);
// OK

assert.strictEqual(1, '1');
// AssertionEqual: 1 === '1'
```

如果这两个值不严格相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.notStrictEqual(actual, expected[, message])

使用不全等运算符（!==）进行测试。

``` javascript
const assert = require('assert');

assert.notStrictEqual(1, 2);
// OK

assert.notStrictEqual(1, 1);
// AssertionEqual: 1 !== 1

assert.strictEqual(1, '1');
// OK
```

如果这两个值严格相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.deepEqual(actual, expected[, message])

深度比较 `actual` 和 `expected` 参数，原始值采用（==）进行比较。

只检测参数本身可枚举的属性。`deepEqual()` 方法检测被测试对象的原型，绑定的符号，或者不可枚举的属性。这会导致一些潜在的不可思议的结果。例如：下面的例子不抛出 `AssertionError`, 因为 [Error](../error/#) 对象是不可枚举的。

``` javascript
// WARNING: This does not throw an AsertionError!
assert.deepEqual(Error('a'), Error('b'))

// 译者注：符号属性同样不会做值的测试
const syb = Symbol('foo');

let a = {
	[syb]: 1,
	value: 1
}

let b = {
	[syb]: 2,
	value: 1
}

assert.deepEqual(a, b);
```

深度比较意味着子对象可枚举的属性也会进行测试：

``` javascript
const assert = require('assert');

const obj1 = {
	a: {
		b: 1
	}
};
const obj2 = {
	a: {
		b: 2
	}
};
const obj3 = {
	a: {
		b: 1
	}
};
const obj4 = Object.create(obj1);

assert.deepEqual(obj1, obj1);
// OK, object is equal to itself

assert.deepEqual(obj1, obj2);
// AssertionError: { a: { b: 1 } } deepEqual { a: { b: 2 } }
// values of b are different

assert.deepEqual(obj1, obj2);
// OK, objects are equal

assert.deepEqual(obj1, obj4);
// AssertionError: { a: { b: 1 } } deepEqual {}
// Prototypes are ignored
```

如果这两个值不相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.notDeepEqual(actual, expected[, message])

深度的不相等测试。与 [assert.deepEqual()](#assertdeepequalactual-expected-message) 相反。

``` javascript
const assert = require('assert');

const obj1 = {
	a: {
		b: 1
	}
};
const obj2 = {
	a: {
		b: 2
	}
};
const obj3 = {
	a: {
		b: 1
	}
};
const obj4 = Object.create(obj1);

assert.notDeepEqual(obj1, obj1);
// AssertionError: { a: { b: 1 } } notDeepEqual { a: { b: 1 } }

assert.notDeepEqual(obj1, obj2);
// OK, obj1 and obj2 are not not deeply equal

assert.notDeepEqual(obj1, obj3);
// AssertionError: { a: { b: 1 } } notDeepEqual { a: { b: 1 } }

assert.notDeepEqual(obj1, obj4);
// OK, obj1 and obj2 are not deeply equal
```

如果这两个值深度相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.deepStrictEqual(actual, expected[, message])

与 `assert.deepEqual` 方法相似，但有两点不同。首先，原始值是使用全等运算符（===）进行比较的。其次，将会对两个比较对象的原型进行严格比较

``` javascript
const assert = requeire('assert');

assert.deepEqual({ a: 1 }, { a: '1' });
// OK, because 1 == '1'

assert.deepStictEqual({ a: 1 }, { a: '1' });
// AssertionError: { a: 1 } deepStrictEqual { a: '1' }
// because 1 !== '1' using strict equality
```

如果这两个值不相等，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.notDeepStrictEqual(actual, expected[, message])

深度严格不相等测试。与[assert.deepStrictEqual()](#q)。

``` javascript
const assert = require('assert');

assert.notDeepEqual({ a: 1 }, { a: '1' });
// AssertionError: { a: 1 } notDeepEqual { a: '1' }

assert.notDeepStrictEqual({ a: 1 }, { a: '1' });
// OK
```

---

### assert.ok(value[, message])

测试 `value` 是不是真值。它等同于 `assert.equal(!!value, true, message)`。

``` javascript
const assert = require('assert');

assert.ok(true);
// OK
assert.ok(false);
// throw "AssertionError: false == true"
assert.ok(0);
// throw "AssertionError: 0 == true"
assert.ok(false, 'it\'s false');
// throw "AssertionError: it's false"
```

如果 `value` 不是真值，将会抛出一个带有 `message` 属性(等于 `message` 参数的值)的 `AssertionError` 错误。如果 `message` 参数未定义，将会分配默认的错误信息。

---

### assert.fail(actual, expected, message, operator)

抛出一个 `AssertionError`。如果 `message` 是假值，错误信息会被设置为被 `operator` 分隔在两边 `actual` 和 `expected` 的值。否则，该错误信息会是 `message` 的值。

``` javascript
const assert = require('assert');

assert.fail(1, 2, undefined, '>');
// AssertionError: 1 > 2

assert.fail(1, 2, 'whoops', '>');
// AssertionError: whoops
```

---

### assert.ifError(value)

如果 `value` 是真值，抛出 `value` 错误。它对于验证回调函数的第一个参数十分有用。

``` javascript
const assert = require('assert');

assert.ifError(0);
// OK
assert.ifError(1);
// Throws 1
assert.ifError('error');
// Throws 'error'
assert.ifError(new Error());
// Throws Error

// 译者注：

function api(options, callback) {
	// 假设接口访问失败
	callback(error);	
}

api({}, (err, data) => {
	assert.ifError(err);
	// 接口范围错误时将会抛出错误
});
```

---

### assert.throws(block[, error][, message])

断言 `block` 函数会抛出错误。

`error` 可以指定为一个构造函数、[RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)或验证函数。

如果指定 `message`，`block` 因为失败而抛出错误，`message` 将提供为 `AssertionError` 的值。

使用构造函数验证的示例：

``` javascript
assert.throws(
	() => {
		throw new Error('Wrong value');
	},
	Error
);
```

使用[RegExp]()验证错误信息：

``` javascript
assert.throws(
	() => {
		throw new Error('Wrong value');
	},
	/value/
);
```

自定义错误验证：

``` javascript
assert.throws(
	() => {
		throw new Error('Wrong value');
	},
	function(err) {
		if ((err instanceof Error) && /value/.test(err)) {
			return true;
		}
	},
	'unexpected error'
);
```

请注意，`Error` 不能是字符串。如果字符串是作为第二个参数，那么 `error` 会被假定省略，字符串将会使用 `message` 替代。这很容易导致丢失错误：

``` javascript
// THIS IS A MISTAKE! DO NOT DO THIS!
assert.throws(myFunction, 'missing foo', 'did not throw with expected message');

// Do this instead.
assert.throws(myFunction, /missing foo/, 'did not throw with expected message');
```

---

### assert.doesNotThrow(block[, error][, message])

断言 `block` 函数不会抛出错误。更多细节请参照 [assert.thorws()]()

当调用 `assert.doesNotThrow()` 时，会立即调用 `block` 函数。

如果抛出了错误，并且是与指定的 `error` 参数相同类型的错误，那么将会抛出一个 `AssertionError` 。如果是不同类型的错误，或者未定义 `error` 参数，错误将会被重新返回给调用函数。

下面例子将会抛出 [TypeError](#q)，因为在断言中并没有相匹配的错误类型：

``` javascript
assert.doesNotThrow(
	() => {
		throw now TypeError('Wrong value');
	},
	SyntaxError
);
```

然而，以下将会导致一个带有 “Got unwanted exception(TypeError)..” 信息的 `AssertionError`；

``` javascript
assert.doesNotThrow(
	() => {
		throw now TypeError('Wrong value');
	},
	TypeError
);
```

如果提供了一个 `message` 参数，`AssertionError` 被抛出时，`message` 参数的值将会被追加到 `AssertionError` 的信息中：

``` javascript
assert.doesNotThrow(
	() => {
		throw new TypeError('Wrong value');
	},
	TypeError,
	'Whoops'
);
// Throws: AssertionError: Got unwanted exception (TypeError). Whoops
```