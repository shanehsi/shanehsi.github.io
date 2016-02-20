title: 使用 Router5
date: 2016-01-15 17:12:45
tags:
---

Router5 的[文档](http://router5.github.io/docs/understanding-router5.html) 对 router 的解释非常直观详尽，是深入理解 SPA 中 router 的很好资料。其对 react 的绑定，[react-router5](http://router5.github.io/docs/with-react.html#/inbox) 的实现也很简单直观。理解越多，使用越灵活，功能就越强大。

## Get started

```
npm install router5

```

其接口：

```js
var router5 = require('router5');

// router5.Router5
// router5.RouteNode
```

提供的插件：

```
npm install router5-listeners
npm install router5-history
```

Code example:


```js
import { Router5 } from 'router5';
import historyPlugin from 'router5-history';
import listenersPlugin from 'router5-listeners';

const router = new Router()
    .addNode('home', '/home')
    .usePlugin(historyPlugin())
    .usePlugin(listenersPlugin())
    // Development helper
    .usePlugin(Router5.loggerPlugin())
    .start();
```

```js
router.canDeactivate('home', true);
// is the equivalent of, 为 0.x 的写法，canDeactivate 是 shortcut version
router.registerComponent('home', {
    canDeactivate: function () {
        return true;
    }
});
```

## Configuring routes

```js
const router = new Router5()
    .setOption('useHash', true)
    // .setOption('hashPrefix', '!')
    .setOption('defaultRoute', 'inbox')
    // Routes
    .addNode('inbox',         '/inbox')
    .addNode('inbox.message', '/message/:id')
    .addNode('compose',       '/compose')
    .addNode('contacts',      '/contacts')
```

## Options

```js
var router = new Router5([], {
        useHash: true,
        hashPrefix: '!',
        defaultRoute: 'home',
        defaultParams: {},
        base: '',
        trailingSlash: false,
        autoCleanUp: true,
        strictQueryParams: true
    })
    .setOption('useHash', false)
    .setOption('hashPrefix', '');
```

useHash，hashPrefix 会被 `router5-history` 用到。

defaultRoute，defaultParams 会被用作默认 navigation。

## Preventing navigation

场景，允许/阻止用户进入下一个 view：比如用户在填写表单，未保存之前，给出提示，并禁止离开当前 view。

## 整体感受

最终还是选择了先使用 `react-router`。

router5 整体的文档非常清楚，设计感和覆盖场景也很现实常用。只可惜 react-router5 的很多很变扭，而且还需要很多的 biolerplate 代码，最关键的是 NotFound 的设计很原始。

Anyway，有机会自己实现一个。对 router 也是一种熟悉。
