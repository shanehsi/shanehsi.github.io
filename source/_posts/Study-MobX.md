title: Study MobX
date: 2016-05-26 15:15:32
tags:
---

MobX 是一个新的框架，主打 mutable。哈哈。我是来调研 API 设计的。具体实现暂时不看。

> 注：Redux 提供的功能是在是太简陋了。

Everything that can be derived from the application state, should be derived. Automatically.

就是不要通过 props 传递，自动的

{% asset_img flow.png %}

React renders the application state by providing mechanisms to translate it into a tree of renderable components.

MobX provides the mechanism to store and update the application state that React then uses.

上面一句话详细说是：

MobX provides mechanisms to optimally synchronize application state with your React components by using a reactive virtual dependency state graph that is only updated when strictly needed and is never stale.

使用一个 reactive 的虚拟 state graph 的机制来优化地同步应用状态和 React 组件。 只在严格需要的时候更新，并且不会 stale。

## Core concepts

### Observable state

MobX adds observable capabilities to existing data structures like objects, arrays and class instances.

MobX 给存在的 data structures（比如 objects，arrays，和 class instances） 增加了 observable 的功能，

**API：**

This can simply be done by annotating your class properties with the [@observable](http://mobxjs.github.io/mobx/refguide/observable-decorator.html) decorator (ES.Next), or by invoking the [`observable`](http://mobxjs.github.io/mobx/refguide/observable.html) or [`extendObservable`](http://mobxjs.github.io/mobx/refguide/extend-observable.html) functions (ES5). See [Language support](https://github.com/mobxjs/mobx/wiki/Language-Support) for language-specific examples.

```js
class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}
```

Using `@observable` is like turning a value into a spreadsheet cell. But unlike spreadsheets, these values can be not just primitive values, but references, objects and arrays as well. You can even [define your own](http://mobxjs.github.io/mobx/refguide/extending.html) observable data sources.

speadsheets 就是 reactive 编程了。

我想到了一点，这个的确挺不清晰的啊。都是有东西来通知。自动通知。

### Computed values

With MobX you can define values that will be derived automatically when relevant data is modified. By using the [`@computed`](http://mobxjs.github.io/mobx/refguide/computed-decorator.html) decorator or by using parameterless functions as property values in `extendObservable`.

```js
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}
```

### Reactions

Reactions are similar to a computed value, but instead of producing a new value, a reaction produces a side effect for things like printing to the console, making network requests, incrementally updating the React component tree to patch the DOM, etc.
In short, reactions bridge [reactive](https://en.wikipedia.org/wiki/Reactive_programming) and [imperative](https://en.wikipedia.org/wiki/Imperative_programming) programming.

和 computed value 类似，但是它不是为了生成一个新的值，而是产生一个 side effect，比如：打印到 console，请求网络，逐步更新 React Component tree 来 patch DOM。

简而言之，reactions 是来桥接 reactive 和 imperative。

If you are using React, you can turn your (stateless function) components into reactive components by simply adding the [`@observer`](http://mobxjs.github.io/mobx/refguide/observer-component.html) decorator from the `mobx-react` package onto them.

`observer` turns React (function) components into derivations of the data they render.

`observer` 将 React 组件变成他们需要 render 的 data 的起源。

When using MobX there are no smart or dumb components.

All components render smartly but are defined in a dumb manner.

所有的组件都是 smartly 的 render 的，但是写法是 dumb 的。（因为都是通过 observable）。

MobX will simply make sure the components are always re-rendered whenever needed, but also no more than that. 


