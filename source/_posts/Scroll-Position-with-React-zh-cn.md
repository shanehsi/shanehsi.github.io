title: Scroll Position with React 译
date: 2016-02-04 15:00:38
tags:
---

原文：[Scroll Position with React](http://blog.vjeux.com/2013/javascript/scroll-position-with-react.html)

> 通过阅读 vjeux 的博客来学习下 React 的设计理念

Dealing with scroll position when you insert content is usually a difficult problem to solve. We'll see how to use React life cycle methods to solve it elegantly.

当插入 content 时会改变 scroll position 一般来说是比较复杂的问题。我们来看下如何使用 React life cycle 方法来优雅解决。

## Insertion at the bottom 在底部插入

The first example is to maintain the scroll position at the bottom when an element is inserted at the bottom. A common use case is a chat application.

当插入新的元素到底部，继续保持在底部。比如聊天应用。

In order to scroll at the bottom, we can do that on componentDidUpdate. It happens every time the element is re-rendered.

为了滚动到底部，可以在 componentDidUpdate 来做。

```js
componentDidUpdate: function() {
  var node = this.getDOMNode();
  node.scrollTop = node.scrollHeight;
},
```

`Element.scrollTop` 指拿到 element 向上滑动的像素。关键这个操作有 side effect，scrollTop 修改之后，会触发一个动作。应该叫做 node.scrollTop(node.scrollHeight)

`Element.scrollHeight` 只读属性，是指元素内容的高度，包括看不见的（滚动的）。

But this is going to always scroll to the bottom, which can be very annoying if you want to read what was above. Instead you want to scroll only if the user was already at the bottom. To do that, we can check the scroll position before the component has updated with componentWillUpdate and scroll if necessary at componentDidUpdate

但是这样做的话，会始终滚到底部（比如当你想滚动到上面看内容，来了条新的内容，就滚到下面）。现在的需求是，只有已经在底部，才滚动。

```js
componentWillUpdate: function() {
  var node = this.getDOMNode();
  // 搞了一个实例变量在保存状态
  this.shouldScrollBottom = node.scrollTop + node.offsetHeight === node.scrollHeight;
},
 
componentDidUpdate: function() {
  if (this.shouldScrollBottom) {
    var node = this.getDOMNode();
    node.scrollTop = node.scrollHeight
  }
},
```


> 发现盒模型的各种 Height 概念挺多的
> 读一下文档

## Conclusion 结论

React has not been designed to handle scroll position natively. However, it provides escape hatches from the declarative paradigm in order to be able to implement them.

