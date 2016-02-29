title: You don't Know JS - this & prototype 笔记
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

最重要的是分析 **调用栈** （即为了到达当前执行位置调用的所有函数，可以想象成一个函数调用链）。我们关心的调用位置就是当前执行函数的前一个调用。

使用浏览器的调试工具可以方便得查看调用栈。

## 2.2 绑定规则

函数在执行过程中如何决定 `this` 的绑定对象。

找到调用位置，判断应用下面4条规则的哪一条。

### 2.2.1 默认绑定

独立函数调用

```js
function foo() {
    console.log(this.a);
}
var a = 2;
foo(); // 2
```

`a` 首先是全局对象。

但是，`this` 指向的也是全局对象。

但是，如果是 "strict mode"，全局对象无法使用默认绑定，则是：

```js
function foo() {
    "use strict";
    console.log(this.a);
}
var a = 2;
foo(); // TypeError: this is undefined
```

> 理解这层意思，但是规避这种写法。

### 2.2.2 隐式绑定

调用位置是否有上下文对象，或者说是否被某个对象包含或拥有。严格说来，后面一句的说法不准确：

```js
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); //2
```

严格说来，`foo()` 不属于 `obj` 对象。

然而，调用位置，会使用 `obj` 上下文来引用函数。

对象属性引用链只有最顶层或者说最后一层会影响调用位置。举例来说：

```js
function foo() {
    console.log( this.a );
}
var obj2 = {
    a: 42,
    foo: foo
};
var obj1 = {
    a: 2,
    obj2: obj2
};
obj1.obj2.foo(); //42
```

**隐式丢失**：

被隐式绑定的函数会丢失绑定对象，也就是说会应用默认绑定（取决于是否是 "strict mode"，决定 `this` 是绑定到全局对象还是 undefined）。

```js
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo; // 函数别名
var a = "opps, global"; // a 是全局对象的属性
bar(); //"oops, global"
```

虽然，`bar` 和 `obj.foo` 是一个引用，但是实际上，它引用的确实 `foo` 函数本身。还是要看函数有没有修饰。

还有一种同样道理，但是较隐蔽，常在回调函数中出现：

```js
function foo() {
    console.log( this.a );
}
function dooFoo(fn) {
    // fn 其实引用的是 foo
    fn(); // <-- 调用位置！
}
var obj = {
    a: 2,
    foo: foo
};
var a = "opps, global"; // a 是全局对象的属性
doFoo( obj.foo ); //"oops, global"
```

参数传递其实就是一种隐式传递（隐式赋值）。

内置函数也是一样的：

```js
setTimeout(obj.foo ,100); //"opps, global"
```

因为在 browser 端，`setTimeout` 大概伪代码是：

```js
function setTimeout(fn, delay) {
    // 等待delay毫秒
    fn(); // <-- 调用位置！
}
```

所以，回调函数丢失 `this` 绑定是很常见的。

含有更出乎意料的，某些流行的 JavaScrit 库中事件处理器常会把回调函数的 `this` 强制绑定到触发时间的 DOM 元素中（郁闷）。

反正， `this` 的改变都是意想不到的。

之后我们会介绍如何通过固定 `this` 来修复。

### 2.2.3 显式绑定

再回顾下 **隐式绑定**。

是将函数作为一个对象的属性引用，通过属性间接调用函数，从而把 `this` 间接（隐式）绑定到这个对象上。

现在显式，就是指，强制在某个对象上调用函数。

利用 JavaScript 的原型，使用 `call(..)` 和 `apply(..)` 方法。

它们的第一个参数，就是调用时，`this` 会指定绑定的对象。

```js
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2
};
foo.call(obj); //2
```

如果是 primitive 类型的值，会被转换成它的对象形式（`new String(..)`、`new Boolean(..)`、`new Number(..)`。这通常被称为『装箱』。

可惜，显式绑定也并不能解决丢失绑定的问题。原因是会修改。现在要做的是，找到一种方式，不会修改 this。

