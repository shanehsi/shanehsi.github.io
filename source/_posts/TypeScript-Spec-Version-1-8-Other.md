title: TypeScript Spec Version 1.8 - Other
date: 2016-02-26 08:21:39
tags:
---

## [Type Aliases](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#3.10)

```js
type StringOrNumber = string | number;  
type Text = string | { text: string };  
type NameLookup = Dictionary<string, Person>;  
type ObjectStatics = typeof Object;  
type Callback<T> = (data: T) => void;  
type Pair<T> = [T, T];  
type Coordinates = Pair<number>;  
type Tree<T> = T | { left: Tree<T>, right: Tree<T> };
```

Interface types have many similarities to type aliases for object type literals, but since interface types offer more capabilities they are generally preferred to type aliases. For example, the interface type

```js
interface Point {  
    x: number;  
    y: number;  
}
```

could be written as the type alias

```js
type Point = {  
    x: number;  
    y: number;  
};
```

However, doing so means the following capabilities are lost:

- An interface can be named in an extends or implements clause, but a type alias for an object type literal cannot. 可以用在 extend 和 implements 子句中
- An interface can have multiple merged declarations, but a type alias for an object type literal cannot. interface 是 open-end，同名的会被 merge。 

## [ Excess Properties](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#3.11.5)

```js
interface InputElement {  
    name: string;  
    visible?: boolean;  
    [x: string]: any;            // Allow additional properties of any type  
}

var address: InputElement = {  
    name: "Address",  
    visible: true,  
    help: "Enter address here",  // Allowed because of index signature  
    shortcut: "Alt-A"            // Allowed because of index signature  
};
```


