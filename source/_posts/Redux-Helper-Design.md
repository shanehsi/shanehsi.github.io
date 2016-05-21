title: Redux Helper Design
date: 2016-05-13 14:03:57
tags:
---

目前的设计, actions: 

```js
export default class ActionFactory {
  protected namespace: any
  protected defaultActions: any

  constructor(ns) {
    this.namespace = ns
  }

  createAction(type, payload?) {
    return {
      type: `${this.namespace}/${type}`,
      payload,
    }
  }

  actions() {
    return this.defaultActions
  }
}
```

就是调用了 createAction 这个方法. 

现在有一个问题是, OO 的设计模式如何体现.

```js
import * as _ from 'lodash'
import ActionFactory from './ActionFactory'

export default class CommonActionFactory extends ActionFactory {
  constructor(ns) {
    super(ns)
    this.defaultActions = {
      set: (path, value) => {
        return this.createAction('set', {path, value})
      },
      openModal: (key) => {
        return this.createAction('openModal', {key})
      },
      closeModal: (key) => {
        return this.createAction('closeModal', {key})
      },
      start: (key) => {
        return this.createAction('start', {key})
      },
      end: (key) => {
        return this.createAction('end', {key})
      },
    }
  }

  static isShow = function (state, key) {
    return _.get(state, ['view', 'showModal', key])
  }

  static isPending = function (state, key) {
    return _.get(state, ['view', 'pending', key])
  }
}
```

对于 Form

```js
import ActionFactory from './ActionFactory'

export default class FormActionFactory extends ActionFactory {
  constructor(ns) {
    super(ns)
    this.defaultActions = {
      setField: (path, value) => {
        return this.createAction('setField', {path, value})
      },
      reset: (path, value) => {
        return this.createAction('reset', {path, value})
      },
      resetError: (path, value) => {
        return this.createAction('resetError', {path, value})
      },
      setError: (path, value) => {
        return this.createAction('setError', {path, value})
      },
      setPending: (path, value) => {
        return this.createAction('setPending', {path, value})
      },
    }
  }
}
```



对于 ReducerFactory:

```js
import * as _ from 'lodash'

export default class ReducerFactory {
  protected namespace: any
  protected defaultHandlers: any = {}
  protected handlers: any = {}
  protected initialState: any = {}

  constructor(ns) {
    this.namespace = ns
  }

  setInitialState(initialState) {
    this.initialState = _.merge({}, this.initialState, initialState)
    return this
  }

  setHandlers(handlers) {
    this.handlers = handlers
    return this
  }

  getReducer() {
    const finalHandlers = _.merge({}, this.defaultHandlers, this.handlers)

    return (state, action) => {
      if (!Boolean(state)) {
        state = this.initialState
      }
      if (finalHandlers.hasOwnProperty(action.type)) {
        return finalHandlers[action.type](state, action)
      }
      return state
    }
  }
}
```

然后子类:

```js
import * as _ from 'lodash'
import ReducerFactory from './ReducerFactory'
const update = require('./update')

export default class FormReducerFactory extends ReducerFactory {
  constructor(ns) {
    super(ns)

    this.setInitialState({
      view: {},
      form: {},
      model: {},
    })

    this.defaultHandlers = {
      [`${this.namespace}/setField`]: (state, action) => {
        return _.set(_.merge({}, state), ['form'].concat(action.payload.path), action.payload.value)
      },
      [`${this.namespace}/reset`]: (state, action) => {
        return update(state, {
          form: {$set: state.model},
        })
      },
      [`${this.namespace}/resetError`]: (state, action) => {
        return update(state, {
          validate: {$set: {}},
        })
      },
      [`${this.namespace}/setError`]: (state, action) => {
        return _.set(_.merge({}, state), `validate.${action.payload.path}`, action.payload.value)
      },
      [`${this.namespace}/setPending`]: (state, action) => {
        return _.set(_.merge({}, state), `field.${action.payload.path}.pending`, action.payload.value)
      },
    }
  }
}
```

先来看下 Alt 的设计, 来获取下如何设计 API 的想法.

## 一些基本的想法

