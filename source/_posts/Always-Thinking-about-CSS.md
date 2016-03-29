title: 一直念着想着 CSS
date: 2016-03-27 19:33:46
tags:
---

并不是不喜欢 CSS。而是找不到 CSS 的学习好方法。这东西就和学数学一样（原谅我用数学比较，因为我一直后悔没有多练习数学题），要大量的实践，总结规律，然后新的场景用总结的规律尝试去套，不能套的要去拓展和完善规律。

> 哪一天我也能学好数学？O(∩_∩)O

反正我不喜欢熟记一堆堆的理论然后找最佳实践，事实证明这样做真的好难好难。/(ㄒoㄒ)/~~

目前终于有了一个好的机会，在好的时间：

- 做一个不是很复杂的 web app。
- 我脑海里已经有了一堆一堆的理论，什么 position relative/absolute，display inline-block/block。盒模型内部的目前觉得还好。

## 所有的总结

### 总结 2016年03月27日19:50:38

关键还是要总结下原语。CSS 和 HTML 分开还是就使用 HTML（JSX）的名字做布局？分开的话，都是用 HTML 的话，就是有两类 Component。一类 layout。关键是这样写 Layout 可不可以重用。能重用的话，就完美了。

### 总结 2016年03月28日10:07:53

## 2016年03月28日10:07:53

### 碎碎念

margin 是可以重叠的，使用这个有风险。使用 margin 主要是为了隔开一些距离。其实只要一方隔开就行，可以选择 main part 设置 margin。

margin 和 padding 都可以隔开，建议在隔开这块，用 margin 多一点。画出的线条很漂亮（当然是因为 margin 隔着 border，而 padding 隔着的是看不见的字体周围的线条，从设计上也是 border 更有效）。

{% asset_img 1.png %}

### 总结

## 2016年03月27日19:50:38

### 想到的

现在第一个要解决的是大的布局问题。这个用 position 可以很好的解决。

absolute 做大的布局的摆布。

relative 其实是做定位的时候，借用下其他元素的容量。这个看起来好难好难。但用的时候真的必须要。比如下面说到的同一行布局。

但是其实 relative 更多的是做些一些位移。要深刻理解 relative，应该比较下 relative/static。

参考：[Difference between static and relative positioning][]。

**Statically positioned elements don't obey `left`, `top`, `right` and `bottom` rules:**。就酱，static 不会相对于自己的 normal position 位移。

如果有这样的需求就是 relative。

然而 absolute 都不用考虑 sibling，只要考虑 parent（为了方便定位，找一个好的 parent，^_^）。

同行摆布使用 `inline-block`，加上 `vertial-align: top`。参考：[Why is this inline-block element pushed downward?][]

`inline-block` 是来替换 float（部分场景下，即放在同一行，但是如果真有一部分按左对齐，一部分按右对齐，还是得要 float），以及模拟 `table-cell` 这个（或者也可能是 ul li，突然发现，语义化元素的自带样式会和 css 有部分功能的重叠）。

参考：[拜拜了,浮动布局-基于display:inline-block的列表布局][]。这里重点说下 `line-height` 方法。

我还要解决一个问题，**垂直居中问题**。

一般是设置，`height` 和 `line-height`（这个是 wrapper） 相同，而且 `line-height` 要大于 `font-size`（一般也是这样）。**而且只适用于单行文字**。

> By default equal space will be given above and below the text and so the text will sit in the vertical center.

其他几种方法是：

`Vertical-Align` 是最简单的方法。但是它只适用于 `table-cell` 和一些 `inline`（估计 `inline-block` 不行）。也就是不适用于 block 元素。

网上关于垂直居中的文章特别多：比如这篇 [6 Methods For Vertical Centering With CSS][]

我觉得其他的方法就研究 [lost] 吧（复杂的场景）。

[Difference between static and relative positioning]:http://stackoverflow.com/questions/5011211/difference-between-static-and-relative-positioning
[Why is this inline-block element pushed downward?]:http://stackoverflow.com/questions/9273016/why-is-this-inline-block-element-pushed-downward
[拜拜了,浮动布局-基于display:inline-block的列表布局]:http://www.zhangxinxu.com/wordpress/2010/11/%E6%8B%9C%E6%8B%9C%E4%BA%86%E6%B5%AE%E5%8A%A8%E5%B8%83%E5%B1%80-%E5%9F%BA%E4%BA%8Edisplayinline-block%E7%9A%84%E5%88%97%E8%A1%A8%E5%B8%83%E5%B1%80/
[6 Methods For Vertical Centering With CSS]:http://vanseodesign.com/css/vertical-centering/

### 总结

关键还是要总结下原语。CSS 和 HTML 分开还是就使用 HTML（JSX）的名字做布局？分开的话，都是用 HTML 的话，就是有两类 Component。一类 layout。关键是这样写 Layout 可不可以重用。能重用的话，就完美了。
