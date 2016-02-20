title: 读 TypeScript 博客 Announcing TypeScript 1.7
date: 2016-01-13 10:10:24
tags:
---

更新：

- 对于 ES6 targets，默认打开 async/await
- polymorphic 'this' typing
- ES2016 的 exponentiation 语法
- ES6 module targeting

## Async/Await for ES6 targets

[Asyc functions](http://tc39.github.io/ecmascript-asyncawait/)，需要 [ES6 generator](http://www.ecma-international.org/ecma-262/6.0/#sec-generator-function-definitions) 支持（比如 node.js v4 及以上）。

函数通过 `async` 关键词指定它为一个 asynchronous function。

`await`关键词用来停止执行（stop execution），直到 `async` 函数的 promise 已经 fulfilled。

```TypeScript
"use strict";
// printDelayed is a 'Promise<void>'
async function printDelayed(elements: string[]) {
    for (const element of elements) {
        await delay(200);
        console.log(element);
    }
}
 
async function delay(milliseconds: number) {
    return new Promise<void>(resolve => {
        setTimeout(resolve, milliseconds);
    });
}
 
printDelayed(["Hello", "beautiful", "asynchronous", "world"]).then(() => {
    console.log();
    console.log("Printed every element!");
});
```


对于目前对 async/await 的实现，参考上一篇[博客](http://blogs.msdn.com/b/typescript/archive/2015/11/03/what-about-async-await.aspx)。

## Polymorphic `this` Typing

背景见 [issue](https://github.com/Microsoft/TypeScript/issues/229)。

这是 TypeScript 1.7 新增的类型。

`this` 类型可以用在 classes 和 interfaces 里，表示某种类型，它是 containing type 的 subtype（而不是 containing type）。

关于多态这个计算机科学词汇，这里加深下理解。

我们先以 [C#中的多态性为例](https://msdn.microsoft.com/zh-cn/library/ms173152.aspx) 理解：

多态指两个方面：

- 运行时，在 方法参数、集合或数组等位置，派生类的对象可以作为基类的对象处理。即，对象的声明类型不再与运行时相同。
- 基类可以定义并实现虚方法，派生类可以重写（override）这些方法，即派生类提供自己的定义和实现。运行时，CLR 查找对象的**运行时**定义，调用虚方法的重写方法。即，你可以调用基类的方法，但执行的是派生类版本。

这些是在 C# 中的实践，再来看下[维基百科](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%9E%8B_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)的定义：

> 多态，是指计算机程序运行时，相同的消息可能会送给多个不同的类之对象，而系统可依据对象所属类，引发对应类的方法，而有不同的行为。
> 多态也可定义为“一种将不同的特殊行为和单个泛化记号相关联的能力”。

这里先重点关于下**动态多态（dynamic polymorphism）**：通过类继承机制和虚函数机制生效于运行期。可以优雅地处理异质对象集合，只要其共同的基类定义了虚函数的接口。也被称为子类型多态（Subtype polymorphism）或包含多态（inclusion polymorphism）。在面向对象程序设计中，这被直接称为多态。

另外提及一点，静态多态中的参数化多态（Parametric polymorphism），把类型作为参数的多态。在面向对象程序设计中，这被称作[泛型编程](https://zh.wikipedia.org/wiki/%E6%B3%9B%E5%9E%8B)。


现在回头再看下 TypeScript 1.7 引入的多态 `this` 类型：

> `this` 类型可以用在 classes 和 interfaces 里，表示某种类型，它是 containing type 的 subtype（而不是 containing type）。

这个特性可以帮助容易写出某些 patterns，比如：hierachical fluent APIs，层级流式 API。

```TypeScript
interface Model {
    setupBase(): this;
}
 
interface AdvancedModel extends Model {
    setupAdvanced(): this;  // 返回值是 this，这个 this 既可以是指本身，也可以指基类
}

declare function createModel(): AdvancedModel;
newModel = newModel.setupBase().setupAdvanced(); // fluent style works
```

> 问：TypeScript 中的 `declare` 关键词是指？

关于 `this` 的更多信息，参考 [TypeScript Wiki](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#this-typing)。

为了支持这个特性，TypeScript 1.7 对从 `this` 推导（inferring）type 做了些许变更。大致一些[潜在的 breaking changes](https://github.com/Microsoft/TypeScript/wiki/Breaking-Changes#TypeScript1.7)：

>In a class, the type of the value this will be inferred to the this type, and subsequent assignments from values of the original type can fail. As a workaround, you could add a type annotation for this. A code sample with recommended work around, along with a list of other potentially breaking changes is available at GitHub.

## ES6 Module Emitting

Node.js v4+，支持很多 ES6 特性，但是不支持 ES6 modules，可以如下配置，适应 Node.js v4+ 的 runtime。

```json
//tsconfig.json targeting node.js v4 and beyond
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es6"
    }
}
```

## ES7 Exponentiation

```TypeScript
let squared = 2 ** 2;  // same as: 2 * 2
let cubed = 2 ** 3;  // same as: 2 * 2 * 2
let num = 2;
num **= 2; // same as: num = num * num;
```

和 `Math.pow()` 说 Byebye。



