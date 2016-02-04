title: React performance zh_cn
date: 2016-02-04 15:31:26
tags:
---

原文：[React performance](http://blog.vjeux.com/2013/javascript/react-performance.html)

React is a JavaScript library for building user interfaces developed by Facebook. It has been designed from the ground up with performance in mind. In this article I will present how the diff algorithm and rendering work in React so you can optimize your own apps.

这里展示下 React 用的 diff 算法和 rendering。

## Diff Algorithm

Before we go into the implementation details it is important to get an overview of how React works.

大致看下 React 的 overview。

```js
var MyComponent = React.createClass({
  render: function() {
    if (this.props.first) {
      return <div className="first"><span>A Span</span></div>;
    } else {
      return <div className="second"><p>A Paragraph</p></div>;
    }
  }
});
```

At any point in time, you describe how you want your UI to look like. It is important to understand that the result of render is not an actual DOM node. Those are just lightweight JavaScript objects. We call them the virtual DOM.

首先先记住，render 返回的不是 DOM，是轻量级的 JavaScript 对象，也就是 Virtual DOM。

React is going to use this representation to try to find the minimum number of steps to go from the previous render to the next. For example, if we mount <MyComponent first={true} />, replace it with <MyComponent first={false} />, then unmount it, here are the DOM instructions that result:

React 尝试找到最小修改内容。

```
None to first
    - Create node: <div className="first"><span>A Span</span></div>

First to second
    - Replace attribute: className="first" by className="second"
    - Replace node: <span>A Span</span> by <p>A Paragraph</p>

Second to none
    - Remove node: <div className="second"><p>A Paragraph</p></div>
```

#### Level by Level

Finding the minimal number of modifications between two arbitrary trees is a O(n4) problem. As you can imagine, this isn't tractable for our use case. React uses simple and yet powerful heuristics to find a very good approximation in O(n).

在两颗 arbitrary trees 找到最小修改数量是 O(n4)。当然，React 使用的不是这种方案。而是使用了一个简单强大的启发算法，时间复杂度接近 O(n)。

React only tries to reconcile trees level by level. This drastically reduces the complexity and isn't a big loss as it is very rare in web applications to have a component being moved to a different level in the tree. They usually only move laterally among children.

React 只会一层一层的合并 trees。这样就会大大减少复杂性，丢失的场景也不多，因为在 web 应用中，component 很少会移到其他的层级，只是在 children 间移动。

{% asset_img react_diff.png React Diff Algorithm %}

#### List

Let say that we have a component that on one iteration renders 5 components and the next inserts a new component in the middle of the list. This would be really hard with just this information to know how to do the mapping between the two lists of components.

假设这样一个场景：在一次迭代中，会渲染 5 个 components。然后再在中间插入一个新的 component。

By default, React associates the first component of the previous list with the first component of the next list, etc. You can provide a key attribute in order to help React figure out the mapping. In practice, this is usually easy to find out a unique key among the children.

默认，React 会结合前面一个 list 的第一个 component 和后面 list 的第一个 component。可以提供一个 `key` 属性，来帮助 React 解析映射。

{% asset_img react_list.png List %}

#### Components

A React app is usually composed of many user defined components that eventually turns into a tree composed mainly of divs. This additional information is being taken into account by the diff algorithm as React will match only components with the same class.

React 只会匹配相同 class 的 components。就不会浪费时间匹配肯定不同的元素。

For example if a <Header> is replaced by an <ExampleBlock>, React will remove the header and create an example block. We don't need to spend precious time trying to match two components that are unlikely to have any resemblance.

{% asset_img react_components.png Components %}

## Event Delegation

Attaching event listeners to DOM nodes is painfully slow and memory-consuming. Instead, React implements a popular technique called "event delegation". React goes even further and re-implements a W3C compliant event system. This means that Internet Explorer 8 event-handling bugs are a thing of the past and all the event names are consistent across browsers.

将 event listeners 附着到 DOM 节点是很慢的，也耗内存。React 实现了一种时髦的技术叫做 『事件委托』。React 走的更远，重新实现了一套 W3C 兼容的事件系统。也就是 IE8 的 event-handling bug 也解决了。

Let me explain how it's implemented. A single event listener is attached to the root of the document. When an event is fired, the browser gives us the target DOM node. In order to propagate the event through the DOM hierarchy, React doesn't iterate on the virtual DOM hierarchy.

解释下实现机制。一个单独的 event listener 会赋给 document 的 root。当一个 event 触发时，browser 会给出 target DOM node。为了能 propagate 这个 event 沿着 DOM 层级，React 并不会迭代 Virtual DOM 层级。

Instead we use the fact that every React component has a unique id that encodes the hierarchy. We can use simple string manipulation to get the id of all the parents. By storing the events in a hash map, we found that it performed better than attaching them to the virtual DOM. Here is an example of what happens when an event is dispatched through the virtual DOM.

而是，我们利用到 React component 有一个 id 来编码了层级。可以使用简单的 string 操作拿到所有 parents 的 ids。通过将 events 保存在一个 hash map 中，我们会发现要比附着到 Virtual DOM 中要快。

```js
// dispatchEvent('click', 'a.b.c', event)
clickCaptureListeners['a'](event);
clickCaptureListeners['a.b'](event);
clickCaptureListeners['a.b.c'](event);
clickBubbleListeners['a.b.c'](event);
clickBubbleListeners['a.b'](event);
clickBubbleListeners['a'](event);
```

The browser creates a new event object for each event and each listener. This has the nice property that you can keep a reference of the event object or even modify it. However, this means doing a high number of memory allocations. React at startup allocates a pool of those objects. Whenever an event object is needed, it is reused from that pool. This dramatically reduces garbage collection.

browser 会为每一个 event 和每个 listener 创建一个 event object。React 为了减少内存使用，会在启动分配一个 pool。

## Rendering

#### Batching

Whenever you call setState on a component, React will mark it as dirty. At the end of the event loop, React looks at all the dirty components and re-renders them.

当你调用 setState 时，React 会将该 component 标记为脏，在 event loop 的末尾，React 会找到这些 dirty component，重新渲染。

> event loop 找时间理解下

This batching means that during an event loop, there is exactly one time when the DOM is being updated. This property is key to building a performant app and yet is extremely difficult to obtain using commonly written JavaScript. In a React application, you get it by default.

batching 也就意味着，一次 event loop 只会更新一次 DOM。

{% asset_img react_batching.png Batching %}

#### Sub-tree Rendering

When setState is called, the component rebuilds the virtual DOM for its children. If you call setState on the root element, then the entire React app is re-rendered. All the components, even if they didn't change, will have their render method called. This may sound scary and inefficient but in practice, this works fine because we're not touching the actual DOM.

当调用 setState 时，会重新构建（rebuild）它 children 的 virtual DOM。如果你调用了 setState 在 root 元素，整个 React app 会 re-rendered。听起来很恐怖，但是 work fine，因为没碰到真正的 DOM。

First of all, we are talking about displaying the user interface. Because screen space is limited, you're usually displaying on the orders of hundreds to thousands of elements at a time. JavaScript has gotten fast enough business logic for the whole interface is manageable.

首先，屏幕大小有限，JavaScript 处理这种量级的用户界面还是够快的。

Another important point is that when writing React code, you usually don't call setState on the root node every time something changes. You call it on the component that received the change event or couple of components above. You very rarely go all the way to the top. This means that changes are localized to where the user interacts

另外就是，一般不会再 root 调用 setState（and Redux 就是这么做的，虽然做了很多判断）。

{% asset_img react_re_render.png Sub-tree Rendering %}

#### Selective Sub-tree Rendering

Finally, you have the possibility to prevent some sub-trees to re-render. If you implement the following method on a component:

```js
boolean shouldComponentUpdate(object nextProps, object nextState)
```

The techniques that make React fast are not new. We've known for a long time that touching the DOM is expensive, you should batch write and read operations, event delegation is faster ...

People still talk about them because in practice, they are very hard to implement in regular JavaScript code. What makes React stand out is that all those optimizations happen by default. This makes it hard to shoot yourself in the foot and make your app slow.

The performance cost model of React is also very simple to understand: every setState re-renders the whole sub-tree. If you want to squeeze out performance, call setState as low as possible and use shouldComponentUpdate to prevent re-rendering an large sub-tree.
