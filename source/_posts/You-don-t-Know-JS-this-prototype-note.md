title: "You don't Know JS - this & prototype note"
date: 2016-02-05 10:35:27
tags:
---

# 1. 关于 `this`

`this` 关键字是 JavaScript 最复杂的机制之一。

被自动定义在所有函数的作用域中。

很难说清它指向什么。

实际上 `this` 没有那么先进。但是开发者往往把理解过程复杂化。

## 1.1 为什么要用 `this`

不同的上下文对象。重复使用的函数。

```js
function identify() {
    return this.name.toUpperCase();
}

var me = {
    name: "Kyle"
};

var you = {
    name: "Reader"
}

identify.call(me); // KYLE
identify.call(you); // READER
```

如果不是 `this`，就是显式传入。

```
function identify(context) {
    return context.name.toUpperCase();
}
```

所以，`this` 提供了一种更优雅的方式来隐式『传递』一个对象引用。可以将API设计的更加简洁。

随着模式越来越复杂，显式传递上下文对象会让代码越来越混乱。

当介绍到对象和原型时，你会明白函数自动引用合适的上下文对象是多么的重要。

## 1.2 误解

之后解释 `this` 的具体用法，先消除误解。

不要拘泥于 `this` 的字面解释。以下是错误的。

### 1.2.1 指向自身

`this` 不指向函数自身。

如果 `this` 要指向自身，什么场景？递归，调用函数内储存的状态（属性的值）。本书介绍的其他模式，有比函数对象更适合储存状态的地方。

```js
function foo(num) {
    console.log("foo: " + num);
    this.count++;
}

foo.count = 0;

var i;
for(i = 0; i < 10; i++) {
    if(i > 5) {
        foo(i);
    }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

console.log(foo.count); // 0 -- WFT?
```

`this.count` 无意间创建了一个全局变量。最终的值是 `NaN`。

当然，也可以使用词法作用域（舒适区）规避这个问题。不过不能放弃对 `this` 的学习。

```js
function foo(num) {
    console.log("foo: " + num);
    data.count++;
}

var data = {
    count: 0
};

```

如果要从函数内部引用自身，只用 `this` 是不够的。

一般来说要通过一个 **指向函数对象的词法标识符（变量）** 来引用它。

思考下：

```js
function foo() {
    foo.count = 4; // foo 指向自身
}

setTimeout(function() {
    // 匿名函数无法指向自身
}, 10);
```

第一个函数称为具名函数，在内部可以通过 `foo` 引用自身。

所以，还有一种解决方式：

```js
function foo(num) {
    console.log("foo: " + num);
    foo.count++;
}

foo.count = 0;
```

然而，这种方法还是回避了 `this`，完全依赖于变量 `foo` 的词法作用域。

另一种方法是强制 `this` 指向 `foo`（使用 `call`）：

```js
for(i = 0; i < 10; i++) {
    if(i > 5) {
        foo.call(foo, i);
    }
}
```

这次我们接受了 `this`。后面会解释。

### 1.2.2 它的作用域

第二种常见的误解是，`this` 指向函数的作用域。这个问题有点复杂，某些情况下正确，其他是错误的。

但是，需要明确，`this` 在任何情况下都不指向函数的作用域。

在 JavaScript 中，作用域确实和对象类似，可见标识符都是它的属性。但是作用域对象无法通过 JavaScript 代码访问。它存在于 JavaScript 引擎内部。

思考下这段代码，它试图（没有成功）跨越边界，使用 `this` 隐式引用函数的词法作用域：

```js
function foo() {
    var a = 2;
    this.bar();
}

function bar() {
    console.log(this.a);
}

foo();  // ReferenceError: a is not defined
```

这段代码的错误不止一个，非常完美（同时令人伤感地）展示了 `this` 多么容易误导人。

- `this.bar()` 引用 `bar()` 是绝对不可能成功的。之后会解释原因。最自然的方法是略去 `this`，直接使用词法 **引用标识符**。

- 还试图使用 `this` 联通 `foo()` 和 `bar()` 的词法作用域。从而让 `bar()` 可以访问 `foo()` 作用域里的变量 `a`。这是不可能的。不能使用 `this` 来引用一个词法作用域内部的东西。

每当你想把 `this` 和 **词法作用域查找** 混合使用的时候，一定要提醒自己，这是不可能实现的。

## 1.3 `this` 到底是什么

`this` 是在运行时绑定的，不是函数声明的位置。取决于函数调用时的各种条件。

当一个函数被调用时，会创建一个活动记录（有时候也成为执行上下文）。

这个记录会包含函数

- 在哪里被调用（调用栈）
- 函数的调用方法
- 传入的参数
- 等

`this` 就是记录了其中一个属性，它会在执行过程中用到。

## 1.4 小结

要投入时间去理解。

第一步是消除两个误解，`this` 既不指向函数自身，也不指向函数的词法作用域。

`this` 在函数被调用中发生时的绑定，取决于调用方式。

# 2. `this` 全面解析

## 2.1 调用位置

怎么找调用位置？似乎就是函数被调用的位置。但是不是那么简单，某些编程模式会隐藏真正的调用位置。

最重要的是分析 **调用栈** （即为了到达当前执行位置调用的所有函数）。我们关心的调用位置就是当前执行函数的前一个调用。






