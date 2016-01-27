title: Import JavaScript files in TypeScript
tags:
---

参考：[TypeScript with KnockoutJS](http://stackoverflow.com/questions/12689716/typescript-with-knockoutjs/12692174#12692174)

TypeScript 具有良好的教学作用。目前初步的想法是，直接 import 现有的 JavaScript 代码，然后对其进行封装。TypeScript 的类型系统，会提供完备的接口定义写法。

这其实是一种组合式的重用方式。对于可用的直接 import 进来，对于不可用的进行重写。

那如何在 TypeScript 文件中 import Javascript 呢？

TypeScript 提供的特性就是写 Definition Files。在写的时候参考 [DefinitelyTyped](https://github.com/borisyankov/DefinitelyTyped) 的写法，先进行模仿。

文件可以直接写在相应 js 文件的旁边，[name].d.ts。

具体在写的时候，如果有新的问题或思路进行补充。
