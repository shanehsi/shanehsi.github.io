title: Wallaby.js Configuration file
date: 2016-04-16 13:00:01
tags:
---

by default, wallaby.js is using PhantomJs to run tests.

默认是在 PhantomJs 跑测试（就是 browsers 啦）。

通过 `env` 来跑在 node 或者 [latest Chromium/V8 via Electron](https://wallabyjs.com/docs/integration/electron.html)

```js
env: {
  type: 'node'
}

 env: {
  kind: 'electron'
}
```

# API

```js
wallaby
    .localProjectDir // string 返回 project 当前的目录 
    .projectCacheDir // string project 的 cache 目录 也就是， wallaby 会把文件从 `files` `tests` 复制到 cache 目录，跑的是 cache 里的
    .compilers // 访问 TypeScript Babel compilers https://wallabyjs.com/docs/config/compilers.html
    .defaults 
```

## 快捷链接

- 配置 compilers [TypeScript](https://wallabyjs.com/docs/integration/typescript.html) 和 [ES6/ES7](https://wallabyjs.com/docs/integration/es-next.html)

- 配置 test framework 跑 test 之前设置下环境 [setup/bootstrap](https://wallabyjs.com/docs/config/bootstrap.html)

- 配置 [env](https://wallabyjs.com/docs/config/runner.html)，node.js 其他 version 的 phantomjs，electron。

- [preprocessor](https://wallabyjs.com/docs/config/preprocessors.html)，比如处理某些 template 为 javascript 来 load tests

## test framework

```js
module.exports = function (wallaby) {
  return {
    files: [
      'src/**/*.js'
    ],

    tests: [
      'test/**/*Spec.js'
    ],

    testFramework: 'mocha'
  };
};
```

## delays

使用默认的

## debug

输出详细信息，一般不用，报 bug 的时候可以用。

# Files and tests

## Files

注意：

- 一定要写全
    - 比如如果你在 js 里 load 了 json，要写上
    - 如果你 require 了其他的 js，要写上
    - 例外
        - node_modules 目录（因为 wallaby 不会 cache 它们，就用 local 的）
        - 如果用了 webpack 等 bundler（除非用的是 global）
        - 用了 中间件 [Middleware](https://wallabyjs.com/docs/config/middleware.html)???
- 注意根据依赖设置前后顺序
- 可以是 js，也可以是 css， image json 等任何类型


## Tests

略过

## File object

pattern

```js
files: [
      'src/**/*.js',
      // is the same as
      { pattern: 'src/**/*.js', instrument: true, load: true, ignore: false }
    ],
```

其中 `ignore` 的快捷写法是 `!` 。

```js
{
  "files": [
    "src/**/*.js",
    "!src/**/*Spec.js"
  ],

  "tests": [
    "src/**/*Spec.js"
  ]
}
```

`load` 只是否被 sandbox HTML 加载（通过 `<script>` ）。如果用了 Webpack 就不需要了。

## Overriding defaults

```js
module.exports = function (wallaby) {
  wallaby.defaults.files.instrument = false;
  return {
    files: [
      'libs/lib1.js',
      'libs/lib2.js',
      'libs/lib3.js',
      'libs/lib4.js',
      { pattern: 'src/**/*.js', instrument: true }
    ],

    tests: [
      'test/**/*Spec.js'
    ]
  };
};
```

# Runner 

PhantomJs, Chromium/V8 via Electron and node.js

默认的是 By default, PhantomJs (1.9.8)

具体的是细节，默认的和 node 就可以 

[更多](https://wallabyjs.com/docs/config/runner.html)

# Compilers

这个和 preprocessors 会混淆。

preprocessors 也能 transform，然后给到 test runner，但是如果你要有 code coverage，首先 wallaby 需要在任意的 preprocessors 前，intstrument 它们。

也就是说 wallaby 需要了解这些 dialect。

Wallaby.js 本身就支持 ES6，JSX。其他的需要配置。

内置了三款：

> Wallaby.js has three built-in compilers: TypeScript, CoffeeScript and Babel.

如果用了 Babel compiler ，就不要用 Babel preprocessor。

如果只用了 ES6，没用到 ES7，可以 keep using Babel preprocessor without Babel compiler。

但是如果用到了 TypeScript，则必须用 compiler。preprocessors for TypeScript and CoffeeScript are not required

连配置 compiler 的语法和 preprocessors 都差不多。

```js
compilers: {
  '**/*.js': wallaby.compilers.babel({
    // babel options
    // like `stage: n` for Babel 5.x or `presets: [...]` for Babel 6
    // (no need to duplicate .babelrc, if you have it, it'll be automatically loaded)
  }),

  '**/*.ts': wallaby.compilers.typeScript({
    // TypeScript compiler specific options
    // https://github.com/Microsoft/TypeScript/wiki/Compiler-Options
    // (no need to duplicate tsconfig.json, if you have it, it'll be automatically used)
  }),

}
```

## Preprocessors

我的需求已经不需要配置 Preprocessors 了。

[链接](https://wallabyjs.com/docs/config/preprocessors.html)

- compilers
- instrumentation
- preprocessors
- postprocessor
- test run

It is not recommended to keep any state between function calls because preprocessors and compilers may run in different node processes.

Postprocessor is a function that runs for every batch of file changes after all compilers and preprocessors. It has access not to just one file, but to **all** files (compiled and preprocessed, as well as **original ones**). It is also guaranteed to always run in the **same node process**, so it can **keep some required state**.

Considering all of the above, postprocessor is an ideal place for something like a module bundler, or anything else that needs to  **keep some state between** test runs and have access to all files at the same time.

a function that takes a single argument and is supposed to return a **promise**

```js
module.exports = function () {
   return {
     files: [
       'src/**/*.js'
     ],

     tests: [
       'test/**/*Spec.js'
     ],

     // @param wallaby wallaby.js context
     postprocessor: function (wallaby) {
       return new Promise(
         function (resolve, reject) {
            try {
              // postprocessor work here
              // ...

              // resolve when the work is done
              // or reject on errors
              resolve();

            } catch (err) {
              reject(err);
            }
          });
     }
   };
 };
```


具体的可以看下 webpack 的示例

[wallaby-webpack/index.js](https://github.com/jeffling/wallaby-webpack/blob/master/index.js)

## Setup

setup，alias 是 bootstrap。

```js
 module.exports = function () {
   return {
     files: [
       'src/**/*.js'
     ],

     tests: [
       'test/**/*Spec.js'
     ],

     setup: function (wallaby) {
       // wallaby.testFramework is jasmine/QUnit/mocha object
       wallaby.testFramework.ui('tdd');

       // you can access 'window' object in a browser environment,
       // 'global' object or require(...) something in node environment 比如 window 和 global
     }
   };
 }
```








