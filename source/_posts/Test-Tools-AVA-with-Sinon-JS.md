title: 测试工具篇： AVA 和 Sinon.JS
date: 2016-03-25 17:50:33
tags:
---

链接：[afterEach and beforeEach not running? #560](https://github.com/sindresorhus/ava/issues/560)

虽然这篇文章是讲这个的（在 [sindresorhus/ava][] 的 README 里）:

[sindresorhus/ava]:https://github.com/sindresorhus/ava

Keep in mind that the `beforeEach` and `afterEach` hooks run just before and after a test is run, and that by default tests run concurrently. If you need to set up global state for each test (like spying on `console.log` [for example][]), you'll need to make sure the tests are [run serially][].

[for example]:https://github.com/sindresorhus/ava/issues/560
[run serially]:https://github.com/sindresorhus/ava#running-tests-serially

如果要设置一些 global state，比如 spying `console.log`，则需要保证 tests 是串行运行的。

或者使用 `context` object 来保存原先的 log。

```js
import test from 'ava';
import sinon from 'sinon';

test.beforeEach(t => {
    t.context.log = console.log;

    console.log = sinon.spy();
});

test.afterEach(t => {
    console.log = t.context.log;
});

test('first', t => {
    console.log('first');
    t.true(console.log.calledOnce);
});

test('second', t => {
    console.log('second');
    t.true(console.log.calledOnce);
});
```



