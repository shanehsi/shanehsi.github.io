title: TypeScript typings
date: 2016-03-30 11:57:29
tags:
---

[TypeScript cannot find name IPromise in RxJS definition](http://stackoverflow.com/questions/35919693/typescript-cannot-find-name-ipromise-in-rxjs-definition)

The RxJS type definition from DefinitelyTyped does appear to be outdated. Instead, use the type definition provided by the npm package.

```
typings install --save --ambient npm:rx/ts/rx.all.d.ts
```



