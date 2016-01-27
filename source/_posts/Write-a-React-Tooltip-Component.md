title: Write a React Tooltip Component
date: 2016-01-22 10:25:45
tags:
---

# 前言

今天来写下 tooltip 组件。

如果今天有时间，写下 table 组件，同样，以上两个样式都是以最简单为主。

这些写完之后，加上 button，icon，input 这些非常依赖样式的组件，这些等下周专心写 css 时再说。周末做些组件的类型层级分析归类重构。

## 场景

- Form 的 label 后面可能会带个 ？的图标，鼠标悬浮或者点击会有 popup 浮层。
- Form 校验的时候，发生错误（或者极少数情况人性化的警告下）会在 input（类似的有 select，calendar ）的框靠右内部显示一个 ！的红色 icon，然后默认会自动弹出 popup 浮层，显示错误的提示信息。
- 比如上传按钮，可以在右侧也有一个 popup 浮层。显示上传成功。类似有 提交 按钮。

所以，这个 trigger 相应的事件是

- click
- focus
- hover
- 还有就是自动触发显示。

> 再强调下，当前不要把时间花在抽出公共组件上，先把功能实现。

## 实现方式

代码因为和 Select 的场景类似（都是 trigger 一下，显示其他的组件）。

关键是定位问题。

我发现有两种定位方式，有用 `position:fixed` 的，很明显的问题是，滚动时就会相对 viewport 固定。也有相对 body 绝对定位的，这个基本没有问题。但是 tooltip 是在单独的地方维护的，当然写的时候可以一块，也就是并不真正render在被声明的地方。后面的方式是合理的。不过总感觉有些变扭。

找下其他方案吧。

参考[这篇资料](http://stackoverflow.com/questions/21064101/understanding-offsetwidth-clientwidth-scrollwidth-and-height-respectively)先学习下 `offsetWidth`，`clientWidth`，`scrollWidth` 这些概念。

另外翻到一篇文章，由于 [层叠上下文 Stacking Context](http://web.jobbole.com/83409/)（[另一篇](http://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)），Modal 这种东西应该直接 append 到 body。就和上面第二种方式类似。概念看 spec 会看很久，从浏览器绘制顺序来看，就是浏览器先绘制父元素，在绘制 z-index 为复数，然后是 `position:static` ，然后是正数（从小到大）。

还有一个接口 [getBoundingClientRect](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)，返回元素的大小及其相对于视口的位置。

## To be continued

目前在 Tooltip 的研究上出现停滞，主要是在写的过程中发现了之间漏掉的设计点，这块是看到了这个文章 [React中的Portal组件](https://leozdgao.me/reactzhong-de-portalzu-jian/)之后对 Portal 这个抽象概念，期望有更多的理解。当然也是为了解决层叠上下文。只是，Portal 作为公共组件，沉淀的设计会更多，更值得学习。

另外，目前暂时参考了 react-components，自定义了样式，和再做了一层业务自定义的封装，以保证项目进度。同时，对属性做了接口限制（TypeScript 的优势）。希望对抽象做分层，循序渐进，不断迭代优化。

