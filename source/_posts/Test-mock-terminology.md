title: Test mock terminology
date: 2016-04-16 16:45:55
tags:
---

[sinon examples](http://sinonjs.org/#examples) 非常的清晰。

找到一篇中文：[置换测试: Mock, Stub 和其他](http://www.tuicool.com/articles/iAz6fi)

- double - 理解为置换，它是所有模拟测试对象的 **统称**，我们也可以称它为替身。

- stub - 测试桩，它能实现当特定的方法被调用时，返回一个指定的模拟值（每次一致的模拟数据）。

以 sinon.js 为例：

```js
    var stub = sinon.stub().returns(42);

```

- spy - 侦查，它负责汇报情况，持续追踪什么方法 **被调用了** ，以及调用过程中 **传递了哪些参数** 。比如一个特定的方法是否被调用或者是否使用正确的参数调用。当你需要测试两个对象间的 某些**协议** 或者关系时会非常有用。

> 关键是方法间的 **协议**。

```js

it("calls the original function", function () {
    var spy = sinon.spy();
    var proxy = once(spy);

    proxy();

    assert(spy.called);
});
```

- mock - 与 spy 类似，有些许不同。 spy 追踪所有的方法调用，并在事后让你写断言，而 mock 通常需要你 **事先设定期望**。你告诉它你期望发生什么，然后执行测试代码并验证最后的结果与事先定义的期望是否一致。

```js

it("returns the return value from the original function", function () {
    var myAPI = { method: function () {} };
    var mock = sinon.mock(myAPI);
    mock.expects("method").once().returns(42);

    var proxy = once(myAPI.method);

    assert.equals(proxy(), 42);
    mock.verify();
});
```

- fake - **具备完整** 功能实现和行为的对象，行为上来说它和这个类型的真实对象上一样，但不同于它所模拟的类，它使测试变得更加容易。一个典型的例子是使用内存中的数据库来生成一个数据持久化对象，而不是去访问一个真正的生产环境的数据库。

对细节有更多，参考 [Mocks Aren’t Stubs](http://martinfowler.com/articles/mocksArentStubs.html)

## 模拟主义者 (Mockists) vs. 统计主义者 (Statists)

总结下，模拟的是行为（内部的），统计的是最终状态。

许多关于模拟对象的讨论主要是衍生自 Fowler 的文章的，它们讨论了两种不同类型的程序员，模拟主义者和统计主义者，所写的测试。

模拟主意的方式是测试对象之间的交互。通过使用模拟对象，你可以更容易地验证被测对象是否遵循了它与其他类已建立的协议，使得在正确的时间发生正确的外部调用。对于那些使用行为驱动 (behavior-driven) 的开发者来说，这种测试可以驱动出更好的生产代码，因为你需要明确模拟出特定的方法，这可以帮你设计出在两个对象之间使用的更优雅的API，这种想法与模拟驱动紧密联系在一起。因此模拟主义的测试更偏向于单元级别的测试，而不是完全的端到端 (end-to-end) 测试。

统计主义的方式是不使用模拟对象。这种思路是测试时只测试状态而不是行为，因此这种类型的测试更加健壮。使用模拟测试时，如果你更新了实际类的行为，模拟类也需要同步更新；如果你忘了这么做，你可能会遇到测试可以通过但是代码却不能正确工作的情况。通过强调在测试环境中只使用那些真正的代码，统计主意的测试可以帮助你 **减少测试代码和实现代码的耦合度，并降低出错率**。这种类型的测试，您可能已经猜到，适合于更全面的端到端的测试。

当然，并不是说有两个对立的程序员学派；你不可能看到模拟主义和统计主义的当街对决。这种分歧是有用的，但是，得认识到 mock 在有些时候是你的工具箱里最好的工具，但是有时候又不是。不同类型的测试适用于不同的任务，并且最高效的测试套件往往是不同测试风格的集合体。仔细考虑你到底想要用单个测试来验证些什么，这能帮助你找到最合适的测试方式，而且能帮你决定对于当前工作来说，使用模拟测试对象是否是正确的工具。