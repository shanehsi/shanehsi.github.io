title: 再读 TypeScript Handbook 2016年02月25日
date: 2016-02-25 17:30:23
tags:
---

## Basic Types

Array

```js
var list:number[] = [1, 2, 3];
var list:Array<number> = [1, 2, 3];
```

## Interfaces

Function Types

```js
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

Array Types - Array types have an 'index' type that describes the types allowed to index the object, along with the corresponding return type for accessing the index.

```js
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

There are two types of supported index types: string and number. It is possible to support both types of index, with the restriction that the type returned from the numeric index must be a subtype of the type returned from the string index.

While index signatures are a powerful way to describe the array and 'dictionary' pattern, they also enforce that all properties match their return type. In this example, the property does not match the more general index, and the type-checker gives an error:

```js
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
}
```

Class Types - Implementing an interface

```js
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

## Classes

```js
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

Parameter properties - 对上的简化写法，The properties let you can create and initialize a member in one step.

```js
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

Accessors - TypeScript supports getters/setters as a way of intercepting accesses to a member of an object.

```js
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }
    
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

## Modules
