title: Read recompose API
date: 2016-04-23 07:14:48
tags:
---

# Example

## 数据流

这种方式不好用。

```js
const Counter = withState(
  'counter', 'setCounter', 0
)(({ counter, setCounter }) => (
    <div>
      Count: {counter}
      <button onClick={() => setCounter(n => n + 1)}>Increment</button> />
      <button onClick={() => setCounter(n => n - 1)}>Decrement</button> 
    </div>
  )
)
```

√ 这种方式符合 redux-style，目前够清晰。

```js
const counterReducer = (count, action) => {
  switch (action.type) {
  case INCREMENT:
    return count + 1
  case DECREMENT:
    return count - 1
  default:
    return count
  }
}

const Counter = withReducer(
  'counter', 'dispatch', counterReducer, 0
)(({ counter, dispatch }) => (
    <div>
      Count: {counter}
      <button onClick={() => dispatch({ type: INCREMENT })}>Increment</button> />
      <button onClick={() => dispatch({ type: DECREMENT })}>Decrement</button>
    </div>
  )
)
```

## 性能

```js
// A component that is expensive to render
const ExpensiveComponent = ({ propA, propB }) => {...}

// Optimized version of same component, using shallow comparison of props
// Same effect as React's PureRenderMixin
const OptimizedComponent = pure(ExpensiveComponent)

// Even more optimized: only updates if specific prop keys have changed
const HyperOptimizedComponent = onlyUpdateForKeys(['propA', 'propB'])(ExpensiveComponent)
```


# API

## Pattern

```js
const EnhancedComponent = hoc(BaseComponent)
```

```js
const composedHoc = compose(hoc1, hoc2, hoc3)

// Same as
const composedHoc = BaseComponent => hoc1(hoc2(hoc3(BaseComponent)))

// Same as
@withState(/*...args*/)
@mapProps(/*...args*/)
@pure
class Component extends React.Component {...}
```

## 