**1. 硬绑定**

显式绑定的一种变种可以解决这个问题（通过创建一个包裹函数，这样起作用的 `call` 就不会被修改了）：

```js
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2
};
// 包裹函数
var bar = function() { 
    foo.call(obj);
}
bar(); // 2
setTimeout( bar, 100); // 2
// 硬绑定的bar不可能再修改它的 this
bar.call(window); //2 
```

参数形式：

```js
var bar = function() {
    return foo.apply(obj, arguments);
}
```

提取下：

```js
function bind(fn, obj) {
    return function() {
        return fn.apply(obj, arguments);
    };
}
```

由于硬绑定是一种常用的模式，在 ES5 里提供了内置的方式：`Function.prototype.bind`。

> 所以，bind 返回的是一个硬编码的新函数

**2. API 调用的『上下文』**

第三方库的许多函数，JavaScript 语言和宿主环境中的很多新的函数，都提供了一个可选参数，通常被称为『上下文』（context），作用和 `bind` 一样，确保回调使用的是指定的 `this`。

```js
[1, 2, 3].forEach( foo, obj); // obj 就是第二个可选参数
```

内部实际上就是用 `bind` 或者 `apply` 实现了显式绑定，减少了些代码。

### 2.2.4 `new` 绑定

最后一条，但在此之前，先澄清一个常见误解，关于 JavaScript 中函数和对象的误解。

在传统的面向类的语言中，『构造函数』是类中的一些特殊方法。使用 `new` 初始化类时会调用类中的构造函数，常如：

```js
sth = new MyClz();
```

JavaScript 也有 `new`，使用方式类似，然而机制真得完全不同。

首先，重定义下 JavaScript 中的『构造函数』:

JavaScript 中，构造函数只是使用 `new` 操作符时被调用的函数。它们并不属于某个类，也不会实例化一个类。实际上，你都不能把构造函数认为是一种特殊的函数类型（即可以不要这个名称），它们只是被 `new` 操作符调用的普通函数而已。

例，`Number(..)` 作为构造函数（指会通过 `new` 操作符调用时）时的行为。

> 当 Number 在 new 表达式中被调用时，它是一个构造函数：它会初始化新创建的对象。

所以，包括内置对象函数（比如 `Number`，详情见Ch3）在内的所有函数都可以用 `new` 来调用。这种函数通常被称为『构造函数调用』。

这里有一个重要但非常细微的区别：

**实际上并不存在所谓的『构造函数』，只有对于函数的『构造调用』**。

使用 `new` 来调用函数，或者说发生构造函数调用时，会自动执行以下操作：

1. 创建（或者说构造）一个全新的对象
2. 这个新对象会被执行[[原型]]链接，即将构造函数的 prototype 复制到新对象的 `__proto__`。
3. 这个新对象会绑定到函数调用的 `this`
4. 如果函数没有返回其他对象，那么 `new` 表达式中的函数调用会自动返回这个新对象

```js
function foo(a) {
    this.a = a;
}
var bar = new foo(2);

console.log(bar.a); // 2
```

总结下，这称为 `new` 绑定。

## 2.3 优先级

本节介绍了 ES5 和 MDN `bind` 的实现（polyfill），new 的优先级是会改变硬绑定的 this。这么设计的原因是为了做『部分应用』，是『柯里化』的一种。

```js
function foo(p1, p2) {
    this.val = p1 + p2;
}
var bar = foo.bind(null, "p1");
var baz = new bar("p2");
baz.val; //p1p2
```

## 2.4 规则的例外

### 2.4.1 被忽略的 this

如果传入 null 或者 undefined，会使用默认绑定。

```js
function foo() {
    console.log(this.a);
}
var a = 2;
foo.call(null);
```

所以使用的 null 要注意下影响面。

更安全的 `this`，是做一个特殊对象。其实是为了规避这条例外的规则，传入一个不是 null，但是也没啥意义的特殊对象。

```js
var ø = Object.create(null);
```

### 2.4.2 间接引用

