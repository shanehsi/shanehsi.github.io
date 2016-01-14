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

## Based on react-transform-boilerplate

其余即基于 [react-transform-boilerplate](https://github.com/gaearon/react-transform-boilerplate)。

没有多余的配置，由渐入深。

## CSS

对于移动端，参考 [weui](https://github.com/weui/weui/blob/master/dist/style/weui.css)，但是只初步将其中的 weui-rest.css 和 weui-font.css（本身并没有这些文件，这里只是逻辑分块）作为全局 css。

并参考 [boy](https://github.com/corysimmons/boy) 项目。

## To be continued

