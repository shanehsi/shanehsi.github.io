title: Wallaby.js TypeScript Node Browser
date: 2016-04-16 16:20:07
tags:
---

## 结合 webpack 的

用到了 wallaby-webpack postprocessor 和 TypeScript。

package.json

```js
"devDependencies": {
    "chai": "^2.1.2",
    "wallaby-webpack": "*",
    "webpack": "^1.7.3",
    "ts-loader": "^0.5.5",
    "typescript": "*"
  }
```

tsconfig.json

```js
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es5",
    "noEmit": true
  },
  "exclude": [
    "node_modules"
  ],
  "files": [
    "typings/tsd.d.ts",
    "typings/assertion-error/assertion-error.d.ts",
    "typings/chai/chai.d.ts",
    "typings/jasmine/jasmine.d.ts",
    "typings/node/node.d.ts",
    "test/personSpec.ts",
    "src/Person.ts"
  ]
}

```

wallaby.js

```js
var wallabyWebpack = require('wallaby-webpack');
var webpackPostprocessor = wallabyWebpack({});

module.exports = function () {

  return {
    files: [
      // load 为 false，是因为用到了 webpack。
      { pattern: 'src/**/*.ts', load: false }
    ],

    tests: [
      { pattern: 'test/**/*Spec.ts', load: false }
    ],

    // 结合 webpack
    postprocessor: webpackPostprocessor,

    bootstrap: function () {
      // required to trigger test loading
      window.__moduleBundler.loadTests();
    }
  };
};

```



## 纯 TypeScript 

安装下 jasmine 的 typings

```js
module.exports = function (w) {

  return {
    files: [
      'src/*Browser.ts'
    ],

    tests: [
      'test/*BrowserSpec.ts'
    ]
  };
};

```

然后是这么引用 Jasmine 的

```js
///<reference path="../typings/jasmine/jasmine.d.ts"/>
///<reference path="../src/sayingsBrowser.ts"/>

describe('Sayings Greeter', () => {
    it('should greet', () => {
        var greeter = new Sayings.Greeter('John');
        expect(greeter.greet()).toBe('Hello, John');
    });
});

```


纯的 TypeScript。基本没啥这么写的。要么就用 webpack 来写。

```js
module.exports = function (w) {

  return {
    files: [
      'src/*Node.ts'
    ],

    tests: [
      'test/*NodeSpec.ts'
    ],

    env: {
      type: 'node'
    },
    
    // you may remove the setting if you have a tsconfig.json file where the same is set
    compilers: {
      '**/*.ts': w.compilers.typeScript({module: 'commonjs'})
    }

    // By default TypeScript compiler renames .ts files to .js files.
    // If you'd like to not do it and for example use your own renaming strategy,
    // you may pass 'noRename' option to TS compiler
    //  '**/*.ts': w.compilers.typeScript({ noRename: true })
    // and may use preprocessors to rename files the way you like:
    //preprocessors: {
    //  '**/*.ts': file => file.rename(file.path + '.js').content
    //}

  };
};

```