> 这个我觉得不是其规则例外，而是一种伪装的规则。

```js
function foo() {
    console.log( this.a );
}

var a = 2; // global
var o = { a: 3, foo: foo};
var p = { a : 4 };

o.foo(); //3
(p.foo = o.foo)(); // 2
```

其中 `(p.foo = o.foo)()` 看，调用操作符的位置。

但是如果是：

```js
p.foo = o.foo;
p.foo(); // 4
```

## 2.5 this词法

ES6 介绍的胖箭头函数，`=>`。不适用 this 的四条标准规则，而是根据外层（函数或者全局）作用域来决定 this。当然，外层的 this 是适用于上述四条标准规则的。（像是静态动态的结合）。

箭头函数的 this 无法被修改，通过 new 也不行。

其实就是我们经常在 ES5 用到的方式。

```js
function foo() {
    var self = this; // lexical capture of this
    setTimeout( function(){
        console.log( self.a );
    }, 100 );
}
```

如果经常编写 this 风格的代码，但绝大多数情况都使用 self = this 或者箭头函数来否定 this 机制，那你或许应当：

- 只使用词法作用域并完全抛弃错误 this 风格的代码。
- 完全使用 this 风格，在必要时使用 bind，尽量避免使用 self=this 或者箭头函数。

# 3 对象

## 3.1 语法

对象可以通过两种形式定义：声明（文字）形式和构造形式。

文字语法：

```js
var myObj = {
    key: value
}
```

构造形式（非常少见）：

```js
var myObj = new Object();
myObj.key = value;
```

## 3.2 类型

一共六种主要类型（术语是语言类型）：

- string
- number
- boolean
- null
- undefined
- object

前5个是简单基本类型（primitive），本身不是对象。

> typeof null 返回的是 "object"，这是语言本身的一个 bug，null 本身是基本类型。出现这个 bug 的原因是，不同的对象在底层都表示为二进制。在 JavaScript 中，二进制前三位都是0，被判断为 object 类型。null 的二进制表示全是0。

有一种常见的错误说法是，JavaScript 中万物皆对象。

**内置对象**：某些名字和 primitive 类型一样，具体稍后介绍：

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

## 3.3 内容

### 3.3.4 复制对象

JavaScript 初学者最常见的问题之一就是如何复制一个对象。实际上比较复杂。

```js
function() anotherFunction { /* ... */ }
var anotherObject = {
    c: true
};
var anotherArray = [];
var myObject = {
    a: 2,
    b: anotherObject, // 引用，不是复制
    c: anotherArray, // 另一个引用
    d: anotherFunction
};
anotherArray.push(anotherObject, myObject);
```

那么，如何准确地表示 myObject 的复制呢？

首先，要判断是浅复制还是深复制。

浅复制中，新对象中的 `a:2` 是一份新的拷贝，但是，`b`，`c,`，`d` 只是引用，新对象也是引用。

如果是深复制，会有循环引用导致死循环。

对于 JSON 安全 的对象，有一种巧妙的：

```js
var newObj = JSON.parse(JSON.stringify(someObj));
```

相比于深复制，浅复制相对易懂问题少，Object.assign()，第一个参数就是目标对象，后面是源对象。它会遍历所有源对象的可枚举（enumerable）的自有键（owned key，后面介绍）并把它们复制（使用 `=` 操作符）到目标对象，最后返回目标对象。

> 下一节介绍『属性描述符』以及 `Object.defineProperty()` 的用法。要注意，由于 `Object.assign` 就是使用 `=` 操作符来赋值，所以源对象属性的一些特性（比如 `writable`）不会被复制到目标对象。

### 3.3.5 属性描述符

在 ES5 之前，JavaScript 没有提供检测属性特性的方法，比如判断属性是否是只读。

从 ES5 开始，所有的属性具备属性描述符。

