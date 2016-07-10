title: Make a Simple and Descriptive, a Empathic World!
date: 2016-07-05 15:04:13
tags:
---

> 最近心情也很低落。写程序写着写着还是发现很多不清楚的（比如：如何减少 if，现在组件很多的写法都是根据 props 做简单的 if 判断，是显示 disable 还是 readonly 还是什么）。基本的代码组织也是很乱。刚才抽空看了尤雨溪的 vuejs，API 设计实在是漂亮。没有看代码组织，以及代码细节，相比也不差。这篇文章的标题是：创造一个简洁描述性高的世界，一个同理心的世界。希望写出的代码能够让人快速理解，认同并爱上。

看下 WPF 开发中，为什么使用 MVVM：

- UI 和数据可以分开开发
- 当UI全部改变时，代码可以不改变

UI 与程序通过 Bindings 和 Commands 进行交互。

Bindings 和 Commands 在 ViewModel 中进行设计。

## The Model

Web Services

Rest Services

Generic Collections

反正是数据。

这一层我的想法是：

- GraphQL 主要不是方便前端的，而是方便后端的。
- GraphQL 也方便了前端，但是没必要把 model 直接绑定到 view 上。

但是我如果要做：服务化组件呢？

## The View Model

属性：作用，属性变化，更新 UI

集合

Commands：可以触发的事件。

## The View

这一层可以不写代码，可以可视化编辑。

主要要完成：

- 将 View Model 的属性绑定到 component 上。
- View Model 的集合绑定到 ListBox，DataGrid 上。
- Commands
    - 指出需要实现的 ICommand

# 感想

一定要分层。否则永远实现不了可视化编辑（本来就很难，可能不切实际）。

# 再次爆炸式理解下 MVVM

## UI层的设计模式——从Script、Code Behind到MVC、MVP、MVVM

### Code Behind

很好地处理UI和逻辑各自分开的关系，你可以让UI和逻辑各自做好各自的事情

真正在UI和逻辑分化后带来的实质问题是：

**是按逻辑和UI划分地管理，还是按单界面的事务进行划分地管理**

对了，如果 

React 的 Command，虽然是 direct 的，但是它声明一个 interface，不就是回调了吗？它有 payload（mutation），如何处理就不是它的职责了。

双向绑定数据上好做，但是 Command（@action） 呢？

