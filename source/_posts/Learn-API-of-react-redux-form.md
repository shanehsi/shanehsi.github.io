title: Learn API of react redux form
date: 2016-05-13 15:33:28
tags:
---

Filed 组件:

```js
import { Field } from 'react-redux-form';

// in your component's render() method
<Field model="user.name">
  <input type="text" /> />
</Field>
```

`Field` 其实是一个 connect.

提供的功能:

- handles control updates, such as focus, blur, pristine, etc. 可以控制 focus blur 这些

- keeps track of validity on any part of your model

看下 example, 我着重想知道的, 是如何将值从 model -> form 的.

```js

// app.js

import React from 'react';
import { combineReducers, createStore } from 'redux';
import { Provider } from 'react-redux';
import { modelReducer, formReducer } from 'react-redux-form';

import LoginForm from './forms/login-form';

const store = createStore(combineReducers({
  user: modelReducer('user'),
  userForm: formReducer('user')
}));

export default class App extends React.Component {
  render() {
    return (
      <Provider store={ store }>
        <LoginForm /> />
      </Provider>
    )
  }
}

// forms/login-form.js

import React from 'react';
import { connect } from 'react-redux';
import { Field } from 'react-redux-form';

class LoginForm extends React.Component {
  render() {
    let { user } = this.props;

    return (
      <form>
        <Field model="user.username">
          <label>Username</label>
          <input type="text" /> />
        </Field>

        <Field model="user.password">
          <label>Password</label>
          <input type="password" /> />
        </Field>

        <button>
          Log in as { user.username }
        </button>
      </form>
    )
  }
}

const selector = (state) => ({ user: state.user });

export default connect(selector)(LoginForm);

```

## 如何设置

5. Setup your store.

- modelReducer(model, initialState) will create a model reducer, and
- formReducer(model, initialState) will create a form reducer.


他这里的 form reducer 是可选的, 主要功能是:

Form reducers are always optional. If you are not concerned with field states such as focus, blur, pristine, valid, etc., you can omit it (especially for performance purposes).


# Reducer

Model reducer 是响应 `change()` action 的

Form reducer 是响应 filed actions 的.

# 校验

这个待会再看

# 跟踪 collections


