title: Redux 为什么不要多个 stores？
date: 2016-03-14 16:55:32
tags:
---

实话说，我不认为一个 store 是好事情。

实话说，我不认为多个 store 会出什么问题。当然也遇到过问题：

- 如果一个 store 保存 当前考核周期
- 另一个 store 会根据当前考核周期获取考核群体列表

这样 store 之间拿数据很麻烦。

如果是一个 store 就很方便。其实没有遇到过 flux 中的 `waitFor` 的场景。即使遇到，redux 也无法解决。它本身也是按照顺序执行 reducer 更新 state 的。

最大的问题就是，拿数据很麻烦。毕竟逻辑上分开了。有时又要它们在一起使用。这块没有 common pattern。

[stackoverflow - Redux - multiple stores, why not?](http://stackoverflow.com/questions/33619775/redux-multiple-stores-why-not)

## Question

As a note: I've read the docs for Redux (Baobab, too), and I've done a fair share of Googling & testing.

Why is it so strongly suggested that a Redux app have only one store?

I understand the pros/cons of a single-store setup vs a multiple store setup (There are many Q&A on SO on this subject).

IMO, this architectural decision belongs to the app developers **based on their projects' needs**. So why is it so strongly suggested for Redux, almost to the point of sounding mandatory (though nothing is stopping us from making multiple stores)?

EDIT: feedback after converting to single-store

**After a few months working with redux on what many would consider a complex SPA, I can say that the single store structure has been a pure delight to work with.** 

作者反水了。看看原因。

A few points that might help others understand why single store vs many store is a moot question in many, many use-cases:

- it's reliable: we use selectors to dig through the app state and obtain context-relevant information. We know that all the needed data is in a single store. It avoids all questioning as to where state issues could be.

还好。实变形不变。

- it's fast: our store currently has close to 100 reducers, if not more. Even at that count, only a handful of reducers process data on any given dispatch, the others just return the previous state. The argument that a huge/complex store (nbr of reducers) is slow is pretty much moot. At least we've not seen any performance issues coming from there.

store 会慢吗？逻辑多，或者不必要的逻辑多才会慢。

- debugging friendly: while this is a most convincing argument to use redux as a whole, it also goes for single store vs multiple store. When building an app you're bound to have state errors in the process (programmer mistakes), it's normal. The PITA is when those errors take hours to debug. Thanks to the single store (and redux-logger) we've never spent more than a few minutes on any given state issue.
a few pointers

The true challenge in building your redux store is when deciding how to structure it. Firstly, because changing structure down the road is just a major pain. Secondly, because it largely determines how you'll be using, and querying your app data for any process. There are many suggestions on how to structure a store. In our case we found the following to be ideal:

```js
{
  apis: {     // data from various services
    api1: {},
    api2: {},
    ...
  }, 
  components: {} // UI state data for each widget, component, you name it 
  session: {} // session-specific information
}
```

Hopefully this feedback will help others.


---

## Answer

There are edge cases when you might use multiple stores (e.g. if you have performance problems with updating lists of thousands of items that are on screen at the same time many times per second). That said it's an exception, and in most apps you never need more than a single store.

作者说，只有考虑到性能，才会用到多个 stores。

Why do we stress this in the docs? Because most people coming from Flux will assume multiple stores is the solution to making update code modular. However Redux has a different solution for this: reducer composition.

所以嘛？就是实变形不变（逻辑上一致的）。

Having multiple reducers that are further split into a reducer tree is how you keep updates modular in Redux. If you don't recognize this, and go for multiple stores without fully understanding reducer composition first, you will miss many benefits of Redux single store architecture:

Using reducer composition makes it easy to implement "dependent updates" a la waitFor in Flux by writing a reducer manually calling other reducers with additional information and in a specific order.

reducer 是通过 function 来组合。而且 waitFor 也可以通过顺序调用 reducer 实现。

当然，Rx 的 stream 也可以组合啊。

With a single store, it's very easy to persist, hydrate, and read the state. Server rendering and data prefetching is trivial because there is just one data storage that needs to be filled and rehydrated on the client, and JSON can describe its contents without worrying about store's ID or name.

在 store -> view 之间给一个点，介入这个点就可以。比如直接一个 Store.subject.onNext(..., initalState)。

A single store makes Redux DevTools time travel features possible. It also makes community extensions like redux-undo or redux-optimist easy because they operate on the reducer level. Such "reducer enhancers" can't be written for stores.

但是，我对 DE 没啥兴趣。time travel 到可以做。

Single store guarantees that the subscriptions are called only after the dispatch has been processed. That is, by the time listeners are notified, the state has been fully updated. With many stores, there is no such guarantees. This is one of the reasons Flux needs the waitFor crutch. With a single store, this is not a problem you see in the first place.

这个的确是，只监听一个 store，所有的 reducer 都是完成了，才告诉 view 可以更新了。

Above all, multiple stores are unnecessary in Redux (except for performance edge cases which you are supposed to profile first anyway). We make it an important point in the docs so you are encouraged to learn reducer composition and other Redux patterns instead of using Redux as if it was Flux, and losing its benefits.
