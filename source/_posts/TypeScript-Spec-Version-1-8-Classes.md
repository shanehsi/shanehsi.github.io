title: TypeScript Spec Version 1.8 - Classes
date: 2016-02-26 07:54:14
tags:
---

本身，JavaScript practice has two very common design patterns: the module pattern and the class pattern.

简单来说，Roughly speaking, the module pattern uses closures to hide names and to encapsulate private data, while the class pattern uses prototype chains to implement many variations on object-oriented inheritance mechanisms. 

module 模式是利用闭包。

class 模式用原型链来实现 OO 继承机制。

> 但是注意，EC2015 引入了 module 机制和 module pattern 实现的原理不同，为了防止混淆，在 TypeScript 中，module pattern 对应的是 namespace。

[Classes](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#16-classes)

```js
class BankAccount {  
    balance = 0;  
    deposit(credit: number) {  
        this.balance += credit;  
        return this.balance;  
    }  
}  
```

生成的 JavaScript（TypeScript 的目标是生成 consistent，idiomatic 的 JavaScript 代码）：

```js
var BankAccount = (function () {  
    function BankAccount() {  
        this.balance = 0;  
    }  
    BankAccount.prototype.deposit = function(credit) {  
        this.balance += credit;  
        return this.balance;  
    };  
    return BankAccount;  
})();
```

OK，回过头看 `class BankAccount`，它做了：

- creates a variable named 'BankAccount'
- whose value is the constructor function for 'BankAccount' instances.
-  also creates an instance type of the same name. 

关于 instance type，如果用接口可以这样写：

```js
interface BankAccount {  
    balance: number;  
    deposit(credit: number): number;  
}
```

If we were to write out the function type declaration for the 'BankAccount' constructor variable, it would have the following form.

即，这是关于第一点，BankAccount 的值是一个 constructor 函数。

```js
var BankAccount: new() => BankAccount;
```

The function signature is prefixed with the keyword 'new' indicating that the 'BankAccount' function must be called as a constructor. It is possible for a function's type to have both call and constructor signatures. For example, the type of the built-in JavaScript Date object includes both kinds of signatures.

在 constructor 里做些初始化：

```js
class BankAccount {  
    balance: number;  
    constructor(initially: number) {  
        this.balance = initially;  
    }  
    deposit(credit: number) {  
        this.balance += credit;  
        return this.balance;  
    }  
}
```

简化版本：

```js
class BankAccount {  
    constructor(public balance: number) {  
    }  
    deposit(credit: number) {  
        this.balance += credit;  
        return this.balance;  
    }  
}
```

The 'public' keyword denotes that the constructor parameter is to be retained as a field. 

Public is the default accessibility for class members, but a programmer can also specify private or protected accessibility for a class member.

Section 8 provides additional information about classes.

继承。

```js
class CheckingAccount extends BankAccount {  
    constructor(balance: number) {  
        super(balance);  
    }  
    writeCheck(debit: number) {  
        this.balance -= debit;  
    }  
}
```

生成的核心代码： In the emitted JavaScript code, the prototype of 'CheckingAccount' will chain to the prototype of 'BankAccount'.

```js
__extends(CheckingAccount, _super);

var __extends = (this && this.__extends) || function(d, b) {
    for (var p in b)
        if (b.hasOwnProperty(p)) d[p] = b[p];

    function __() { this.constructor = d; }
    d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
};

```

更多：

Since instance member variable initializers are equivalent to assignments to properties of this in the constructor, the example

```js
class Employee {  
    public name: string;  
    public address: string;  
    public retired = false;  
    public manager: Employee = null;  
    public reports: Employee[] = [];  
}
```

等价：

```js
class Employee {  
    public name: string;  
    public address: string;  
    public retired: boolean;  
    public manager: Employee;  
    public reports: Employee[];  
    constructor() {  
        this.retired = false;  
        this.manager = null;  
        this.reports = [];  
    }  
}
```

更多的见 [Classes](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#8)

namespace：


## 1.10 Namespaces

产生背景：

TypeScript enforces encapsulation of implementation in classes at design time (by restricting use of private and protected members), but cannot enforce encapsulation at runtime because all object properties are accessible at runtime. 

JavaScript 怎么做到：

In JavaScript, a very common way to enforce encapsulation at runtime is to use the module pattern: encapsulate private fields and methods using closure variables.

其他好处：

The module pattern can also provide the ability to introduce namespaces, avoiding use of the global namespace for most software components.

在 JavaScript 是这样：

```js
(function(exports) {  
    var key = generateSecretKey();  
    function sendMessage(message) {  
        sendSecureMessage(message, key);  
    }  
    exports.sendMessage = sendMessage;  
})(MessageModule);
```

在 TypeScript 这么做 

TypeScript namespaces provide a mechanism for succinctly expressing the module pattern. In TypeScript, programmers can combine the module pattern with the class pattern by nesting namespaces and classes within an outer namespace.

```js
namespace M {  
    var s = "hello";  
    export function f() {  
        return s;  
    }  
}

M.f();  
M.s;  // Error, s is not exported
```

