title: Wallaby node
date: 2016-04-16 12:03:55
tags:
---

[demo](https://github.com/shanehsi/wallaby-node-iojs-sample)

先看 package.json

```
{
  "private": true,
  "devDependencies": {
    "chai": "^3.5.0"
  }
}
```

用了 chai 这个 assertion library。

在看 wallaby.js

```js
module.exports = function () {
  return {
    files: [
      'lib/**/*.js'
    ],

    tests: [
      'test/**/*Spec.js'
    ],

    bootstrap: function () {
      global.expect = require('chai').expect;
    },

    env: {
      type: 'node'
      // More options are described here
      // http://wallabyjs.com/docs/integration/node.html
    }
  };
};
```

解释下： bootstrap，顾名思义，就是在 bootstrap 是，重新将 expect 全局变量替换下。

```js
bootstrap: function () {
  global.expect = require('chai').expect;
},
```

更多的[配置](http://wallabyjs.com/docs/integration/node.html)

