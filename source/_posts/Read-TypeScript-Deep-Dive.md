title: 阅读 TypeScript Deep Dive
date: 2016-02-27 09:28:10
tags:
---

## __extends

```js
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```

d - derived class
b - base class

**1.** 将 static members 复制到 d。

```js
for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
```


**2.** 设置 d 的 class function 的 prototype 可选的查找 b 的 proto 的成员。

```js
d.prototype.__proto__ = b.prototype
```

详细解释下 **2**。

首先解释为什么 `d.prototype.__proto__ = b.prototype` 和 

```js
function __() { this.constructor = d; }
__.prototype = b.prototype;
d.prototype = new __();
```

等价。

然后解释这句话的重要性。

先了解这些概念：

1. `__proto__`
2. prototype
3. effect of `new` on `this` inside the called function
4. effect of `new` on `prototype` and `__proto__`

每一个 JavaScript 对象都有 `__proto__`。其中一个目的是：当 `obj.property` 找不到，会找 `obj.__proto__.property`，然后 `obj.__proto__.__.proto__.property`。这就是原型继承。

另一个有用的信息是，所有的 JavaScript function 有一个 property 叫做 `prototype`。并且有一个 member 叫做 `constructor` 指向这个 function。

```js
function Foo() { }
console.log(Foo.prototype);
console.log(Foo.prototype.constructor === Foo);
```

现在再看下：`new` 对 `this` 的作用。

```js
function Foo() {
    this.bar = 123;
}
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

大意是：`this` 会指向新创建的对象。原因是，**在 Function 上使用 `new` 将 `prototype` 复制到新创建对象的 `__proto__`**。

```js
function Foo() {

}

var foo = new Foo();
console.log(foo.__proto__ === Foo.prototype); // True
```

以上。

现在看下： `__extends__`。


```js
function __() { this.constructor = d; }
__.prototype = b.prototype;
d.prototype = new __();
```

d.prototype.__proto__ = __.prototype

-> d.prototype = { __proto__: __.prototype}
-> d.prototype = { __proto__: b.prototype }

等下，我们需要 d 的 prototype.constructor 保持原来的。

d.prototype.constructor = d;



