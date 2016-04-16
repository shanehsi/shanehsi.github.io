title: Babel Stage-x Syntax
date: 2016-04-16 12:14:01
tags:
---

# stage 0

[stage0](https://github.com/tc39/ecma262/blob/master/stage0.md)

## Function Bind Syntax

文章 [Function Bind Syntax](https://babeljs.io/blog/2015/05/14/function-bind).

# stage 1

和 TypeScript 类似，使用 TypeScript 语法即可。

# stage 2

## [es-trailing-function-commas](https://github.com/jeffmo/es-trailing-function-commas)

这个倒还好。

## [ecmascript-rest-spread](https://github.com/sebmarkbage/ecmascript-rest-spread)

1 Rest Properties

```js
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
```

2 Spread Properties

```js
let n = { x, y, ...z };
n; // { x: 1, y: 2, a: 3, b: 4 }
```

# stage 3

## Async to generator transform

In

```js
async function foo() {
  await bar();
}
```

Out

```js
var _asyncToGenerator = function (fn) {
  ...
};
var foo = _asyncToGenerator(function* () {
  yield bar();
});
```

> todo 深入理解 async/await yield promise

## [exponentiation-operator](https://github.com/rwaldron/exponentiation-operator)

```js
// x ** y

let squared = 2 ** 2;
// same as: 2 * 2

let cubed = 2 ** 3;
// same as: 2 * 2 * 2
```




