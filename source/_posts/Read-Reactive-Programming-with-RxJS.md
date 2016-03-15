title: Reactive Programming with RxJS - 笔记
date: 2016-03-14 14:28:04
tags:
---

Untangle Your Asynchronous JavaScript Code

> events happen in random order 事件不同顺序，就要管理事件
> applications crash 就要重启
> networks fails 就要重试
> Few applications are completely synchronous 现状
>  need super-fast responses and  异步，快速响应。
> the ability to process data from different sources at the same time without missing a beat. 同时处理多个源的数据的能力，不丢失。
> code becomes expo- nentially more complex as we add concurrency and application state. 引入并发和应用状态，程序变非常复杂。

## Ch1 - The Reactive Way

多个源头

```js
var button = document.getElementById('retrieveDataBtn');

var source1 = Rx.DOM.getJSON('/resource1').pluck('name');
var source2 = Rx.DOM.getJSON('/resource2').pluck('props', 'name');

function getResults(amount) {
    return source1.merge(source2)
        .pluck('names')
        .flatMap(function(array) { return Rx.Observable.from(array); }
            ).distinct()
        .take(amount);
}

var clicks = Rx.Observable.fromEvent(button, 'click');

clicks.debounce(1000)
    .flatMap(getResults(5))
    .subscribe(
        function(value) { console.log('Received value', value); },
        function(err) { console.error(err); },
        function() { console.log('All values retrieved!'); }
    );
```

重点：
- pluck 来做摘取
- merge 来合并
- flatMap 来展平
- distinct 进行去重
- take 取样
- debounce 防抖
- subscribe 订阅

An Observable represents a stream of data.

再强调：Observable，可被观察到的事物。

这里面：

- 两个 remote source 是 Observable
- mouse clicks 是 Observable

> 后面会看到，`Subject` 可以做 proxy。方便构建架构。

`Rx.Observable.create`

## Ch 2 - Deep in the Sequence

- map
- filter
- reduce
- flatMap 表达式这样： O[O1, O2, O3]

{% asset_img flatMap.png FlatMap %}

| from | Synchronous | 从 arrays 和 iterables |
| range | Sync | |
| interval | Async | 时间间隔生成值 |
| distinct | Same as | | 

### RxJS’s Subject Class

A Subject is a type that implements both Observer and Observable types. As an Observer, it can subscribe to Observables, and as an Observable it can produce values and have Observers subscribe to it.

In some scenarios a single Subject can do the work of a combination of Observers and Observables. For example, for making a proxy object between a data source and the Subject’s listeners, we could use this:

```js
var subject = new Rx.Subject();

var source = Rx.Observable.interval(300)
    .map(function(v) {
        return 'Interval message #' + v; }).take(5);

source.subscribe(subject);

var subscription = subject.subscribe(
    function onNext(x) { console.log('onNext: ' + x); },
    function onError(e) { console.log('onError: ' + e.message); },
    function onCompleted() { console.log('onCompleted'); }
);

subject.onNext('Our message #1');

subject.onNext('Our message #2');

setTimeout(function() {
    subject.onCompleted();
}, 1000);

```

In the preceding example we create a **new Subject** and a **source Observable** that emits an integer every 300 millisecond. Then we **subscribe** the **Subject** to the **Observable**. After that, we **subscribe** an **Observer** to the **Subject** itself. The Subject now behaves as an Observable.

Next we make the Subject **emit values of its own** (message1 and message2). In the final result, we get the Subject’s own messages and then the proxied values from the source Observable. The values from the Observable come later because they are **asynchronous**, whereas we made the Subject’s own values immediate. Notice that even if we tell the source Observable to take the first five values, the output shows only the first three. That’s because after one second we call **onCompleted** on the Subject. This **finishes** the **notifications** to all subscriptions and overrides the take operator in this case.

### AsyncSubject

AsyncSubject emits the **last value** of a sequence only if the sequence completes. This value is then **cached forever**, and any Observer that subscribes after the value has been emitted will receive it right away. AsyncSubject is convenient for asynchronous operations that **return a single value**, such as Ajax requests.

```js
var delayedRange = Rx.Observable.range(0, 5).delay(1000);
var subject = new Rx.AsyncSubject();
delayedRange.subscribe(subject);
subject.subscribe(
    function onNext(item) { console.log('Value:', item); },
    function onError(err) { console.log('Error:', err); },
    function onCompleted() { console.log('Completed.'); }
);

// Value: 4 
// Completed.
```

### BehaviorSubject

会有一个 placehodler，`var subject = new Rx.BehaviorSubject('Waiting for content');`。

会等待 remote source 给出一个值，然后就用 remote source 代替。

```js
subject.subscribe(function(result) {
        document.body.textContent = result.response || result;
    },
    function(err) {
        document.body.textContent = 'There was an error retrieving content';
    });
Rx.DOM.get('/remote/content').subscribe(subject);
```

类似于，就是，Subject，但是他 subscrib 的是一个值（或者自己 onNext 了一个值）。

### ReplaySubject

A ReplaySubject caches its values and re-emits them to any Observer that sub- scribes late to it. Unlike with AsyncSubject, the sequence doesn’t need to be completed for this to happen.

及时是后面才 subscribe，但是会回放 observables 上所有的值。

{% asset_img ReplaySubject.png ReplaySubject %}


