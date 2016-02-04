title: Read React Structure
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

在要阅读源码前。Composite Component 是有相关文档的，先看下文档。