```js
var myObject = {
    a: 2
};
Object.getOwnPropertyDescriptor(myObject, "a");
// {
// value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

所以，可不仅仅是一个 2。可写，可枚举，可配置。

我们可以使用 `Object.defineProperty()` 来添加一个新属性，或者修改一个已有属性（如果它是 `configurable`。

```js
var myObject = {};
Object.defineProperty(myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
});
myObject.a; //2
```

**1. Writable**

```js
var myObject = {};
 Object.defineProperty( myObject, "a", {
     value: 2,
     writable: false, // not writable!
     configurable: true,
     enumerable: true
 } );
 myObject.a = 3;
 myObject.a; // 2
```

如你所见，修改失败。如果是 "use strict"; 则会报错。

```js
myObject.a = 3; // TypeError
```

**2. Configurable**

表示是否可以使用 `defineProperty`。

```js
var myObject = {
    a: 2
};

myObject.a = 3;
myObject.a; // 3
Object.defineProperty(myObject, "a", {
    value: 4,
    writable: true,
    configurable: false, //       enumerable: true
});

myObject.a; // 4
myObject.a = 5;
myObject.a; // 5
Object.defineProperty(myObject, "a", {
    value: 6,
    writable: true,
    configurable: true,
    enumerable: true
}); // TypeError，不管是不是严格模式
```

```js
有一个小小的意外，即便属性是 configurable: false，依然可以 writable 由 true 改为 false。不能由 false 改为 true。
```

除了无法修改，`configurable: false` 还会禁止删除这个属性（`delete` 不会报错，静默失败）。

**3. Enumerable**

这个属性描述符（Property Descriptor）控制的是属性是否会出现在对象的属性枚举中。比如 `for .. in`。

### 3.3.6 不变性

1. 对象常量

结合 `writable: false` 和 `configurable: false`。

2. 禁止扩展

`Object.preventExtensions()`。

```js
var myObject = {
     a:2
 };
 Object.preventExtensions( myObject );
 myObject.b = 3;
 myObject.b; // undefined，严格模式是 TypeError
```

3. 密封

`Object.seal(..)`。实际上在一个现有对象上调用 `Object.preventExtensions()`，并将现有（由于了禁止扩展，也无法添加新的属性）属性的 `configurable: false`。密封之后，禁止扩展，也不能重新配置或者删除。但是可以修改属性的值。

4. 冻结

`Object.freeze(..)`。这个方法会在现有对象上调用 `Object.seal()`，并把所有对象的 `writable: false`。这样就无法修改他们的值。

这个方法是应用在对象上的级别最高的不可变性。（但是，这个对象引用的其他对象是不受影响的）。

### 3.3.7 [[Get]]

属性访问在实现时有一个微妙却非常重要的细节：

```js
var myObject = {
    a: 2
};
myObject.a; // 2
```

`myObject.a` 在 `myObject` 实际上是实现的 `[[Get]]` 操作（有点像函数调用，`[[Get]]()`）。

```js
var myObject = {
     a: undefined
 };
 myObject.a; // undefined
 myObject.b; // undefined
```

从返回值看好像没有区别，但是底层 `[[Get]]` 做了更复杂的处理。

稍后会介绍如何区分。

### 3.3.8 [[Put]]

### 3.3.9 Getter 和 Setter

```js
var myObject = {
    get a() {
        return 2;
    }
};

Object.defineProperty(myObject,
    "b": {
        get: function() {
            return this.a * 2
        },
        enumerable: true
    }
);

myObject.a; // 2
myObject.b; // 4
```

```js
var myObject = {
    get a() {
        return 2;
    }
};


myObject.a = 3;
myObject.a; // 2
```

```js
var myObject = {
    get a() {
        return this._a_;
    },
    set a(val) {
        this._a_ = val * 2;
    }
};
myObject.a = 2;
myObject.a; // 4

```

### 3.3.10 存在性

暂时不看。

# 4. 混合对象 『类』

实例化（instantiation），继承（inheritance）和（相对）多态（polymorphism）。

## 4.1 类理论

认为，好的设计是将数据和它相关的行为打包（封住）。
































