title: Wallaby ES7
date: 2016-04-16 12:28:53
tags:
---

先看看 package.json

```js
{
  "devDependencies": {
    "babel-core": "^6.4.5",
    "babel-polyfill": "^6.3.14",
    "babel-preset-es2015": "^6.3.13",
    "babel-preset-stage-0": "^6.3.13"
  },
}
```

其中 babel-polyfill 上次用到过，对于支持 async/await 必须：

```js
  "dependencies": {
    "core-js": "^2.1.0",
    "babel-regenerator-runtime": "^6.3.13",
    "babel-runtime": "^5.0.0"
  },
```


[core-js](https://github.com/zloirock/core-js) 是一个 polyfills 集合。

具体参见：[Polyfill](https://babeljs.io/docs/usage/polyfill/)

babel-polyfill 和 regenerator-runtime 差别不大。具体差别参考上面的链接和源码。

在看下 `.babelrc` 配置：

```js
{
    "presets": ["es2015", "stage-0"]
}
```

再看下 wallaby.js 配置

```js
module.exports = function (wallaby) {
  return {
    files: [
      {pattern: 'node_modules/babel-polyfill/dist/polyfill.js', instrument: false},
      'src/**/*.js'
    ],

    tests: [
      'test/**/*Spec.js'
    ],

    compilers: {
      '**/*.js': wallaby.compilers.babel()
    }
  };
};

```

其中加入这句话，其中，`instrument` 为 `false`。

```js
 {pattern: 'node_modules/babel-polyfill/dist/polyfill.js', instrument: false},
```

链接 [Configuration file: Files and tests](https://wallabyjs.com/docs/config/files.html)

关于 `instrument`：

> The instrument boolean property (**true by default**) determines whether the file is instrumented. Setting the property to false disables the file **code coverage reporting** and prevents the file changes from **triggering automatic test execution**. The setting should normally be set to false for libraries, utils and other rarely changing files. Using the setting makes wallaby.js run your code **faster**, as it doesn’t have to perform unnecessary work.

还有一个是 `compilers`。链接 [Compilers](https://wallabyjs.com/docs/config/compilers.html)

写到这，我觉得要把 Wallaby/Configuration file 从头到尾看一遍。



