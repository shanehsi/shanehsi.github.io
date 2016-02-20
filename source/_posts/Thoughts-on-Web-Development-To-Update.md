title: Web 开发的思考（不间断更新）
date: 2016-01-29 17:02:05
tags:
---

## 什么是 UI

UI 层抽象成最终的 HTML（DOM 结构，树状层级结构）和样式（CSS）。这块可以再次回顾下《WebKit技术内幕》中的 render 部分，回顾的原因是拟合出适当的布局方式。

UI 上会绑定 Event。抽象成两种：

- Lifecycle Event
- Input Event (keyboard, mouse, touch etc.)

## 如果要实现可视化编辑？

组件是自管理的。所以，先假设组件是 atom。暴露出来 `component_id`。

如何组织（或者叫做布局）组件呢？

#### Layout

参考下 [lightningdesignsystem](https://www.lightningdesignsystem.com/design/layout)。

记住这些名词：

- visual grids 栅格系统，常用的布局方式
- spacing 间距（留白），在栅格系统中也有体现，gutter。
- sections 分区块。是通过栅格系统作为标尺划分区块。

> 栅格设计系统（又称网格设计系统、标准尺寸系统、程序版面设计、瑞士平面设计风格、国际主义平面设计风格）。
> 瑞士平面设计风格：通过瑞士平面设计杂志的宣传
> 国际主义平面设计风格：由于这种风格简单明确，传达功能准确，因而很快得到世界范围内的普遍认可，成为战后影响最大的一种平面设计风格，也是国际最流行的风格。

所以，对于 Layout，可以提供的是设计工具，最原始的是基于栅格系统，然后再在上面设计绘制区块。

当然，这么灵活，要做一些 template。

如何选择（或者设计）template？

- Know your use case. Understand how the information on the page will be used. 了解信息在页面上如何被使用

- Prioritize your content. Organize your content to highlight the most important information. 突出最重要的信息。参考 [网页设计中的F式布局](http://www.uisdc.com/understanding-the-f-layout-in-web-design)

- Group related content together. Make it efficient for users to work with the content. 内容之间的合作，让用户更高效使用。

我们看下 Lighting 提供的几种 template：

##### Record Layouts

Record layouts consist of a page header, a main content area and a sidebar. The content that should appear in each of these areas depends on the primary use case you are solving for.

记录布局：包含 

- page header - 显示区域很大（横跨 viewport），固定，scroll 时可以 collapse，增加垂直高度。
- main content area - 2/3
- sidebar - 最小宽度 400px。如果 reference layout 在 Master/Detail，collapse 成 main content area 的一个 tab。

{% asset_img record_layout.png Record Layout %}

更多解释：[Laying out a Record Home Page and Using Advanced Components](https://developer.salesforce.com/trailhead/en/lightning_design_system/lightning-design-system6)

##### Workspace Layouts

A workspace layout facilitates user collaboration on records. It highlights the activity and discussion that is happening around a record by placing this information prominently in the larger content area, while simultaneously displaying the related records in the sidebar. A summary of the record’s details are in a panel above the content area for easy reference.

{% asset_img workspace_layout.svg Workspace Layout %}

方便用户在 records 上的协作。

- page header 继续保持
- main content area（上部，较小）当前 record 一些详细信息（元数据）
- main content area（下部，较大）当前发生的 record（activity 或者 discussion）
- sidebar 相关的 records

##### Reference Layouts

A reference layout is optimized for when users are primarily jumping to related records. It highlights the related records by displaying this information in the larger content area. Collaborative items are placed in the smaller sidebar. A summary of the record’s details are in a panel above the content area for easy reference.

##### List Layouts

A list layout consists of a simple page header and body that allows users to switch between predefined lists of items. Common controls include sorting, filtering, charting, and actions for the item type. Users can also switch between list layouts using the “Display” menu.

Choose the types of list layout that best supports your use case:

- Table — Best for managing large sets of data and comparing values
- Board — Use to monitor a workflow or milestones where users can drag cards between stages to indicate progress
- Master-Detail — Allows users to see and edit the details of an item on one screen

示例：

{% asset_img record_layout_sample.png Record Layout Sample %}

再有几个示例：

board

{% asset_img board_sample.png Board Sample %}

##### 再谈 Layout

所以，最广度的 Layout 是固定几个 section：

比如：navbar，sidebar(left)，然后在某些 layout 里有 page header。

然后基本就是双栏布局。sidebar(right)，基本上就是 list。大概上如此。

必须做模板。从业务中提取。

但是，有些组件的类型，是特别适用于 Layout 的某些 section 的，有些是通用的，特别是常用的表单组件。

每种抽象出来的 Layout Template，需要发散下，是否可以有相应的 tablet，mobile 形态。猜测，由于 mobile 的 viewport 很小，很可能要做一些向上滑入的隐藏 section。而在 PC 中，对应的就是 steps，和 Modal 这些。

##### Layout 的可视化设计

假设做出来了这样一个 Layout Builder。

其实无法对 section 做更多的自定义的编辑（类似于 Xcode 的 Auto layout 这些）。因为我要做出三套。相信，这些 Layout 的布局加上，section 间的协作都是有最佳方式的。做些，Layout 的可视化设计，为选择预定义好的 template。Template 还可以对内容的布局做一些规则的限定。

其实，也就是把 Layout template 当做组件了。

- Page
- Layout template
- Section - navbar, left sidebar, page header, main content, right sidebar, 其中有些 section 是在 scroll area 中。对于，scroll，要寻找下浏览器原生实现。
- Section template - 比如 navbar，left sidebar 就有基本固定的布局方式。当然，如果让 navbar 再细分，也有可能有多中 navbar template。
- 另外，每种 Section 是对应有多种 Components 可选，但不是所有。

#### 组件

Layout 虽然可以看成组件，但是又不完全是。

Layout 仅仅是布局。也就是组件的组织方式。它没有任何其他的展示特性。

组件要负责的就多了。

- 组件要等撑满布局
- 组件要负责内部结构和展示
- 组件是有生命周期的
- 组件要有动态数据
- 组件负责获取（或者响应）动态数据
- 组件可以绑定事件

注意，组件中特别的一种责任是和数据打交道。数据又分为两块：

- 数据展示在哪里？
- 数据如何获取？

展示在哪里？使用占位符，比如 `props.name` 这种。

如何获取？组件所需的所有占位符，最终是一种数据结构（比如 JSON 这种 tree）。所以，如何获取还没有到获取的细节，而是对外宣称，我要获取这样子的。至于细节部分，可以和组件分离。

分离的方式，就是让组件响应数据。

其实当然也有可能用数据完全映射到组件。主要是两类数据：

- 应用数据
- 展示数据

先从读开始理解：

应用数据用 atom store 很好理解（其实不是 atom store 啦，而是组件作为 pure function，不需要维护任何状态，不过，维护状态的好处是什么？说实话，撇开性能不谈，没有看到什么好处。）

展示数据呢？似乎是应该放在 component state 里。实际上，放在 component 里的原因无非是，它的 scope 可能仅仅限于 component。事实上呢？不一定。因为，放在 component 里，代表了没有 initial state 这种概念了，因为一刷新，没有及时 preserve 的 state （到 persistent layer，也即是 atom store） 都会丢失。当然可以在 state 里放着，然后在 will unmount 前，preserve 起来。

从写的角度理解：

应用数据如何会写呢？我目前能想到的就是响应事件。然后（从最终结果上看）修改两块地方：1. 持久化到 atom store（然后持久化到服务端）；2. 反馈显示到 component，也就是界面上。来看看数据流：

1. mutation -> atom store
2. mutation -> component

再回顾读里面的数据流：

- atom store -> component

所以，合并下三者，就成单向数据流了（响应式是 undirectional data flow 的形成条件）。

这样组件通信也解决了，因为 component 之间根本不会（也不需要）知道对方，只需要知道数据，数据和 component 又是分离的（component 只有数据的结构）。

回过头来在思考应用数据和展示数据，就可以弄出两个 data stream 了，然后 combine 起来。毕竟数据的责任是不同的。如果 UI 有优先级的需求（先响应展示还是先应用数据），逻辑上分离开是充分条件。

侧边证明了 Reactive 的数据结构还是很有利的。后面找个时间研究下其他异步方式。因为我感觉很多异步解决方案是用同步方法拟合。

```
Layout
 |
\|/
Component
```

之间的 interface 是，Layout 需要知道 component_id，次序（类似于 document flow 的次序），flow 的方式（layout 的 section 已经预定好了，所以就是简单的从小到大的顺序）。

```
Component
 |
\|/
Data
```

之间的 interface 是：数据结构，以及 mutation 会改变的数据结构这些。

## Virtaul DOM

```
Virtual DOM
 |
\|/
DOM
```

也要找一个接口，目的就是渲染（和更新） DOM。至于细节，可以自由切换：

- Virtual DOM
- Ember Glimmer
- Incremental DOM

## 暂停然后整理下

#### 关于标准

司徒正美最近在写关于 HTML 标准的博客，[表单元素 开篇](http://www.cnblogs.com/rubylouvre/p/5121459.html)。非常赞同。HTML 标准，CSS 标准，这里的标准有两层含义：

- 是浏览器内核级别的实现
- 实现了常用的对外接口（这里的接口包括在 screen 的绘制，和接收的外界 event，比如 touch，mouse，keyboard 等），特别是 event 处理很麻烦，细节太多。

所以，标准一是性能好，二是功能全。而且是很高质量的实现，没必要对标准重新实现，而是利用标准提供的接口来扩展功能。

不要在封装上再做封装，但是也要找到停下来的地方。我认为这个地方就是标准。

#### 找到领域模型，定义它们的边界

- 结构
- 展示
- （动画）
- 数据
    - 读数据
    - 写数据


数据是完全可以后面再考虑。

先考虑结构，展示。甚至假设啊，没有 css。也要能合理地显示。

#### CSS 的重用

##### 什么是主题？

主题：

- 最简单是色系的变化
- 可能有一些圆角，直角
- 字体

再复杂一点，我觉得就没有重用的必要了。不如从结构+样式单独写一套。（看似没有重用，但是数据层肯定重用了。结构+展示很薄，代码量不大。）

##### 如何写样式？

布局是确定的。要么用 lost，只要改下 值 就可以生成两套，一套基于浮动，一套基于 flexbox。不过，使用这种 DSL 最大的问题就是，是否对标准破坏太多，或者封装太多。

现在可以直接使用 flexbox。

然后布局里面就用没有样式的元素。

然后，如果要加样式，认真思考。慢慢总结规律。现在还没有总结出规律。

结构和样式的接口就是 className。总结规律，让暴露的 DOM 结构越少越好。

#### DOM

jQuery 这种好东西（封装恶心细节的东西），一定要用的。即使接口不好，可以封装啊。

React 有两块，一块是接口设计很好。至于实现什么的，可以用多种。



