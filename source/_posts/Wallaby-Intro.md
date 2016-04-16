title: Wallaby Intro
date: 2016-04-16 12:02:06
tags:
---

# 介绍 Wallaby.js

## What about a list of main features?

- continuous testing 持续测试，性能不错
- browser code unit testing，测试 browser，是依赖（PhantomJs, Electron or node.js）
- node.js unit testing
- Supports many testing frameworks（Jasmine 默认）
- 支持 ES6 and React JSX
- 支持 TypeScript, CoffeeScript, and ES7
- 支持  Webpack and Browserify

其他的包括：
- 可扩展 preprocessors, compilers and more
- 性能（只跑变动的，并行的跑，跑选择的）
- 实时覆盖率
- 在编辑器 inline 显示 expectations, errors and console.log messages
- 显示跑的 test 的变量的 data
- test 执行的截图？？

## 和 Karma, Mocha 的区别

其他的 runner，需要手动跑，配置 watch，跑所有的 tests，test 的结果是显示在其他的地方，需要切换。

## 文章链接

- 所有文档 [地址](http://wallabyjs.com/docs/)
- 切换不同的 test framework [favorite testing framework](https://wallabyjs.com/docs/integration/overview.html#supported-testing-frameworks)，主要考虑到如何测试 async/await, promise, observable 这些。Jasmine 似乎不够强大。选择更通用的 mocha 吧。Jasmine 的文档写的也是一绝（不好读）。mocha 肯定有解决 async/await 的方法。社区支持很好。

默认的 test framework：

> By default, wallaby.js is using jasmine for browser and mocha for node.

Todo list:

- [ ]  wallaby

    - [ ] webpack

    - [ ] typescript

    - [ ] mocha

    - [ ] async/await

    - [ ] promise

    - [ ] rx observable

    - [ ] node (express)

- [ ] koa 2

    - [ ] wallaby

- 测试 React，如何集成 tsx？
- 测试 node [Node.js](https://wallabyjs.com/docs/integration/node.html)，特别是 typescript。
- [测试 TypeScript](https://wallabyjs.com/docs/integration/typescript.html)
- [集成 Babel](https://wallabyjs.com/docs/integration/es-next.html)
- [集成 webpack](https://wallabyjs.com/docs/integration/webpack.html)
- [支持 preprocessors](https://wallabyjs.com/docs/config/preprocessors.html)

