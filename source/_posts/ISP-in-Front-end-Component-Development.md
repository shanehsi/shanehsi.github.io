title: ISP 在前端开发中
date: 2016-02-02 10:08:19
tags:
---

## 理论准备

ISP 即 Interface Segregation Principle。

多个客户端特定的接口好于一个通用接口

> many client-specific interfaces are better than one general-purpose interface.

客户端不应该依赖不需要的方法

> The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use.

这种缩水的接口有叫做『角色接口』。

> ISP splits interfaces which are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called role interfaces

参考术语：[GRASP (object-oriented design)](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design))，General Responsibility Assignment Software Patterns (or Principles)。

我记得 Erik Meijer 在 youtube 上有个讲座介绍了接口分离原则的数学证明。回头翻看下。

参考链接：[Software craftsmanship](https://en.wikipedia.org/wiki/Software_craftsmanship)。

什么是接口？

在计算机系统中，接口是两个组件交换信息的共享边界。这种信息的交换可能是双向的（send/receive data) 也可能是单向的。

> an interface is a shared boundary across which two separate components of a computer system exchange information. 

[Role Interface](http://martinfowler.com/bliki/RoleInterface.html) 的反面是 Header Interface（即一个 Interface，就是把 public methods 都写出 interface）。

使用 Role Interface，就是，你要了解 exchange 的具体的 information。而 Header Interface 认为 supplier 不需关心谁用你的 service，或怎么用。其实就是控制权的问题。Role 控制权在 consumer，Header 控制权在 supplier。长远来看，关注 consumer 是友好的。

## 场景

前端开发（React.js）如何利用 interface？

按照刚才的解释，是要找到其中的 supplier 和 consumer，然后定义它们交换的信息。

这里的 supplier 就是编写的组件。consumer 就是高阶组件。它们之间需要交换的信息就是 props。

其实更担心的是，如果功能要增加，如何扩展？一种原则是，不要去扩展，而是重新复制一份。然后看情况是否需要提取公用的逻辑。不要把公用的边界提前设定。

目前的 props 看来更多的还是数据。可能会有部分展示相关的属性（初始值）。