Redux 的 atom store, Redux 的生态( HMR, 一些 dev tools)是必须利用的.

Redux 的本质也很简单, 一个回调函数的注册. 

我想要做的, 只是让写法能更模块化.
这个底层暂时也不要改. 改了也不影响上层. 

## Alt

Actions 暂时还是做吧. 但是 Actions 可以封装起来. 如何传给 Component 还是问题. 用 HOC ?

```js
this.props.actions.doSomething
```

```js
generateActions() // 快速生成 actions
```

还有一点, Actions 要能够对异步请求做一层封装, 不要每次都这么多代码.

```js
actions.async
.setPendingKey('loadBasicInfo')
(async.loadBasicInfo))
.end()
.fail()

actins.isPending('loadBasicInfo')
```

而且要思考下, 什么情况下会 cancel 一个请求.


action 有一个 id.

```js
action.id
```

### Store

也就是我想给 Reducer 加的

这里面就有一个神奇的(pull way)

```js
class MyStore {
  constructor() {
    this.bindAction(MyActions.FOO, this.handleFoo);
  }

  handleFoo(data) {
    // do something with data
  }
}
```

```js
class MyStore {
  constructor() {
    this.bindActions(MyActions);
  }

  onFoo(data) {
    // do something with data
  }
}

```

它还能反过来(push way):


```js
class MyStore {
  constructor() {
    this.bindListeners({
      handleFoo: MyActions.FOO,
      handleBar: [MyActions.BAR, OtherActions.BAR]
    });
  }

  handleFoo(data) {
    // will only be called by MyActions.foo()
  }

  handleBar(data) {
    // will be called by MyActions.bar() and OtherActions.bar()
  }
}
```

并且它也封装了 `update`, 当然它在 alt 中, 主要作用是 emit change

```js
class MyStore {
  handleFoo() {
    this.state = { foo: 0 };

    setTimeout(() => {
      // set foo to 1 and emit a change.
      this.setState({
        foo: 1
      });
    }, 100);

    // supress emitting a change.
    this.preventDefault();
  }
}
```

来看下 Alt 对 Data Sources 的处理:

它的 API 设计:

```

```

我觉得要给值做一个 box:

```js
this.props.xxx.isLoading

value([xx,xx,xx])
get(['xx',xx,xx]).value
get(['xx',xx,xx]).loading

{$loading, $error, $expire, value: xx}
```

这个我们可以设置几个 box, 只有 box 才有 meta data. 比如缓存, 只有在这个级别可以做 $expire.

首次加载, 首次加载的时机, 可以在 route 时候做. 或者监听 quick click 事件 之类的. 这个时候感觉必须用 observable 做, 或者 event emitter.

```js
this.model.firstLoad()
```

 其他的异步:

 ```js
 actions.async
.setPendingKey('loadBasicInfo')
.setAction(async.loadBasicInfo))
.end()
.fail()
 ```

 大概打开了一些思路, 准备看看 react-redux-form

 ```js

// 其中 alt 自动绑定了 store

@alt.createActions()
class FormActions extends Actions {
    constuctor() {
        this.generateActions(['xx', 'xx', 'xx'])
    }

    updateTodo(id, text) {
        return { id, text }
    }
}

FormActions.get('xx') // 不依赖动态特性
FormActions.updateTodo()
 ```

需要研究下 RelayContainer 是如何处理异步的.

可能一个思路:

```js
@alt.createActions()
class FormActions extends Actions {
    constuctor() {
        this.generateActions(['xx', 'xx', 'xx'])
    }

    updateTodo(id, text) {
        return { id, text }
    }
}

@formActions('xx')

@compose(connect(state => state.xx), [commonActions, formActions])

```

## 首次加载数据的方式

暂时适用 componetDidMount 无所谓

```js
// box data
basicInfo: {
    value: {

    },
    $expire: true, // -1 代表还没有初始化
    $pending: false,
    $showPending: 计算属性: 不需要 expire 就不显示, 否则就再判断下 $pending
}
```


很想改写下 `get`, `set` 这些 property descriptor.

## 看下 

## 简单看下 OO Design Pattern

