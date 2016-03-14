title: Redux RxJS
date: 2016-03-14 15:53:21
tags:
---

使用 RxJS 的目的是为了引入 stream 的操作。

- actions 作为 mutations 保留，并且 view 对 actions 是 imperative。
- store 监听 action。

- stream 在进入 store 之前 merge 一次。
- store 到 view，最简单的是通过 throttle，交给 requestAnimationFrame。
- （但是如果有 animation store，可以在加入一层 combine）


附：

[Using RxJS for data flow instead of Flux with React](https://gist.github.com/justinwoo/08f9f8fcdcf865025f18)

Reposted from Qiita

For almost a year now, I've been using this "flux" architecture to organize my React applications and to work on other people's projects, and its popularity has grown quite a lot, to the point where it shows up on job listings for React and a lot of people get confused about what it is.


# Why I'm tired of using and teaching flux

There are a billion explainations on the internet, so I'll skip explaining the parts. Instead, let's cut to the chase -- the main parts I hate about flux are the Dispatcher and the Store's own updating mechanism.

If you use a setup similar to the examples in facebook/flux, and you use flux.Dispatcher, you probably have this kind of flow:

1. You have a method on an `Action` object that you call, which then does `Dispatcher.prototype.dispatch(payload)`, where `payload` is `actionType: String, {...}`
2. flux.Dispatcher dispatches only that single payload, and doesn't let anything else happen. It goes through registered callbacks, calling each one with this payload.
3. All of your registered callbacks fire off, in your `Store` objects' closures. The values in your Stores' closures get set and then you call `Store.emitChange`.
4. `Store.emitChange` then uses `EventEmitter.prototype.emit` to call callbacks you registered with `EventEmitter.prototype.addListener` for whatever key you used, like `'change'` or something.
5. Your callback was probably defined in your component, in which case it would then call `this.setState` but also have to call `Store.getState` to retrieve your application's state.

No wonder there are tons of questions about flux, because seriously, what the flux is this? And if you think this is just extra information nobody cares about, guess what you have to think and worry about when you need to have all of the separate store states come together to make up your whole application state? When you have one store update that then triggers another action to update another store? Or you could also just add an event listener to that Store's emitChange so that you can update your dependent store and emit a change event too.

And so at the end of the day, all the store does is use its callbacks on the Dispatcher to receive actions, act on them, emit its own events to consume itself, and then call callbacks which call methods on the Store to update your components.

I haven't seen any flux library implement anything that doesn't look like this. It may not use flux.Dispatcher (though most do) or EventEmitter (though, a lot of them do also), but they almost always come down to this same system, and the only benefit they provide is that you don't have to provide your own Dispatcher or Store implementation (in which you consume theirs, which almost always works in the same way).

Did I mention I also hate how you're supposed to load up your application state? Most of the time, you don't want to render without having your Store's state, so you have to use `getInitialState` to do it. What ends up happening, inevitably, is that you end up with a method called `getAppState` which does `return Store.getAppState()` and you use that in `getInitialState` and your `onChange` handler which probably is `this.setState(getAppState())` and will get called back like so in your Store. Ugh.

这个还好，不是因为 getInitalState()，而是因为就要从 store 中获取 state。在 redux 中也用 selector 实现。

# Using RxJS instead

So instead of all that crap we get in our flux setup, let's just use RxJS. A few things you'll want to know:

1. [Rx.Observable](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md) -- these are the observable "streams" that you can subscribe to, where the value coming down is either what is provided by the source, or transformed by something.
2. [Rx.Observable.prototype.subscribe](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/subscribe.md) -- this is what you use to register an observer in the end to the values coming out of your stream.
3. [Rx.Subject](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/subject.md) -- these are Observables that provide some other features also, but I'm mostly concerned with onNext, which lets me publish the newest value of my stream.

## A note about using ReplaySubject

Instead of doing the awkward thing of figuring out how to load up our application state and then observe our stream, I think we should just use the ReplaySubject with a buffer of 1, i.e., when we subscribe to this, we want the latest value sent to us to use it as if we are getting fresh data from the stream.

1 代表最近的一次值。

## Actually building our application

For our simple example, I'm just going to build a clicking application.

### Setting up Keys

For a variety of reasons, we want to use keys to identify Intents to our Model so it can handle data changes appropriately.

```js
var keyMirror = require('keymirror');

module.exports = keyMirror({
  INCREMENT_COUNTER: null
});
```

能不能不要这个 key 的逻辑？直连而不是先存储，再根据 key 检索。

### Seting up our Intents

Instead of using a dispatcher, we're just going to provide a stream of user Intents and let our Model handle them as necessary. I'm just going to use a ReplaySubject since it's easy enough to use, though we will likely never need a replay.

```js
var Rx = require('rx');
var Keys = require('./keys');
var intentSubject = new Rx.ReplaySubject(1);

module.exports = {
  subject: intentSubject,

  incrementCounter: function () {
    intentSubject.onNext({
      key: Keys.INCREMENT_COUNTER
    });
  }
};
```

And so you can see that when `Intent.incrementCounter()` is called, our subject will publish a payload.

这里我还是叫做 actions 吧。只是包装成 stream 了而已。

### Setting up our Model

Now comes the interesting part, where we need to subscribe to our Intents so we can handle them appropriately. This is so easy, it's boring.

```js
var Rx = require('rx');
var update = require('react/lib/update');
var Keys = require('./keys');
var Intent = require('./intent');

var subject = new Rx.ReplaySubject(1);

var state = {
  counter: 0
};

// 这个是 helpers 可以单独拿出来
function incrementCounter() {
  state = update(state, {
    $merge: {
      counter: state.counter + 1
    }
  });
  subject.onNext(state);
}

Intent.subject.subscribe(function (payload) {
  switch(payload.key) {
    case Keys.INCREMENT_COUNTER:
      incrementCounter();
      break;
    default:
      console.warn(`${payload.key} not recognized in model.`);
  }
});

subject.onNext(state);

module.exports = {
  subject: subject
};
```

And so we can see that the Model has its own ReplaySubject(1), and handled intents will make our subject publish our new application state.

### Setting up our root component

So our Store was dead easy to set up. Surely, there's going to be a lot of work to set up our root component, right? Well, I really hate having to mess with component state, so I decided to just shove it all into props instead.

// 我也是，不要 state。

```js
var React = require('react');
var Rx = require('rx');
var Model = require('./model');
var Root = require('./views/root');

Model.subject.subscribe((appState) => {
  React.render(
    <Root {...appState}/>, document.getElementById('app') />
  );
});
```

`Model.subject` is a `Rx.Observable`, and by using `Rx.Observable.prototype.subscribe`, we are able to subscribe an observer which renders our application using the appState data published to it. And because we used a `ReplaySubject(1)`, the initializing `subject.onNext(state)` in our model plays back, rendering our application. This part is pretty boring too, since we didn't write much code at all.

In the end, it looks like this (after clicking the button some):

![kobito.1430010588.424467.png](https://qiita-image-store.s3.amazonaws.com/0/42481/1cd3d418-b816-4c7f-3f61-af73d1895b2a.png "kobito.1430010588.424467.png")

Let's look at some other demos since this was so short and boring.

# Dealing with multiple Models

What if we have multiple Models that we need to subscribe to in order to get our application state? Well, in that case, we do need another Model and its stream.

```js
var Rx = require('rx');
var update = require('react/lib/update');

var YahharoKeys = require('../keys/yahharo-keys');
var YahharoIntent = require('../intents/yahharo-intent');

var subject = new Rx.ReplaySubject(1);

var yahharo = 'やっはろー';
var haroharo = 'はろはろー';

var state = {
  greeting: yahharo
};

function switchGreeting() {
  state = update(state, {
    $merge: {
      greeting: state.greeting === yahharo ? haroharo : yahharo
    }
  });
  subject.onNext(state);
}

YahharoIntent.subject.subscribe(function (payload) {
  switch(payload.key) {
    case YahharoKeys.SWITCH_GREETING:
      switchGreeting();
      break;
    default:
      console.warn(`${payload.key} not recognized in model.`);
  }
});

subject.onNext(state);

module.exports = {
  subject: subject
};
```

But how do we handle the logic for combining the two streams? Well, time to call [Rx.Observable.combineLatest](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/combinelatest.md):

```js
var React = require('react');
var Rx = require('rx');

var CounterModel = require('./models/counter-model');
var YahharoModel = require('./models/yahharo-model');
var Root = require('./views/root');

var AppObservable = Rx.Observable.combineLatest(
  CounterModel.subject,
  YahharoModel.subject,
  function (
    CounterState,
    YahharoState
  ) {
    return {
      CounterState: CounterState,
      YahharoState: YahharoState
    };
  }
);

AppObservable.subscribe((appState) => {
  React.render(
    <Root {...appState}/>,
    document.getElementById('app')
  );
});
```

As we can see, we can just use RxJS to take the latest data from all three and combine them into a new Observable to feed into our Root component:

![kobito.1430010899.143385.png](https://qiita-image-store.s3.amazonaws.com/0/42481/2e866e3c-afcf-cd3e-8ace-136700114980.png "kobito.1430010899.143385.png")

Let's do another since this one was pretty short too.

# Dependent events

This one is pretty annoying for me currently, setting up a store that only does logical operations, and then having its emitChange trigger another store that does calculations based on it.

Let's consider this:

```js
var state = {
  counter: 0,
  list: [],
  filterEvens: true
};

function incrementCounter() {
  state = update(state, {
    $merge: {
      counter: state.counter + 1,
      list: state.list.concat(state.counter)
    }
  });
  subject.onNext(state);
}
```

That `filterEvens` will get switched between true and false, for filtering out even values (of course, in the real world, I have a list of predicates that will go through a big list to figure out what gets rendered and gets unmounted).

Turns out, this is one of the most common operations in Rx. To write a transform that applies to each published event, we end up using [Rx.Observable.prototype.map](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/select.md):

```js
var React = require('react');
var Rx = require('rx');
var update = require('react/lib/update');

var Model = require('./model');
var Root = require('./views/root');

var FilteredObservable = Model.subject.map(function (appState) {
  var filteredList = appState.list.filter(function (item) {
    var isEven = item % 2 === 0
    return appState.filterEvens ? !isEven : isEven;
  });
  return update(appState, {
    $merge: {
      filteredList: filteredList
    }
  });
});

FilteredObservable.subscribe((appState) => {
  React.render(
    <Root {...appState}/>,
    document.getElementById('app')
  );
});
```

Again, we get a new Observable that we then subscribe to in order to render our application with the filtered data:

![kobito.1430011586.218478.png](https://qiita-image-store.s3.amazonaws.com/0/42481/a8ec0665-6ff6-eb6e-087d-2577007ba9e9.png "kobito.1430011586.218478.png")


# In Closing

I know I probably turned some people off talking about how much I hate flux when it's honestly done a lot to change how people think about data flow in their applications. For that, flux remains quite significant to me and I think it's done quite a lot of good, but I am no longer content to use it to manage data flows in my applications. If you think otherwise, I hope you might consider taking the time to write to me on why you think I should change my practices when using flux to make it a more enjoyable and productive experience.

Otherwise, if you were intrigued by the use of RxJS and want to see the source code for this article, see the repo here: https://github.com/justinwoo/react-rxjs-flow

If you made it this far, thanks for reading!

## Update - Apr 26

Do you have thoughts about using a Rx.Subject for each intent rather than using the intent keys system? I did the keys-based system for this demonstration, but I'm still on the fence about whether or not I'm going to use keys to identify payloads or use separate subjects. Let me know on [twitter](https://twitter.com/jusrin00) what you think!

## Update - Apr 28

I pushed another branch that gets rid of the Keys-based single subject with multiple subjects that can be subscribed to here: https://github.com/justinwoo/react-rxjs-flow/tree/each-intent-subject. Seems pretty nice, thanks to [@saneyuki_s](https://twitter.com/saneyuki_s) for the suggestion.

## Update - Sep 27

A lot of this is very early rough drafts of how you might use Rx for your data modeling. A lot of the time, if you need a stream with a initial value that you use but then gets updated (like window size, etc.), you might/will probably be better off using the `.startWith` or `.just` operators instead of Subjects. Regardless, I hope you find this a little bit useful to thinking about new ways to do things ;)

## External links

* Repo: https://github.com/justinwoo/react-rxjs-flow
    * Multiple models: https://github.com/justinwoo/react-rxjs-flow/tree/multiple-models-intents
    * Transform: https://github.com/justinwoo/react-rxjs-flow/tree/subject-transform
    * Separate Subjects per intent: https://github.com/justinwoo/react-rxjs-flow/tree/each-intent-subject
* RxJS: https://github.com/Reactive-Extensions/RxJS
* The staltz intro to Reactive Programming: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754
* Another crappy article I wrote on RxJS usage: http://qiita.com/kimagure/items/88d2cd9c3d7aeaa1ac73
* Rx Marbles, because pictures are worth a lot of words: http://rxmarbles.com/