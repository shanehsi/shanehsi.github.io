title: redux data flow
date: 2016-04-27 18:22:48
tags:
---

原文 1：[jarvisaoieong/redux-architecture](https://github.com/jarvisaoieong/redux-architecture)

In classical Redux, which side effect is handled by thunk middleware, is not fractal (a term that is nicely explained by @stalz)

传统的 redux 中，side effect 是通过 thunk 中间件来处理的，

它不是 **[fractal](http://staltz.com/unidirectional-user-interface-architectures.html)** 的

{% asset_img redux-architecture.png %}

Even with some new Redux additions, like redux-saga, are also not composable in a fractal way with the rest of architecture.

redux-saga 也无法用 fractal 的方式和其他模块进行组合。

I think [elm architecture](https://github.com/evancz/elm-architecture-tutorial/) has found the proper way to do it right. Beside composing Views, State and Reducers (which are already composed in classical Redux), Actions and Effects should be composed too. All that leads to composition of application pieces at the higher level.

Elm 架构找到了一种正确的解决方式。除了组合 Views，States 和 reducers（redux 已经 compose），Actions 和 Effects 也会被 compose。总体来说，就能将 application pieces 组合到一个高的层级。

{% asset_img elm-architecture.png %}

**注：** 需要理解下 elm

Redux is awesome. In order to make redux architecture fractal. We only need two adjustment in our application.

1. Use [redux-loop](https://github.com/raisemarketplace/redux-loop) as side effect solution. 

  It will make the effects composable and the reducers more domain centric. It give you elm architecture side effect like solutions. 可以让 effects 变的可以 compose。reducer 更加面向 domain。给了类似 elm 架构的解决方案。

2. Don't use `bindActionCreators`, just pass `dispatch` as the parameter to the components.

  What the component needed is model (the data) and dispatch (a way to communicate with the rest architecture). It doesnt need the action callback as the parameters.不要使用 `bindActionCreators`，只要将 dispatch 传入 components。component 需要 modal（就是 data 数据）和 dispatch（和应用的其他架构部分做交互的方式）。不需要 action callback。

This repo is port of the elm architecture examples in redux with redux-loop to show the benefits of hierarchical composition everywhere. In this example, I used my fork of [redux-loop](https://github.com/jarvisaoieong/redux-loop) and [redux-logger](https://github.com/jarvisaoieong/redux-logger) to demonstrate how to log the high order action and async action. (Please open the console in the [live demo](http://jarvisaoieong.github.io/redux-architecture/).)

![](http://i.imgur.com/33MQJvu.png)

{% asset_img log.png %}

I hope we can make elm architecture to the mainstream to create a truly reusable and well encapsulated application.

