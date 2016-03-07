
title: 阅读 React 代码结构
date: 2016-02-02 18:00:25
tags:
---

## 声明

- 解读的是结构

## 计划

- 读下 react.d.ts，目的是会写复杂的 definition 文件。
- 会画结构图，这个优先级低了，初期用纸笔

## 开始吧

#### npm dependencies

[esprima](https://www.npmjs.com/package/esprima)

这个似乎有更快的。在 webpack 还是 rollup 的依赖里有。

[commoner](https://www.npmjs.com/package/commoner)

Flexible tool for translating any dialect of JavaScript into Node-readable CommonJS modules。这个是 React 团队维护的包。

[recast](https://www.npmjs.com/package/recast)

JavaScript syntax tree transformer, nondestructive pretty-printer, and automatic source map generator

commoner，recast 的主要作者都是 [@benjamn](https://github.com/benjamn)。曾经在 Facebook 工作。[TC39](https://github.com/tc39) 成员。

#### react.d.ts

##### declare 关键字

有时我们需要对相关定义进行补充, 但又不希望这些内容生成相关 JS 代码 (也许是本身某些对象或者值就已经存在在运行环境里, 另在用于桥接 TypeScript 和原有 JavaScript 的声明文件 *.d.ts 中就会大量使用 declare).

关键词是：补充，不会生成 JS 代码

另外 TypeScript 专门为一些 JS 类库扩展了 declare module, 以适应一些包含特殊字符的包/模块名称.

```js
declare module 'some-package' {
    //...
}
```

##### namespaces

Class 和 interface 做了些 design time 的限制（private，protected），但是没有做 runtime。

在 JavaScript 中，runtime 的 封装是用了 module pattern。即用闭包封装 private 的 fields 和 methods。


module pattern 是很清晰的方式，用来组织化结构（ organizational structure ），以及动态加载选项（dynamic loading options）。

```js
namespace M {  
    var s = "hello";  
    export function f() {  
        return s;  
    }  
}

M.f();  
M.s;  // Error, s is not exported

// 编译出来的是
var M;
(function (M) {
    var s = "hello";
    function f() {
        return s;
    }
    M.f = f;
})(M || (M = {}));
M.f();
M.s; // Error, s is not exported

```

##### 看代码

```js
interface ReactElement<P extends Props<any>> {
        type: string | ComponentClass<P> | StatelessComponent<P>;
        props: P;
        key: string | number;
        ref: string | ((component: Component<P, any> | Element) => any);
    }
```

其中看下 `Component<P, any>`：

```js
// Base component for plain JS classes
class Component<P, S> implements ComponentLifecycle<P, S> {
    constructor(props?: P, context?: any);
    setState(f: (prevState: S, props: P) => S, callback?: () => any): void;
    setState(state: S, callback?: () => any): void;
    forceUpdate(callBack?: () => any): void;
    render(): JSX.Element;
    props: P;
    state: S;
    context: {};
    refs: {
        [key: string]: ReactInstance
    };
}
```

这里面还有重载：

```js
setState(f: (prevState: S, props: P) => S, callback?: () => any): void;
setState(state: S, callback?: () => any): void;
```

任意参数：

```js
interface Factory<P> {
    (props?: P, ...children: ReactNode[]): ReactElement<P>;
}
```

主要就是： `...children: ReactNode[]`。

这些也是重载：

```js
function createElement<P>(
        type: string,
        props?: P,
        ...children: ReactNode[]): DOMElement<P>;
function createElement<P>(
    type: ClassicComponentClass<P>,
    props?: P,
    ...children: ReactNode[]): ClassicElement<P>;
function createElement<P>(
    type: ComponentClass<P> | StatelessComponent<P>,
    props?: P,
    ...children: ReactNode[]): ReactElement<P>;
```

看下整体的 module 写法：

```js
declare namespace __React {
    // ...
}

declare module "react" {
    export = __React;
}
```

另外，如何定义 function interface？

```js
declare type ClassValue = string | number | ClassDictionary | ClassArray;

interface ClassDictionary {
    [id: string]: boolean;
}

interface ClassArray extends Array<ClassValue> { }

interface ClassNamesFn {
    (...classes: ClassValue[]): string;
}

declare var classNames: ClassNamesFn;

declare module "classnames" {
    export = classNames
}
```

重点是：

```js
interface ClassNamesFn {
    (...classes: ClassValue[]): string;
}
```

#### 第一次公开发布的 React 源码

因为这时候，代码量相对是最精简的。

##### core/React.js

```js
"use strict";

var ReactCompositeComponent = require('ReactCompositeComponent');
var ReactComponent = require('ReactComponent');
var ReactDOM = require('ReactDOM');
var ReactMount = require('ReactMount');

// 这个看起来是比较独立的代码
var ReactDefaultInjection = require('ReactDefaultInjection');

ReactDefaultInjection.inject();

var React = {
  DOM: ReactDOM,
  initializeTouchEvents: function(shouldUseTouch) {
    ReactMount.useTouchEvents = shouldUseTouch;
  },
  autoBind: ReactCompositeComponent.autoBind,
  createClass: ReactCompositeComponent.createClass,
  createComponentRenderer: ReactMount.createComponentRenderer,
  constructAndRenderComponent: ReactMount.constructAndRenderComponent,
  constructAndRenderComponentByID: ReactMount.constructAndRenderComponentByID,
  renderComponent: ReactMount.renderComponent,
  unmountAndReleaseReactRootNode: ReactMount.unmountAndReleaseReactRootNode,
  isValidComponent: ReactComponent.isValidComponent
};

module.exports = React;
```

##### core/ReactDefaultInjection.js

```js
"use strict";

var ReactDOM = require('ReactDOM');
var ReactDOMForm = require('ReactDOMForm');

var DefaultEventPluginOrder = require('DefaultEventPluginOrder');
var EnterLeaveEventPlugin = require('EnterLeaveEventPlugin');
var EventPluginHub = require('EventPluginHub');
var ReactInstanceHandles = require('ReactInstanceHandles');
var SimpleEventPlugin = require('SimpleEventPlugin');

function inject() {
  /**
   * Inject module for resolving DOM hierarchy and plugin ordering.
   * 注入用来解析 DOM 层级和 plugin 顺序的模块
   */
  EventPluginHub.injection.injectEventPluginOrder(DefaultEventPluginOrder);
  EventPluginHub.injection.injectInstanceHandle(ReactInstanceHandles);

  /**
   * Two important event plugins included by default (without having to require
   * them).
   * 默认注入的两个重要的 plugins
   */
  EventPluginHub.injection.injectEventPluginsByName({
    'SimpleEventPlugin': SimpleEventPlugin,
    'EnterLeaveEventPlugin': EnterLeaveEventPlugin
  });

  /*
   * This is a bit of a hack. We need to override the <form> element
   * to be a composite component because IE8 does not bubble or capture
   * submit to the top level. In order to make this work with our
   * dependency graph we need to inject it here.
   * 这是hack，因为 IE8 的 form 标签不会 bubble 或者 capture submit 操作到 top level。
   * 为了让 dependency grapth 能够工作，这里覆盖下 <form>
   */
  ReactDOM.injection.injectComponentClasses({
    form: ReactDOMForm
  });
}

// 暴露 inject
module.exports = {
  inject: inject
};

```

这里面基本做了很多 module 之间的组合。

```
EventPluginHub - inject - DefaultEventPluginOrder
                        - ReactInstanceHandles
                        - SimpleEventPlugin
                        - EnterLeaveEventPlugin
                        - injectComponentClasses - ReactDOMForm
```

##### core/ReactDOM.js

```js

"use strict";

var ReactNativeComponent = require('ReactNativeComponent');

// 使用第二个参数的同key值修改第一个的
var mergeInto = require('mergeInto');
var objMapKeyVal = require('objMapKeyVal');

/**
 * Creates a new React class that is idempotent and capable of containing other
 * React components. It accepts event listeners and DOM properties that are
 * valid according to `DOMProperty`.
 *
 *  - Event listeners: `onClick`, `onMouseDown`, etc.
 *  - DOM properties: `className`, `name`, `title`, etc.
 *
 * The `style` property functions differently from the DOM API. It accepts an
 * object mapping of style properties to values.
 *
 * 创建一个新的 React class。它是幂等的，可以包含其他的 React Component。可以接受 event listeners 和 DOM properties（右 DOMProperty 校验）
 * 已经开了一篇新的博客，学习下 this constructor prototype, __proto__，暂时不求甚解。
 * 
 * @param {string} tag Tag name (e.g. `div`).
 * @param {boolean} omitClose True if the close tag should be omitted.
 * @private
 */
function createDOMComponentClass(tag, omitClose) {
  var Constructor = function(initialProps, children) {
    this.construct(initialProps, children);
  };

  Constructor.prototype = new ReactNativeComponent(tag, omitClose);
  Constructor.prototype.constructor = Constructor;

  return function(props, children) {
    return new Constructor(props, children);
  };
}

/**
 * Creates a mapping from supported HTML tags to `ReactNativeComponent` classes.
 * This is also accessible via `React.DOM`.
 *
 * @public
 */
var ReactDOM = objMapKeyVal({
  // 略去一些 tag，紧凑下代码
  a: false,
  footer: false,
  // Danger: this gets monkeypatched! See ReactDOMForm for more info.
  form: false,
  h1: false,
  h6: false,
  // SVG
  circle: false,
  g: false,
  path: false,
}, createDOMComponentClass);

var injection = {
  injectComponentClasses: function(componentClasses) {
    mergeInto(ReactDOM, componentClasses);
  }
};

ReactDOM.injection = injection;

module.exports = ReactDOM;
```

关于 `objMap` 和 `objMapKeyVal.js`：

```

objMap.js
For each key/value pair, invokes callback func and constructs a resulting
 * object which contains, for every key in obj, values that are the result of
 * of invoking the function:

 *   func(value, key, iteration)


objMapKeyVal.js
Behaves the same as `objMap` but invokes func with the key first, and value
 * second. Use `objMap` unless you need this special case.
 * Invokes func as:
 *
 *   func(key, value, iteration)
```

{% asset_img objMap.png objMap vs objMapKeyVal %}

以 `objMapKeyVal(obj, func, context)` 为例，就是，对一个 `obj`，调用 `func` 重新求各 key 的 value。


在关于：

```js
var injection = {
  injectComponentClasses: function(componentClasses) {
    // 使用后者替换前者
    mergeInto(ReactDOM, componentClasses);
  }
};
```

在 `core/ReactDefaultInjection.js` 使用到：

```js
  ReactDOM.injection.injectComponentClasses({
    form: ReactDOMForm
  });
```

##### core/ReactMount.js

```js
"use strict";

var ReactEvent = require('ReactEvent');
var ReactInstanceHandles = require('ReactInstanceHandles');
var ReactEventTopLevelCallback = require('ReactEventTopLevelCallback');

var $ = require('$');

var globalMountPointCounter = 0;

/** Mapping from reactRoot DOM ID to React component instance. 
 * reactRoot DOM ID : React component instace
 */
var instanceByReactRootID = {};

/** Mapping from reactRoot DOM ID to `container` nodes. 
 *  reactRoot DOM ID : `container` node
 */
var containersByReactRootID = {};

/**
 * @param {DOMElement} container DOM element that may contain a React component.
 * @return {?string} A "reactRoot" ID, if a React component is rendered.
 * 可能包含一个 React component 的 container DOM element
 * 可能返回 reactRoot ID，如果这个 React component 已经渲染了
 */
function getReactRootID(container) {
  return container.firstChild && container.firstChild.id;
}

/**
 * Mounting is the process of initializing a React component by creatings its
 * representative DOM elements and inserting them into a supplied `container`.
 * Any prior content inside `container` is destroyed in the process.
 * Mounting 是初始化 React component 的过程，通过创建它的代理 DOM 元素，然后插入到提供的 container。
 * 并且，之前如果在 container 里有内容，会被 destroyed。
 *
 *   ReactMount.renderComponent(component, $('container'));
 *
 *   <div id="container">         <-- Supplied `container`.
 *     <div id=".reactRoot[3]">   <-- Rendered reactRoot of React component.
 *       // ...
 *     </div>
 *   </div>
 *
 * Inside of `container`, the first element rendered is the "reactRoot".
 */
var ReactMount = {

  /** Time spent generating markup. 创建 markup 的时间 */
  totalInstantiationTime: 0,

  /** Time spent inserting markup into the DOM. 将 markup 插入到 DOM 的时间 */
  totalInjectionTime: 0,

  /** Whether support for touch events should be initialized. 是否要支持 touch events */
  // 可以在外面修改 
  // initializeTouchEvents: function(shouldUseTouch) {
  //    ReactMount.useTouchEvents = shouldUseTouch;
  // },
  useTouchEvents: false,

  /**
   * This is a hook provided to support rendering React components while
   * ensuring that the apparent scroll position of its `container` does not
   * change.
   * 提供的一个 hook，来支持渲染 React components 时保证 container 的 scroll positon 不变。
   * vjeux 有一篇博客讲解 [Scroll Position with React](http://blog.vjeux.com/2013/javascript/scroll-position-with-react.html)
   *
   * @param {DOMElement} container The `container` being rendered into.
   * @param {function} renderCallback This must be called once to do the render.
   */
  scrollMonitor: function(container, renderCallback) {
    // 怎么没用到 container
    renderCallback();
  },

  /**
   * Ensures tht the top-level event delegation listener is set up. This will be
   * invoked some time before the first time any React component is rendered.
   * 保证顶级的 event delegation listener 被设置。这个会被调用几次，知道有一个 React component 被渲染。
   * @param {object} TopLevelCallbackCreator
   * @private
   */
  prepareTopLevelEvents: function(TopLevelCallbackCreator) {
    ReactEvent.ensureListening(
      ReactMount.useTouchEvents,
      TopLevelCallbackCreator
    );
  },

  /**
   * Renders a React component into the DOM in the supplied `container`.
   *
   * If the React component was previously rendered into `container`, this will
   * perform an update on it and only mutate the DOM as necessary to reflect the
   * latest React component.
   *
   * @param {ReactComponent} nextComponent Component instance to render.
   * @param {DOMElement} container DOM element to render into.
   * @return {ReactComponent} Component instance rendered in `container`.
   * 这个就是 mouting 的过程了。
   * 详细说来，如果一个 React component 之前已经渲染过了，会执行 update，并只更改必要的 DOM 变更
   * 会暴露 React.renderComponent
   */
  renderComponent: function(nextComponent, container) {
    // 首先尝试是否在 container 里有没有 React component instance
    var prevComponent = instanceByReactRootID[getReactRootID(container)];
    // 如果有
    if (prevComponent) {
      // 拿到 props
      var nextProps = nextComponent.props;
      // 
      ReactMount.scrollMonitor(container, function() {
        // 所以要做的更新是在 React component 上的方法，replaceProps
        prevComponent.replaceProps(nextProps);
      });
      return prevComponent;
    }
    // 如果没有，就...

    // 保证下顶级的 event delegation listener 被设置
    ReactMount.prepareTopLevelEvents(ReactEventTopLevelCallback);

    // 保存下 React component instace 到 map 里
    var reactRootID = ReactMount.registerContainer(container);
    instanceByReactRootID[reactRootID] = nextComponent;
    // 将 component 加载到 Node 中
    nextComponent.mountComponentIntoNode(reactRootID, container);
    return nextComponent;
  },

  /**
   * Creates a function that accepts a `container` and renders the supplied
   * React component instance into it.
   *
   *   var renderInto = ReactMount.createComponentRenderer(component);
   *   // ...
   *   var component = renderInto($('container'));
   *
   * @param {ReactComponent} component Component instance to render.
   * @return {function(DOMElement): ReactComponent}
   * 创建的是一个 Renderer，还差一个 container
   * 返回一个函数，参数是 `container`，并将提供的 React component 实例渲染进去
   * 这个会暴露在 React.createComponentRenderer
   */
  createComponentRenderer: function(component) {
    return function(container) {
      return ReactMount.renderComponent(component, container);
    };
  },

  /**
   * Constructs a component instance of `constructor` with `initialProps` and
   * renders it into the supplied `container`.
   *
   * @param {function} constructor React component constructor.
   * @param {?object} props Initial props of the component instance.
   * @param {DOMElement} container DOM element to render into.
   * @return {ReactComponent} Component instance rendered in `container`.
   * 提供的是 React component constructor
   * 会暴露 React.constructAndRenderComponent
   */
  constructAndRenderComponent: function(constructor, props, container) {
    return ReactMount.renderComponent(constructor(props), container);
  },

  /**
   * Constructs a component instance of `constructor` with `initialProps` and
   * renders it into a container node identified by supplied `id`.
   *
   * @param {function} componentConstructor React component constructor
   * @param {?object} props Initial props of the component instance.
   * @param {string} id ID of the DOM element to render into.
   * @return {ReactComponent} Component instance rendered in the container node.
   * container 是用提供的 id 获取的
   * 会暴露 React.constructAndRenderComponentByID
   */
  constructAndRenderComponentByID: function(constructor, props, id) {
    return ReactMount.constructAndRenderComponent(constructor, props, $(id));
  },

  /**
   * Registers a container node into which React components will be rendered.
   * This also creates the "reatRoot" ID that will be assigned to the element
   * rendered within.
   *
   * @param {DOMElement} container DOM element to register as a container.
   * @return {string} The "reactRoot" ID of elements rendered within.
   * 注册一个 container node 到 container 里。
   * 会创建一个 reactRoot Id，并赋给被渲染的 element
   */
  registerContainer: function(container) {
    // container.firstChild.id
    var reactRootID = getReactRootID(container);
    if (reactRootID) {
      // If one exists, make sure it is a valid "reactRoot" ID.
      reactRootID = ReactInstanceHandles.getReactRootIDFromNodeID(reactRootID);
    }
    if (!reactRootID) {
      // No valid "reactRoot" ID found, create one.
      reactRootID = ReactInstanceHandles.getReactRootID(
        globalMountPointCounter++
      );
    }
    containersByReactRootID[reactRootID] = container;
    return reactRootID;
  },

  /**
   * Unmounts and destroys the React component rendered in the `container`.
   *
   * @param {DOMElement} container DOM element containing a React component.
   * 卸载和销毁，就通过 reactRootId（container first child Id）拿到，然后调用 component.unmountComponentFromNode
   * 会暴露 React.unmountAndReleaseReactRootNode
   */
  unmountAndReleaseReactRootNode: function(container) {
    var reactRootID = getReactRootID(container);
    var component = instanceByReactRootID[reactRootID];
    // TODO: Consider throwing if no `component` was found.
    component.unmountComponentFromNode(container);
    // 删除 container 里的 component
    delete instanceByReactRootID[reactRootID];
    // 删除 container，container 的 id 也是用 first child id 表示的
    delete containersByReactRootID[reactRootID];
  },

  /**
   * Finds the container DOM element that contains React component to which the
   * supplied DOM `id` belongs.
   *
   * @param {string} id The ID of an element rendered by a React component.
   * @return {?DOMElement} DOM element that contains the `id`.
   */
  findReactContainerForID: function(id) {
    var reatRootID = ReactInstanceHandles.getReactRootIDFromNodeID(id);
    // TODO: Consider throwing if `id` is not a valid React element ID.
    return containersByReactRootID[reatRootID];
  },

  /**
   * Given the ID of a DOM node rendered by a React component, finds the root
   * DOM node of the React component.
   *
   * @param {string} id ID of a DOM node in the React component.
   * @return {?DOMElement} Root DOM node of the React component.
   */
  findReactRenderedDOMNodeSlow: function(id) {
    // 先找到 container
    var reactRoot = ReactMount.findReactContainerForID(id);
    return ReactInstanceHandles.findComponentRoot(reactRoot, id);
  }

};

module.exports = ReactMount;
```

这里先看下 `$.js` 和 `ge.js`。

```js

/**
 * Find a node by ID. 通过 ID 获取 node
 *
 * If your application code depends on the existence of the element, use $,
 * which will throw if the element doesn't exist.
 * 如果你的应用程序依赖 element 的存在性，使用 $，它会 throw 如果不存在。
 *
 * If you're not sure whether or not the element exists, use ge instead, and
 * manually check for the element's existence in your application code.
 * 如果不要 throw，使用 ge，然后手动检查
 */
function $(arg) {
  var element = ge(arg);
  if (!element) {
    if (typeof arg == 'undefined') {
      arg = 'undefined';
    } else if (arg === null) {
      arg = 'null';
    }
    throw new Error(
      'Tried to get element "' + arg.toString() + '" but it is not present ' +
      'on the page.'
    );
  }
  return element;
}
```

看下 `ge.js`：

重点是这句：

```js
document.getElementById(arg)
```

Mounting 的意思：

```
 *   ReactMount.renderComponent(component, $('container'));
 *
 *   <div id="container">         <-- Supplied `container`.
 *     <div id=".reactRoot[3]">   <-- Rendered reactRoot of React component.
 *       // ...
 *     </div>
 *   </div>
```

关于 scroll position，vjeux 有一篇博客讲解， [Scroll Position with React](http://blog.vjeux.com/2013/javascript/scroll-position-with-react.html)。

##### core/ReactCompositeComponent.js

A composite component is a component comprised of other components. Composite Component 就是用来包含其他 components 的 component。

invariant.js

```
/**
 * Use invariant() to assert state which your program assumes to be true.
 *
 * Provide sprintf style format and arguments to provide information about
 * what broke and what you were expecting.
 *
 * The invariant message will be stripped in production, but the invariant
 * will remain to ensure logic does not differ in production.
 */
```

merge.js，浅复制。

mixInto.js，Simply copies properties to the prototype.

> 对于原型还是不理解，这块必须看懂原型才能看懂啊。

```js

"use strict";

var ReactComponent = require('ReactComponent');
var ReactCurrentOwner = require('ReactCurrentOwner');
var ReactOwner = require('ReactOwner');
var ReactPropTransferer = require('ReactPropTransferer');

var invariant = require('invariant');
var keyMirror = require('keyMirror');
var merge = require('merge');
var mixInto = require('mixInto');

/**
 * Policies that describe methods in `ReactCompositeComponentInterface`.
 * 某些方法的策略
 */
var SpecPolicy = keyMirror({
  /**
   * These methods may be defined only once by the class specification or mixin.
   */
  DEFINE_ONCE: null,
  /**
   * These methods may be defined by both the class specification and mixins.
   * Subsequent definitions will be chained. These methods must return void.
   */
  DEFINE_MANY: null,
  /**
   * These methods are overriding the base ReactCompositeComponent class.
   */
  OVERRIDE_BASE: null
});

/**
 * Composite components are higher-level components that compose other composite
 * or native components.
 * 是高阶组件，组合其他 composite 或者 DOM 原生的组件
 *
 * To create a new type of `ReactCompositeComponent`, pass a specification of
 * your new class to `React.createClass`. The only requirement of your class
 * specification is that you implement a `render` method.
 * 使用 createClass 创建，唯一要实现的是 render 方法
 *
 *   var MyComponent = React.createClass({
 *     render: function() {
 *       return <div>Hello World</div>;
 *     }
 *   });
 *
 * The class specification supports a specific protocol of methods that have
 * special meaning (e.g. `render`). See `ReactCompositeComponentInterface` for
 * more the comprehensive protocol. Any other properties and methods in the
 * class specification will available on the prototype.
 * class 规范支持一些特定的协议。参看 ReactCompositeComponentInterface 来了解详情
 *
 * @interface ReactCompositeComponentInterface
 * @internal
 */
var ReactCompositeComponentInterface = {

  /**
   * An array of Mixin objects to include when defining your component.
   *
   * @type {array}
   * @optional
   */
  mixins: SpecPolicy.DEFINE_MANY,

  /**
   * Definition of props for this component.
   *
   * @type {array}
   * @optional
   */
  props: SpecPolicy.DEFINE_ONCE,



  // ==== Definition methods ====

  /**
   * Invoked once before the component is mounted. The return value will be used
   * as the initial value of `this.state`.
   * 会在 component 第一次挂载时调用一次，返回值就是 this.state 的初始值
   *
   *   getInitialState: function() {
   *     return {
   *       isOn: false,
   *       fooBaz: new BazFoo()
   *     }
   *   }
   *
   * @return {object}
   * @optional
   */
  getInitialState: SpecPolicy.DEFINE_ONCE,

  /**
   * Uses props from `this.props` and state from `this.state` to render the
   * structure of the component.
   *
   * No guarantees are made about when or how often this method is invoked, so
   * it must not have side effects.
   * 
   * 使用 this.props 和 this.state 来渲染 component 的结构。不能保证调用多少次，必须是幂等，没有 side effects
   *
   *   render: function() {
   *     var name = this.props.name;
   *     return <div>Hello, {name}!</div>;
   *   }
   *
   * @return {ReactComponent}
   * @nosideeffects
   * @required
   */
  render: SpecPolicy.DEFINE_ONCE,



  // ==== Delegate methods ====

  /**
   * Invoked when the component is initially created and about to be mounted.
   * This may have side effects, but any external subscriptions or data created
   * by this method must be cleaned up in `componentWillUnmount`.
   * 可能有 side effect，但是要注意，外部的订阅或者数据，必须在 componentWillUnmount 清理掉
   *
   * @optional
   */
  componentWillMount: SpecPolicy.DEFINE_MANY,

  /**
   * Invoked when the component has been mounted and has a DOM representation.
   * However, there is no guarantee that the DOM node is in the document.
   *
   * Use this as an opportunity to operate on the DOM when the component has
   * been mounted (initialized and rendered) for the first time.
   *
   * @param {DOMElement} rootNode DOM element representing the component.
   * @optional
   */
  componentDidMount: SpecPolicy.DEFINE_MANY,

  /**
   * Invoked before the component receives new props.
   *
   * Use this as an opportunity to react to a prop transition by updating the
   * state using `this.setState`. Current props are accessed via `this.props`.
   *
   *   componentWillReceiveProps: function(nextProps) {
   *     this.setState({
   *       likesIncreasing: nextProps.likeCount > this.props.likeCount
   *     });
   *   }
   *
   * NOTE: There is no equivalent `componentWillReceiveState`. An incoming prop
   * transition may cause a state change, but the opposite is not true. If you
   * need it, you are probably looking for `componentWillUpdate`.
   *
   * @param {object} nextProps
   * @optional
   */
  componentWillReceiveProps: SpecPolicy.DEFINE_MANY,

  /**
   * Invoked while deciding if the component should be updated as a result of
   * receiving new props and state.
   *
   * Use this as an opportunity to `return false` when you're certain that the
   * transition to the new props and state will not require a component update.
   *
   *   shouldComponentUpdate: function(nextProps, nextState) {
   *     return !equal(nextProps, this.props) || !equal(nextState, this.state);
   *   }
   *
   * @param {object} nextProps
   * @param {?object} nextState
   * @return {boolean} True if the component should update.
   * @optional
   */
  shouldComponentUpdate: SpecPolicy.DEFINE_ONCE,

  /**
   * Invoked when the component is about to update due to a transition from
   * `this.props` and `this.state` to `nextProps` and `nextState`.
   *
   * Use this as an opportunity to perform preparation before an update occurs.
   *
   * NOTE: You **cannot** use `this.setState()` in this method.
   *
   * @param {object} nextProps
   * @param {?object} nextState
   * @param {ReactReconcileTransaction} transaction
   * @optional
   */
  componentWillUpdate: SpecPolicy.DEFINE_MANY,

  /**
   * Invoked when the component's DOM representation has been updated.
   *
   * Use this as an opportunity to operate on the DOM when the component has
   * been updated.
   *
   * @param {object} prevProps
   * @param {?object} prevState
   * @param {DOMElement} rootNode DOM element representing the component.
   * @optional
   */
  componentDidUpdate: SpecPolicy.DEFINE_MANY,

  /**
   * Invoked when the component is about to be removed from its parent and have
   * its DOM representation destroyed.
   *
   * Use this as an opportunity to deallocate any external resources.
   *
   * NOTE: There is no `componentDidUnmount` since your component will have been
   * destroyed by that point.
   *
   * @optional
   */
  componentWillUnmount: SpecPolicy.DEFINE_MANY,



  // ==== Advanced methods ====

  /**
   * Updates the component's currently mounted DOM representation.
   *
   * By default, this implements React's rendering and reconciliation algorithm.
   * Sophisticated clients may wish to override this.
   *
   * @param {ReactReconcileTransaction} transaction
   * @internal
   * @overridable
   */
  updateComponent: SpecPolicy.OVERRIDE_BASE

};

/**
 * Mapping from class specification keys to special processing functions.
 *
 * Although these are declared in the specification when defining classes
 * using `React.createClass`, they will not be on the component's prototype.
 * 尽管这些会在定义 class 是声明，但是不会放到 component 的 prototype 上。
 */
var RESERVED_SPEC_KEYS = {
  displayName: function(Constructor, displayName) {
    Constructor.displayName = displayName;
  },
  mixins: function(Constructor, mixins) {
    if (mixins) {
      for (var i = 0; i < mixins.length; i++) {
        mixSpecIntoComponent(Constructor, mixins[i]);
      }
    }
  },
  props: function(Constructor, props) {
    Constructor.propDeclarations = props;
  }
};

/**
 * Custom version of `mixInto` which handles policy validation and reserved
 * specification keys when building `ReactCompositeComponent` classses.
 */
function mixSpecIntoComponent(Constructor, spec) {
  var proto = Constructor.prototype;
  for (var name in spec) {
    if (!spec.hasOwnProperty(name)) {
      continue;
    }
    var property = spec[name];
    var specPolicy = ReactCompositeComponentInterface[name];

    // Disallow overriding of base class methods unless explicitly allowed.
    if (ReactCompositeComponentMixin.hasOwnProperty(name)) {
      invariant(
        specPolicy === SpecPolicy.OVERRIDE_BASE,
        'ReactCompositeComponentInterface: You are attempting to override ' +
        '`%s` from your class specification. Ensure that your method names ' +
        'do not overlap with React methods.',
        name
      );
    }

    // Disallow using `React.autoBind` on internal methods.
    if (specPolicy != null) {
      invariant(
        !property || !property.__reactAutoBind,
        'ReactCompositeComponentInterface: You are attempting to use ' +
        '`React.autoBind` on `%s`, a method that is internal to React.' +
        'Internal methods are called with the component as the context.',
        name
      );
    }

    // Disallow defining methods more than once unless explicitly allowed.
    if (proto.hasOwnProperty(name)) {
      invariant(
        specPolicy === SpecPolicy.DEFINE_MANY,
        'ReactCompositeComponentInterface: You are attempting to define ' +
        '`%s` on your component more than once. This conflict may be due ' +
        'to a mixin.',
        name
      );
    }

    if (RESERVED_SPEC_KEYS.hasOwnProperty(name)) {
      RESERVED_SPEC_KEYS[name](Constructor, property);
    } else if (property && property.__reactAutoBind) {
      if (!proto.__reactAutoBindMap) {
        proto.__reactAutoBindMap = {};
      }
      proto.__reactAutoBindMap[name] = property.__reactAutoBind;
    } else if (proto.hasOwnProperty(name)) {
      // For methods which are defined more than once, call the existing methods
      // before calling the new property.
      proto[name] = createChainedFunction(proto[name], property);
    } else {
      proto[name] = property;
    }
  }
}

/**
 * Creates a function that invokes two functions and ignores their return vales.
 * 常见一个 function，可以顺序调用其他 function，但是会忽略掉 rtv
 * @param {function} one Function to invoke first.
 * @param {function} two Function to invoke second.
 * @return {function} Function that invokes the two argument functions.
 * @private
 */
function createChainedFunction(one, two) {
  return function chainedFunction(a, b, c, d, e, tooMany) {
    invariant(
      typeof tooMany === 'undefined',
      'Chained function can only take a maximum of 5 arguments.'
    );
    one.call(this, a, b, c, d, e);
    two.call(this, a, b, c, d, e);
  };
}

/**
 * `ReactCompositeComponent` maintains an auxiliary life cycle state in
 * `this._compositeLifeCycleState` (which can be null).
 *
 * 会维护一个辅助的 life cycle state 在 this._compositeLifeCycleState
 * This is different from the life cycle state maintained by `ReactComponent` in
 * `this._lifeCycleState`.
 * 和 ReactComponent 中的 this._lifeCycleState 有区别
 */
var CompositeLifeCycle = keyMirror({
  /**
   * Components in the process of being mounted respond to state changes
   * differently.
   */
  MOUNTING: null,
  /**
   * Components in the process of being unmounted are guarded against state
   * changes.
   */
  UNMOUNTING: null,
  /**
   * Components that are mounted and receiving new props respond to state
   * changes differently.
   */
  RECEIVING_PROPS: null,
  /**
   * Components that are mounted and receiving new state are guarded against
   * additional state changes.
   */
  RECEIVING_STATE: null
});

/**
 * @lends {ReactCompositeComponent.prototype}
 */
var ReactCompositeComponentMixin = {

  /**
   * Base constructor for all composite component.
   *
   * @param {?object} initialProps
   * @param {*} children
   * @final
   * @internal
   */
  construct: function(initialProps, children) {
    ReactComponent.Mixin.construct.call(this, initialProps, children);
    this.state = null;
    this._pendingState = null;
    this._compositeLifeCycleState = null;
  },

  /**
   * Initializes the component, renders markup, and registers event listeners.
   *
   * @param {string} rootID DOM ID of the root node.
   * @param {ReactReconcileTransaction} transaction
   * @return {?string} Rendered markup to be inserted into the DOM.
   * @final
   * @internal
   */
  mountComponent: function(rootID, transaction) {
    ReactComponent.Mixin.mountComponent.call(this, rootID, transaction);

    // Unset `this._lifeCycleState` until after this method is finished.
    this._lifeCycleState = ReactComponent.LifeCycle.UNMOUNTED;
    this._compositeLifeCycleState = CompositeLifeCycle.MOUNTING;

    if (this.constructor.propDeclarations) {
      this._assertValidProps(this.props);
    }

    if (this.__reactAutoBindMap) {
      this._bindAutoBindMethods();
    }

    this.state = this.getInitialState ? this.getInitialState() : null;
    this._pendingState = null;

    if (this.componentWillMount) {
      this.componentWillMount();
      // When mounting, calls to `setState` by `componentWillMount` will set
      // `this._pendingState` without triggering a re-render.
      if (this._pendingState) {
        this.state = this._pendingState;
        this._pendingState = null;
      }
    }

    if (this.componentDidMount) {
      transaction.getReactOnDOMReady().enqueue(this, this.componentDidMount);
    }

    this._renderedComponent = this._renderValidatedComponent();

    // Done with mounting, `setState` will now trigger UI changes.
    this._compositeLifeCycleState = null;
    this._lifeCycleState = ReactComponent.LifeCycle.MOUNTED;

    return this._renderedComponent.mountComponent(rootID, transaction);
  },

  /**
   * Releases any resources allocated by `mountComponent`.
   *
   * @final
   * @internal
   */
  unmountComponent: function() {
    this._compositeLifeCycleState = CompositeLifeCycle.UNMOUNTING;
    if (this.componentWillUnmount) {
      this.componentWillUnmount();
    }
    this._compositeLifeCycleState = null;

    ReactComponent.Mixin.unmountComponent.call(this);
    this._renderedComponent.unmountComponent();
    this._renderedComponent = null;

    if (this.refs) {
      this.refs = null;
    }

    // Some existing components rely on this.props even after they've been
    // destroyed (in event handlers).
    // TODO: this.props = null;
    // TODO: this.state = null;
  },

  /**
   * Updates the rendered DOM nodes given a new set of props.
   *
   * @param {object} nextProps Next set of properties.
   * @param {ReactReconcileTransaction} transaction
   * @final
   * @internal
   */
  receiveProps: function(nextProps, transaction) {
    if (this.constructor.propDeclarations) {
      this._assertValidProps(nextProps);
    }
    ReactComponent.Mixin.receiveProps.call(this, nextProps, transaction);

    this._compositeLifeCycleState = CompositeLifeCycle.RECEIVING_PROPS;
    if (this.componentWillReceiveProps) {
      this.componentWillReceiveProps(nextProps, transaction);
    }
    this._compositeLifeCycleState = CompositeLifeCycle.RECEIVING_STATE;
    // When receiving props, calls to `setState` by `componentWillReceiveProps`
    // will set `this._pendingState` without triggering a re-render.
    var nextState = this._pendingState || this.state;
    this._pendingState = null;
    this._receivePropsAndState(nextProps, nextState, transaction);
    this._compositeLifeCycleState = null;
  },

  /**
   * Sets a subset of the state. Always use this or `replaceState` to mutate
   * state. You should treat `this.state` as immutable.
   *
   * There is no guarantee that `this.state` will be immediately updated, so
   * accessing `this.state` after calling this method may return the old value.
   *
   * @param {object} partialState Next partial state to be merged with state.
   * @final
   * @protected
   */
  setState: function(partialState) {
    // Merge with `_pendingState` if it exists, otherwise with existing state.
    this.replaceState(merge(this._pendingState || this.state, partialState));
  },

  /**
   * Replaces all of the state. Always use this or `setState` to mutate state.
   * You should treat `this.state` as immutable.
   *
   * There is no guarantee that `this.state` will be immediately updated, so
   * accessing `this.state` after calling this method may return the old value.
   *
   * @param {object} completeState Next state.
   * @final
   * @protected
   */
  replaceState: function(completeState) {
    var compositeLifeCycleState = this._compositeLifeCycleState;
    invariant(
      this._lifeCycleState === ReactComponent.LifeCycle.MOUNTED ||
      compositeLifeCycleState === CompositeLifeCycle.MOUNTING,
      'replaceState(...): Can only update a mounted (or mounting) component.'
    );
    invariant(
      compositeLifeCycleState !== CompositeLifeCycle.RECEIVING_STATE &&
      compositeLifeCycleState !== CompositeLifeCycle.UNMOUNTING,
      'replaceState(...): Cannot update while unmounting component or during ' +
      'an existing state transition (such as within `render`).'
    );

    this._pendingState = completeState;

    // Do not trigger a state transition if we are in the middle of mounting or
    // receiving props because both of those will already be doing this.
    if (compositeLifeCycleState !== CompositeLifeCycle.MOUNTING &&
        compositeLifeCycleState !== CompositeLifeCycle.RECEIVING_PROPS) {
      this._compositeLifeCycleState = CompositeLifeCycle.RECEIVING_STATE;

      var nextState = this._pendingState;
      this._pendingState = null;

      var transaction = ReactComponent.ReactReconcileTransaction.getPooled();
      transaction.perform(
        this._receivePropsAndState,
        this,
        this.props,
        nextState,
        transaction
      );
      ReactComponent.ReactReconcileTransaction.release(transaction);

      this._compositeLifeCycleState = null;
    }
  },

  /**
   * Receives next props and next state, and negotiates whether or not the
   * component should update as a result.
   *
   * @param {object} nextProps Next object to set as props.
   * @param {?object} nextState Next object to set as state.
   * @param {ReactReconcileTransaction} transaction
   * @private
   */
  _receivePropsAndState: function(nextProps, nextState, transaction) {
    if (!this.shouldComponentUpdate ||
        this.shouldComponentUpdate(nextProps, nextState)) {
      // Will set `this.props` and `this.state`.
      this._performComponentUpdate(nextProps, nextState, transaction);
    } else {
      // If it's determined that a component should not update, we still want
      // to set props and state.
      this.props = nextProps;
      this.state = nextState;
    }
  },

  /**
   * Merges new props and state, notifies delegate methods of update and
   * performs update.
   *
   * @param {object} nextProps Next object to set as properties.
   * @param {?object} nextState Next object to set as state.
   * @param {ReactReconcileTransaction} transaction
   * @private
   */
  _performComponentUpdate: function(nextProps, nextState, transaction) {
    var prevProps = this.props;
    var prevState = this.state;

    if (this.componentWillUpdate) {
      this.componentWillUpdate(nextProps, nextState, transaction);
    }

    this.props = nextProps;
    this.state = nextState;

    this.updateComponent(transaction);

    if (this.componentDidUpdate) {
      transaction.getReactOnDOMReady().enqueue(
        this,
        this.componentDidUpdate.bind(this, prevProps, prevState)
      );
    }
  },

  /**
   * Updates the component's currently mounted DOM representation.
   *
   * By default, this implements React's rendering and reconciliation algorithm.
   * Sophisticated clients may wish to override this.
   *
   * @param {ReactReconcileTransaction} transaction
   * @internal
   * @overridable
   */
  updateComponent: function(transaction) {
    var currentComponent = this._renderedComponent;
    var nextComponent = this._renderValidatedComponent();
    if (currentComponent.constructor === nextComponent.constructor) {
      if (!nextComponent.props.isStatic) {
        currentComponent.receiveProps(nextComponent.props, transaction);
      }
    } else {
      // These two IDs are actually the same! But nothing should rely on that.
      var thisID = this._rootNodeID;
      var currentComponentID = currentComponent._rootNodeID;
      currentComponent.unmountComponent();
      var nextMarkup = nextComponent.mountComponent(thisID, transaction);
      ReactComponent.DOMIDOperations.dangerouslyReplaceNodeWithMarkupByID(
        currentComponentID,
        nextMarkup
      );
      this._renderedComponent = nextComponent;
    }
  },

  /**
   * Forces an update. This should only be invoked when it is known with
   * certainty that we are **not** in a DOM transaction.
   *
   * You may want to call this when you know that some deeper aspect of the
   * component's state has changed but `setState` was not called.
   *
   * This will not invoke `shouldUpdateComponent`, but it will invoke
   * `componentWillUpdate` and `componentDidUpdate`.
   *
   * @final
   * @protected
   */
  forceUpdate: function() {
    var transaction = ReactComponent.ReactReconcileTransaction.getPooled();
    transaction.perform(
      this._performComponentUpdate,
      this,
      this.props,
      this.state,
      transaction
    );
    ReactComponent.ReactReconcileTransaction.release(transaction);
  },

  /**
   * @private
   */
  _renderValidatedComponent: function() {
    ReactCurrentOwner.current = this;
    var renderedComponent = this.render();
    ReactCurrentOwner.current = null;
    invariant(
      ReactComponent.isValidComponent(renderedComponent),
      '%s.render(): A valid ReactComponent must be returned.',
      this.constructor.displayName || 'ReactCompositeComponent'
    );
    return renderedComponent;
  },

  /**
   * @param {object} props
   * @private
   */
  _assertValidProps: function(props) {
    var propDeclarations = this.constructor.propDeclarations;
    var componentName = this.constructor.displayName;
    for (var propName in propDeclarations) {
      var checkProp = propDeclarations[propName];
      if (checkProp) {
        checkProp(props, propName, componentName);
      }
    }
  },

  /**
   * @private
   */
  _bindAutoBindMethods: function() {
    for (var autoBindKey in this.__reactAutoBindMap) {
      if (!this.__reactAutoBindMap.hasOwnProperty(autoBindKey)) {
        continue;
      }
      var method = this.__reactAutoBindMap[autoBindKey];
      this[autoBindKey] = this._bindAutoBindMethod(method);
    }
  },

  /**
   * Binds a method to the component.
   *
   * @param {function} method Method to be bound.
   * @private
   */
  _bindAutoBindMethod: function(method) {
    var component = this;
    var hasWarned = false;
    function autoBound(a, b, c, d, e, tooMany) {
      invariant(
        typeof tooMany === 'undefined',
        'React.autoBind(...): Methods can only take a maximum of 5 arguments.'
      );
      if (component._lifeCycleState === ReactComponent.LifeCycle.MOUNTED) {
        return method.call(component, a, b, c, d, e);
      } else if (!hasWarned) {
        hasWarned = true;
        if (__DEV__) {
          console.warn(
            'React.autoBind(...): Attempted to invoke an auto-bound method ' +
            'on an unmounted instance of `%s`. You either have a memory leak ' +
            'or an event handler that is being run after unmounting.',
            component.constructor.displayName || 'ReactCompositeComponent'
          );
        }
      }
    }
    return autoBound;
  }

};

var ReactCompositeComponentBase = function() {};
mixInto(ReactCompositeComponentBase, ReactComponent.Mixin);
mixInto(ReactCompositeComponentBase, ReactOwner.Mixin);
mixInto(ReactCompositeComponentBase, ReactPropTransferer.Mixin);
mixInto(ReactCompositeComponentBase, ReactCompositeComponentMixin);

/**
 * Module for creating composite components.
 *
 * @class ReactCompositeComponent
 * @extends ReactComponent
 * @extends ReactOwner
 * @extends ReactPropTransferer
 */
var ReactCompositeComponent = {

  LifeCycle: CompositeLifeCycle,

  Base: ReactCompositeComponentBase,

  /**
   * Creates a composite component class given a class specification.
   *
   * @param {object} spec Class specification (which must define `render`).
   * @return {function} Component constructor function.
   * @public
   */
  createClass: function(spec) {
    var Constructor = function(initialProps, children) {
      this.construct(initialProps, children);
    };
    Constructor.prototype = new ReactCompositeComponentBase();
    Constructor.prototype.constructor = Constructor;
    mixSpecIntoComponent(Constructor, spec);
    invariant(
      Constructor.prototype.render,
      'createClass(...): Class specification must implement a `render` method.'
    );

    var ConvenienceConstructor = function(props, children) {
      return new Constructor(props, children);
    };
    ConvenienceConstructor.componentConstructor = Constructor;
    ConvenienceConstructor.originalSpec = spec;
    return ConvenienceConstructor;
  },

  /**
   * Marks the provided method to be automatically bound to the component.
   * This means the method's context will always be the component.
   *
   *   React.createClass({
   *     handleClick: React.autoBind(function() {
   *       this.setState({jumping: true});
   *     }),
   *     render: function() {
   *       return <a onClick={this.handleClick}>Jump</a>;
   *     }
   *   });
   *
   * @param {function} method Method to be bound.
   * @public
   */
  autoBind: function(method) {
    function unbound() {
      invariant(
        false,
        'React.autoBind(...): Attempted to invoke an auto-bound method that ' +
        'was not correctly defined on the class specification.'
      );
    }
    unbound.__reactAutoBind = method;
    return unbound;
  }

};

module.exports = ReactCompositeComponent;
```


---

## 停一下 & 回顾

2016年02月24日22:09:36

React 之前的读法，有点类似于是学习细节。

下一步想换个思路：

- 一是不断学习 TypeScript 的语法，不断重构
- 理解 React 的思路，一条线走下去（完成一个最简单的功能）
- 然后实现
- 然后不断填充细节
- 不断重构

### pseudo-react

React.createClass(

React.createFactory(

ReactDOM.render(
