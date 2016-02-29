title: JavaScript new constructor this 来自知乎
date: 2016-02-04 10:22:19
tags:
---

知乎链接：[JavaScript 中对象的 constructor 属性的作用是什么？](https://www.zhihu.com/question/19951896)

主要解释：`constructor`。

```js
var a,b;
(function(){
  function A (arg1,arg2) {
    this.a = 1;
    this.b=2; 
  }

  A.prototype.log = function () {
    console.log(this.a);
  }
  a = new A();
  b = new A();
})()
a.log();
// 1
b.log();
// 1
```

a 是 A 的实例，a 的 constructor 就是 A

```js
> a.constructor
< function A(arg1,arg2) {
    this.a = 1;
    this.b=2; 
  }
```

所以，可以这样给『类A』增加新方法：

```
// a.constructor.prototype 在chrome,firefox中可以通过 a.__proto__ 直接访问
a.constructor.prototype.log2 = function () {
  console.log(this.b)
}

a.log2();
// 2
b.log2();
// 2
```

继续：

```js
> a.constructor.prototype === a.__proto__
< true
```

```js
> a.constructor.prototype.constructor === a.constructor
< true
```

知乎链接：[如何理解 JavaScript 中的 this 关键字？](https://www.zhihu.com/question/19636194)

详细：[JavaScript中的对象查找](http://otakustay.com/object-lookup-in-javascript/)。
