title: TypeScript Spec Version 1.8 - Ambient Declarations (Object Types)
date: 2016-02-26 07:32:36
tags:
---

## Ambient Declaration

[Ambient Declaration](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#11-ambient-declarations)

An ambient declaration introduces a variable into a TypeScript scope, but has zero impact on the emitted JavaScript program. 

引入一个变量，仅在 TypeScript 范围，不会生成 JavaScript 代码。

```js
declare var document;  
document.title = "Hello"
```

Section 1.3 provides a more extensive example of how a programmer can add type information for jQuery and other libraries.

## Object Types

[1.3 Object Types](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#1.3)

TypeScript programmers use object types to declare their expectations of object behavior. 

适用 object types 声明对象的行为。

主要接着 ambient declaration，对 jQuery 的声明。

先了解下 object tyeps：

### object type literal

```js
var MakePoint: () => {  
    x: number; y: number;  
};
```

### interface

```js
interface Friend {  
    name: string;  
    favoriteColor?: string;  
}

function add(friend: Friend) {  
    var name = friend.name;  
}

add({ name: "Fred" });  // Ok  
add({ favoriteColor: "blue" });  // Error, name required  
add({ name: "Jill", favoriteColor: "green" });  // Ok
```

### jQuery Ambient Declaration

结果：

- `$` 有 methods，fields
- `$` 本身也是 function
- `$` 作为 function 还可以被 overload

```js
interface JQuery {  
    text(content: string);  
}  

interface JQueryStatic {  
    get(url: string, callback: (data: string) => any);     
    (query: string): JQuery;  
}

declare var $: JQueryStatic;

$.get("http://mysite.org/divContent",  
      function (data: string) {  
          $("div").text(data);  
      }  
);
```

The 'JQueryStatic' interface references another interface: 'JQuery'. 

Finally, the 'JQueryStatic' interface contains a bare function signature

```js
(query: string): JQuery;
```

The bare signature indicates that instances of the interface are callable. 接口的实例是可以被调用的。

This example illustrates that TypeScript function types are just special cases of TypeScript object types. Specifically, function types are object types that contain one or more call signatures. Function types 是 object types 的一种，即包含一个以上的 call signature。

Function types 也可以用 literal 表示。

```js
var f: { (): string; };  
var sameType: () => string = f;     // Ok  
var nope: () => number = sameType;  // Error: type mismatch
```

重载。To specify multiple behaviors, TypeScript supports overloading of function signatures in object types. 

```js
interface JQueryStatic {  
    get(url: string, callback: (data: string) => any);     
    (query: string): JQuery;
    (ready: () => any): any; // overload
}
```
