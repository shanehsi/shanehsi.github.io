title: Redux APIs
date: 2016-03-15 17:02:36
tags:
---

因为我在使用 rxjs 自己做一个简单的 flux。主要目的是做精细的 stream 控制。

这里想借鉴下 Redux 的 API 设计。

从例子看：

```js
render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root') />
)
```

store 是通过 context 全局可见的。

```js
function mapStateToProps(state) {
  return {
    counter: state.counter
  }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators(CounterActions, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```

connect 里主要的是 mapStateToProps。类似于 cursor 的功能。

## API Design 2016-03-15 17:11:11

### store 通过 context 传递

```js
state$.subscribe(function (state) {
  ReactDOM.render(
    <Component />,
    document.getElementById('root'));
});
```

变成？

```js
state$.subscribe( state => render(
    <Provider store={state}>
        <Component />    
    </Provider> ,  document.getElementById('root')   />
));
```

让 state 全局可见。不用显式往下传递。

### cursor 拿值

view 直接修改 store（或者调用 store 里的方法）。暂时不觉得 actions 有什么特别大的作用。

```js
import React, { Component } from 'react'

@cursor(state => {
    return state.counter;
})
class Counter extends Component<any, any> {
  constructor(props) {
    super(props);
  }

  render() {
    const { value, onIncrement, onDecrement } = this.props;
    onIncrement(<mutations>);
    return (
      <p>
      </p> />
    );
  }
}

export default Counter;
```

因为 store 已经全局可见了。所以 `cursor` 肯定是可以拿到值的。

counter 的 model，或者 store

```js
<Provider store={state} animateStore={}>
    <Component />    
</Provider> />
```

还是使用 redux 的 API 吧。只不过是使用 RxJS 来实现。

{% asset_img diagram.png %}

