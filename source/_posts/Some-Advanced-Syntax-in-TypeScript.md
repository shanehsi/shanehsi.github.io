title: TypeScript 的一些高级语法和使用场景
date: 2016-02-25 16:49:34
tags:
---

直接从 [Microsoft/TypeScript](https://github.com/Microsoft/TypeScript/) 和 [angular](https://github.com/angular/angular) 找。也会从 RxJS 中找。


## Namespace

```js
namespace ts.formatting {
    ...
}
```

## export 

```js
export interface TextRange {
    pos: number;
    end: number;
}
```

## extends

```js
export interface TextRangeWithKind extends TextRange {
    kind: SyntaxKind;
}
```

## generics 和声明 

```js
export interface Map<T> {
    [index: string]: T;
}
```

注意：`[index: string]` 这种声明 Dictionary 的方法。

```js
export interface SymbolTable {
    [index: string]: Symbol;
}
```

## 高级类型

```js
export type Path = string & { __pathBrand: any };
```

## enums

```js
export const enum NodeFlags {
    None =               0,
    Export =             1 << 0,  // Declarations
    Ambient =            1 << 1,  // Declarations
    Public =             1 << 2,  // Property/Method
    Private =            1 << 3,  // Property/Method
    Protected =          1 << 4,  // Property/Method

    Modifier = Export | Ambient | Public | Private | Protected | Static | Abstract | Default | Async,
    AccessibilityModifier = Public | Private | Protected,
    BlockScoped = Let | Const,
}
```

和不加 `const` 的区别：

```js
export enum ExitStatus {
    Success = 0,
    DiagnosticsPresent_OutputsSkipped = 1,
    DiagnosticsPresent_OutputsGenerated = 2,
}
```

## type

```js
export type EntityName = Identifier | QualifiedName;
```

## 第一看到 toString

```js
///<reference path='references.ts' />

/* @internal */
namespace ts.formatting {
    export class Rule {
        constructor(
            public Descriptor: RuleDescriptor,
            public Operation: RuleOperation,
            public Flag: RuleFlags = RuleFlags.None) {
        }

        public toString() {
            return "[desc=" + this.Descriptor + "," +
                "operation=" + this.Operation + "," +
                "flag=" + this.Flag + "]";
        }
    }
}
```

## implements

```js
export class Observable<T> implements CoreOperators<T>  {
}
```

