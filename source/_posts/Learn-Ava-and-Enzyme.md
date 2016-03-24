title: 学习 Ava 和 Enzyme
date: 2016-03-23 11:06:23
tags:
---

# Ava

[github](https://github.com/sindresorhus/ava)。

ava 配置很简单

```js
{
  "name": "awesome-package",
  "scripts": {
    "test": "ava"
  },
  "devDependencies": {
    "ava": "^0.11.0"
  }
}
```

```js
import test from 'ava';

test('foo', t => {
    t.pass();
});

test('bar', async t => {
    const bar = Promise.resolve('bar');

    t.is(await bar, 'bar');
});
```

跑测试：

```
npm test
npm test -- --watch
```

更多的配置在 pakcage.json 中。

```js
{
  "ava": {
    "files": [
      "my-test-folder/*.js",
      "!**/not-this-file.js"
    ],
    "source": [
      "**/*.{js,jsx}",
      "!dist/**/*"
    ],
    "match": [
      "*oo",
      "!foo"
    ],
    "failFast": true,
    "tap": true,
    "require": [
      "babel-register"
    ],
    "babel": "inherit"
  }
}
```

关于对 ES2015 的支持。Ava 是内置了 Babel 6，Ava 使用的配置如下：

```js
{
  "presets": [
    "es2015",
    "stage-2",
  ],
  "plugins": [
    "espower",
    "transform-runtime"
  ]
}
```

可以修改的：

```js
{
    "ava": {
         "babel": {
             "presets": [
                    "es2015",
                    "stage-0",
                    "react"
             ]
         }
    },
}
```

并且可以使用 `inherit` 来使用 `.babelrc` 的配置，不用再为了 ava 在 transpile 一次。

```js
{
    "babel": {
        "presets": [
            "es2015",
            "stage-0",
            "react"
        ]
    },
    "ava": {
        "babel": "inherit",
    },
}
```

但不管怎样，Note that AVA will always apply the `espower` and `transform-runtime` plugins.

## 文档

建议使用 async function

We highly recommend the use of async functions. They make asynchronous code concise and readable, and they implicitly return a promise so you don't have to.

```js
test(async function (t) {
    const value = await promiseFn();
    t.true(value);
});

// async arrow function
test(async t => {
    const value = await promiseFn();
    t.true(value);
});
```

下面的句子看下，涉及到 spec 文件的放置目录。

Test files are run from their current directory, so process.cwd() is always the same as __dirname. You can just use relative paths instead of doing path.join(__dirname, 'relative/path').

其他的 API 用到再看。

## 测试浏览器

依赖 `jsdom`。

[Setting up AVA for browser testing](https://github.com/sindresorhus/ava/blob/master/docs/recipes/browser-testing.md)



# Enzyme

文章介绍，[Enzyme: JavaScript Testing utilities for React](https://medium.com/airbnb-engineering/enzyme-javascript-testing-utilities-for-react-a417e5e5090f#.sjli8b8zm)

[github](https://github.com/airbnb/enzyme/)

Airbnb open sourced Enzyme, a JavaScript library for testing React components. 

Historically, testing UI has been hard to accomplish for a variety of reasons, but using React removes a lot of these hurdles. We hope that enzyme does a good job removing the remaining ones!

测试 UI 很难，React 帮我们除去了大多的障碍，希望 enzyme 等除去剩下的。

## Declarative UIs are Testable UIs

Pure functions (and thus React components) are much easier to test because they simply return a description for what UI of the component should look like, given some application state, rather than actually mutating the UI and having side-effects. This “description” is known as a “Virtual DOM” and is a tree-like data structure.

- Pure Function 易测试，input -> output，没有 side effects
- 测试 Virutal DOM，a tree-like data structure.

Making assertions on the state of a React render tree can include a lot of boilerplate code and is hard to read, which detracts from the value of the test. Moreover, directly asserting on the resulting tree can strongly couple your tests to implementation details that end up making your tests extremely fragile.

- 测试 React 的 render tree 会有很多 boilerplate 代码。难以阅读，也就降低了 test 的价值。
- 直接 assert resulting tree 会将 test 和实现细节强耦合，最终 fragile。

Enzyme makes asking questions about the rendered output of your React components easy and intuitive by providing a fluent interface around rendered React components.

Enzyme 做的事情是，提供了 fluent interface。

## Example

Enzyme exports three different “modes” to render and test components, shallow, mount, and render. Shallow is the recommended mode to start with since it does a better job of isolating your tests to just a single component. If shallow doesn’t work for your use case (for example, if you are relying on the presence of a real DOM), mount or render likely will.

Enzyme 有3种“模式”来 render 和 test 组件。

- shallow 推荐，
- mount 
- render

## Github 的文档

Enzyme is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your React Components' output.

Enzyme's API is meant to be intuitive and flexible by mimicking jQuery's API for DOM manipulation and traversal.

模拟了 jQuery 操作和 traverse DOM 的 API。

对于 `React 0.14`：

```
npm i --save-dev react-addons-test-utils@1.4
npm i --save-dev react-dom@1.4
```




