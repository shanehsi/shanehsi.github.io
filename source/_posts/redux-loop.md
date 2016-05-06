title: redux-loop
date: 2016-04-28 16:00:50
tags:
---

sequence your effects naturally and purely by returning them from your reducers.

> Doesn't redux-loop put side-effects in the reducer?

It doesn't. The values returned from the reducer when **scheduling an effect** with redux-loop only **describe** the effect. Calling the reducer will **not cause the effect to run**. The value returned by the reducer is just an object that the store knows how to interpret when it is enhanced by redux-loop. You can safely call a reducer in your tests **without worrying about waiting for effects to finish** and what they will do to your environment.

只是一个形容，一个占位。

## quick example

```js
//... 略去

const firstAction = {
  type: 'FIRST_ACTION',
};

const doSecondAction = (value) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        type: 'SECOND_ACTION',
        payload: value,
      });
    });
  });
}

const thirdAction = {
  type: 'THIRD_ACTION',
};

// 这个还是蛮牛逼的
// immutable store state allowed by default, but not required
const initialState = fromJS({
  firstRun: false,
  secondRun: false,
  thirdRun: false,
});

function reducer(state, action) {
  switch(action.type) {

  case 'FIRST_ACTION':
    // 序列是： 首先是 first 然后是 batch 到一个 promise，一个 constant 
    // Enter a sequence at FIRST_ACTION, SECOND_ACTION and THIRD_ACTION will be
    // dispatched in the order they are passed to batch
    return loop(
      state.set('firstRun', true),
      Effects.batch([
        Effects.promise(doSecondAction, 'hello'),
        Effects.constant(thirdAction)
      ])
    );

  case 'SECOND_ACTION':
    return state.set('secondRun', action.payload);

  case 'THIRD_ACTION':
    return state.set('thirdRun', true);

  default:
    return state;
  }
}

// Note: passing enhancer as the last argument to createStore requires redux@>=3.1.0
const store = createStore(reducer, initialState, install());

store
  .dispatch(firstAction)
  .then(() => {
    // dispatch returns a promise for when the current sequence is complete
    // { firstRun: true, secondRun: 'hello', thirdRun: true }
    console.log(store.getState().toJS());
  });
```

## Why use this?

Having used and followed the progression of Redux and the Elm Architecture, and after trying other effect patterns for Redux, we came to the following conclusion:

> Synchronous state transitions caused by returning a new state from the reducer in response to an action are just one of all possible effects an action can have on application state.
> 同步的状态变化一般是 reducer 响应 action，这只是其中 action 能给到 application state 的 effects 的一种。

Many other methods for handling effects in Redux, especially those implemented with action-creators, incorrectly teach the user that asynchronous effects are fundamentally different from synchronous state transitions. 

很多 redux 处理 effects 的方法，特别是通过 actionCreators 实现的，错误的教给了用户，异步 effects 和同步是本质不同的。

This separation encourages divergent and increasingly specific means of processing particular types effects. 

Instead, we should focus on making our reducers powerful enough to handle asynchronous effects as well as synchronous state transitions. 

相反的，我们应该让 reduer 能处理异步，就像处理同步一样。

With redux-loop, the reducer doesn't just decide what happens now due to a particular action, it decides what happens next. All of the behavior of your application can be traced through one place, and that behavior can be easily broken apart and composed back together. This is one of the most powerful features of the Elm architecture, and with redux-loop it is a feature of Redux as well.

这样，redcuer 除了响应 action，也能决定下一步做什么。

这些都是 behaviror，都能在一处 traced。这些 behavior 都可以被轻松的 分解和组合。这就是 redux 的强度特性之一。


### Write a reducer with some effects

```js
import { Effects, loop } from 'redux-loop';
import { loadingStart, loadingSuccess, loadingFailure } from './actions';

export function fetchDetails(id) {
  return fetch(`/api/details/${id}`)
    .then((r) => r.json())
    .then(loadingSuccess)
    .catch(loadingFailure);
}

export default function reducer(state, action) {
  switch (action.type) {
    case 'LOADING_START':
      return loop(
        { ...state, loading: true },
        Effects.promise(fetchDetails, action.payload.id)
      );

    case 'LOADING_SUCCESS':
      return {
        ...state,
        loading: false,
        details: action.payload
      };

    case 'LOADING_FAILURE':
      return {
        ...state,
        loading: false,
        error: action.payload.message
      };

    default:
      return state;
  }
}
```

返回的是 `loop`。

A loop joins an updated model state with an effect for the store to process

组合了一个被更新过的 model state，和一个 effect。store 会之后处理。

Effects are declarative specifications of the next behavior of the store.


## Avoid circular loops!

```js
function reducer(state, action) {
  switch (action.type) {
    case 'FIRST':
      return loop(
        state,
        Effects.constant(second())
      );

    case 'SECOND':
      return loop(
        state,
        Effects.constant(first())
      );
  }
}
```

This minimal example will cause perpetual dispatching! While it is also possible to make this mistake with large, complicated networks of redux-thunk action creators, it is much easier to spot the mistake before it is made. It helps to keep your reducers small and focused, and use combineReducers or manually compose reducers so that the number of actions you deal with at one time is small. 

A small set of actions which initiate a loop will help reduce the likelihood of causing circular dispatches.




