title: Learn Redux
date: 2016-01-16 13:01:29
tags:
---

## Provider

### 使用场景

```js
<Provider store={store}>
    <Router history={history}>
      <Route component={AppContainer} path='/'/>
    </Router>
  </Provider>
```

然后必须和 connect 结合使用。

```js
function select(state) {
  return {
    counters: state.counters
  };
}

@connect(select)
export class AppContainer extends Component<IAppContainerProps, {}> {
  public render() {
    const { dispatch } = this.props;
    const actions = bindActionCreators(CounterActions, dispatch);
    return (
      <App name="John" actions={actions}/>
    );
  }
}
```

### 源码

仅提取核心部分：

```js
const { Component, Children } = require('react')

class Provider extends Component {
  // 表示 this.store 会被 Provider 的 children 通过 context 获取
  getChildContext() {
    return { store: this.store }
  }

  constructor(props, context) {
    super(props, context)
    this.store = props.store
  }

  render() {
    let { children } = this.props
    // 仅仅是对 children 的无 DOM 包装
    return Children.only(children)
  }
}

module.exports = Provider

```

## connect

### 使用场景

使用场景已经介绍过:

```js
function select(state) {
  return {
    counters: state.counters
  };
}

connect(select)(view)
...
```

这里 connect 是 curry 高阶函数。

### 源码

```js
function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
    return function wrapWithConnect(WrappedComponent) {
        class Connect extends Component {
        }
        // Copies non-react specific statics from a child component to a parent component. Similar to Object.assign, but with React static keywords blacklisted from being overridden.
        // https://github.com/mridgway/hoist-non-react-statics
        // hoistNonReactStatics(targetComponent, sourceComponent)
        // 即将被 connect wrapper 的 component 的 keys，除去 contextTypes，mixins，defaultProps，displayName 等，复制到 Connect 上，并返回 Connect
        return hoistStatics(Connect, WrappedComponent)
    }
}
module.exports = connect;
```
