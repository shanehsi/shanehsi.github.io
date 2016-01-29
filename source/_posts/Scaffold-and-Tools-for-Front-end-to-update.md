title: Scaffold and Tools for Front-end (will update regularly)
date: 2016-01-13 14:36:49
tags:
---

技术栈：

- TypeScript
- React
- Webpack
- Babel
- PostCSS & CSS Modules
- React Router
- Redux
- IE8 兼容性
- Ajax 库
- 关于 gulp 和 webpack

## Only TypeScript's Type

其中 TypeScript 借助 IDE（如 webstorm）或者直接命令行 `tsc --watch` 编译成 `/\.jsx?$/`。也就是说，虽然用了 Webpack，但是没有使用 Webpack 的 TypeScript loader，保证出去这一步外，和使用 ES6 开发相同。`tsconfig.json` 如下：

```json
{
  "compilerOptions": {
    "target": "es2015",
    "module": "es2015",
    "jsx": "preserve"
  },
  "exclude": [
    "node_modules"
  ]
}
```

使用 TypeScript 并不会增加学习的曲线，目前仅仅是使用 TypeScript 中的 Type 系统，做 compile 期的类型检查，而不是在运行时（比如使用 PropTypes）。

我们来看几个例子：

```javascript
export function action(type:string, actionCreator?:any):(...args:any[]) => IAction {
}
```

这里用到了 `...args:any[]`，表明 `arguments` 是 array-like 的类型。并且 action 的返回值的类型是 `(...args:any[]) => IAction` 表明这是 curry。

```js
export function onActions(handlers:{[key: string]: Reducer}, initialState:Object):Reducer {
    }
```

这里的是如何对 Dictionary 做类型，`{[key: string]: Reducer}`。

## Based on react-transform-boilerplate

其余即基于 [react-transform-boilerplate](https://github.com/gaearon/react-transform-boilerplate)。

没有多余的配置，由浅入深。

## CSS

对于移动端，参考 [weui](https://github.com/weui/weui/blob/master/dist/style/weui.css)，但是只初步将其中的 weui-rest.css 和 weui-font.css（本身并没有这些文件，这里只是逻辑分块）作为全局 css。

并参考 [boy](https://github.com/corysimmons/boy) 项目。

## React Router

首先，并没有说使用 Redux 来管理 router history。

另外，我是想自己写一个 router 的。

## Redux

Redux 之前看过一遍源码，整体的实现原理，包括响应式的实现，而且 reducer 的 hot reload 等。前几天开始重新整理下源码的思路。

## IE8 兼容性

React 本身支持 IE9+，使用 es5-shim 可以支持到 IE8+。另外还有一点是，default 保留字是不允许 `exports.default`，所以要使用 es3ify 转换成 `exports['default']`。本身是不增加代码量就可以兼容。

兼容 polyfill:

```
<!--[if lt IE 9]>
<script src="/assets/js/vendor/es5-shim.min.js"></script>
<script src="/assets/js/vendor/es5-sham.min.js"></script>
<![endif]-->
```

另外，也要考虑使用的依赖和编写的组件是否兼容。而且这些只有在写的过程中不断地在 IE8 测试，总之不是轻松的活计。

## Ajax

Ajax 库感兴趣完全可以自己写一个，相对比较固定的写法。网上类似的参考也很多。

- [ForbesLindesay/ajax](https://github.com/ForbesLindesay/ajax)
- [ma3uk/ajax](https://github.com/ma3uk/ajax)
- [mythz/xhr.js](https://gist.github.com/mythz/1334560)
- [dexteryy/mo](https://github.com/dexteryy/mo)

目前使用 [superagent](https://github.com/visionmedia/superagent)，是因为发现它的 response 的类型很丰富。

## 关于 gulp 和 webpack

webpack 出现后，曾经有一段时间，前端社区开始去 gulp，而开始使用 npm scripts。并且 webpack 的 loader 功能上和 gulp 有重合。

gulp，将所有的文件类型，比如：css，通过 src 读取文件，然后 pipe 一系列的 plguin，最终写到 dest。

webpack，js 文件可以 require 各种类型，font，img，css 等，然后不同的 loader 进行处理，如果可以写在 html（其实是 jsx）里，结果就是相应的写在 js 的方法（比如 inline img），要么就是 extract 出来。

webpack 的特点就是：最好都 inline 到 js 里。

gulp 的特点是：由于没有 js 这种变态的集大成者（html，inline style，inline img 等），所以最终需要在 html 里做集成（`<style />`, `<link style />` `<script` 等）。

所以，结论就好下了，预测 webpack 的处理方式是符合趋势的（比如 shadow dom）。目前 webpack 和 react 结合的紧密。所以，需要一种标准，而不是很重的实现（接口分离开发）。

jsx 作为集成者：

- 本身作为结构（DOM）
- 绑定样式：inline style，后者 css modules（class 作为 key）
- 绑定事件系统
- 绑定图片，在样式里
- 绑定字体，在样式里



## To be continued
