title: 单向 UI 架构
date: 2016-04-28 14:12:43
tags:
---

- **User events**
- **User interface rendering** 在屏幕上的图形输出，通常表示为HTML或一些可比高级别声明性代码如JSX。
- **UI** 它接受用户事件作为输入，并输出渲染，一个持续的过程，而不是一次性的变换。

> A unidirectional architecture is said to be **fractal** if subcomponents are structured in the same way as the whole is.

单向被认为是 fractal（分形），如果子组件的组织方式是统一的。

In fractal architectures, the whole can be naively packaged as a component to be used in some larger application.


In non-fractal architectures, the non-repeatable parts are said to be orchestrators over the parts that have hierarchical composition.

非分形架构中，非重复模块和其他模块需要特定的协调器。

## Flux

- Dispatcher 是 singleton。

- Only View has composable components.

    A React component is a UI program, and is usually not written as a Flux architecture internally. Hence Flux is not fractal, where the **orchestrators** are the **Dispatcher** and the **Stores**.

- User event handlers are declared in the rendering. event 在 rendering 里声明

    `onClick={this.clickHandler}`

## Redux

Redux is a variation of Flux where the **singleton Dispatcher** was adapted to become **a singleton Store**. 

- Singleton Store
- Provider
- Provider
- Reducers

### Peculiarities

Like Flux, Redux is not (by design) fractal and the **Store** is an **orchestrator**.

## Elm

- Model: a type defining the structure of state data 定义 data 的结构
- View: a pure function transforming state into rendering 将 state 转换成rendering
- Actions: a type defining user events sent through mailboxes 就是 type
- Update: a pure function from previous state and an action to new state 就是 reducer

和 redux 相比，他没有一个 store。

---

关于 reactive ？

{% asset_img passive.png %}

Bar is **passive**: it allows other modules to change its state. Foo is proactive: it is responsible for making Bar’s state function correctly. The passive module is unaware of the existence of the arrow which affects it.

被调用方（passive）不知道调用方（proactive）的存在。Bar 允许其他 modules，比如 Foo 来改变 Bar 本身的状态。

{% asset_img reactive.png %}

Bar is **reactive**: it is fully responsible for managing its own state by reacting to external events. Foo, on the other hand, is unaware of the existence of the arrow originating from its network request event.

Bar 完全控制自己的状态。Foo 不知道 Bar 的存在。

> What is the benefit of this approach? Inversion of Control, mainly because Bar is responsible for itself. Plus, we can hide Bar’s incrementCounter() as a private function. In the passive case, it was required to have incrementCounter() public, which means we are exposing Bar’s internal state management outwards. It also means if we want to discover how Bar’s counter works, we need to find all usages of incrementCounter() in the codebase. In this regard, Reactive and Passive seem to be duals of each other.

IoC，可以隐藏 Bar 的 incrementCounter() 作为私有方法。**如果要知道 Bar incrementCounter() 的工作机制，我们要找到所有 incrementCounter() 被调用的地方**


> 这个我觉得是重点，public/private 的本身含义很简单，关键就是 private 之后会带来的倾向性。Bar 只相应唯一一个 event 的 payload。Bar 的最终吐出的 state，外界对它的影响要最低。state，主要在 Bar，而不是把 Bar 当做一个改变 state 的操作。

---

