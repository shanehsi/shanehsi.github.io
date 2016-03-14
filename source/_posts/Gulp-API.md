title: Gulp API
date: 2016-03-13 21:15:58
tags:
---

我很喜欢 gulp。接口很纯粹，模块化的思路很简洁。

所谓模块化的思路，就是 pipe，公用一套 steamm，vinyl。把文件的处理给分块。

- src
- dest
- task

纯粹一点，js 不要引用 font，css，img。

~~不对，js 必须引用 svg，font，css，img 这些。~~

~~因为可以 inline。~~

前端比较麻烦的就是，各种格式都要用。我觉得还是没有必要 inline 吧。根据标准走。

如果经常变化的，inline（比如验证码图片）。所以，inline 是一种特殊需求的解决方案，不要做通用场景。

那其中一个 task 就来了。

- 字体文件的合并
- 图片的合并，sprite

> 图片优化的其他策略，比如通过css，svg，iconfont 代替图片之类的。不是 task。

这些是单独的开发流程。都是对其他资源的引用。

---

## 附：再谈前端需求

其他资源搞定了。

现在是 html，css 开发。

大的东西，一般不希望变（比如布局）。首先，作为大的东西，语义本来就很少。比如，只是设定边界。如果变，就可能变了30%。

如果是小的东西，可以变，变的话，只变了 1%。比如。小的东西，语义非常多。

大的东西都是精炼好的，职责单一的。才能做通用。

从布局组件来看，React 我觉得不好模拟。

不是我包含你，而是你附着我。

而且，因为是 whole store，不需要特殊的通信逻辑。

数据如果很慢的话，DOM 慢就不怕了。

```html
<Anchor.Head>
    <Elem.Logo>
    </Elem.Logo>
    <Elem.
</Anchor.Head>
<Anchor.Anchor1>
</Anchor.Anchor1>
<Anchor.Anchor2>
</Anchor.Anchor2>
<Anchor.Foot>
</Anchor.Foot>
```

> 看更新：

从左至右，从上至下。

```html
<L>
    <L.Sidebar>
    </L.Sidebar>
    <L.Head>
    </L.Head>
    <L.Workspace>
    </L.Workspace>
</L>
```

```html
<L.Head>
    <L.OmniSearch>
    </L.OmniSearch>
    <L.FunctonArea>
    </L.FunctonArea>
</L.Head>
```

```html
<L.OmniSearch>
    <E.SearchBox>
    </E.SearchBox>
</L.OmniSearch> 
```

```html
<L.FunctonArea>
    <E.LFA.Add />
    <E.Apps />
    <E.LFA.Help />
    <E.Setting />
    <E.Notification />
    <E.LFA.Uesr />
</L.FunctonArea>
```

考虑到命名冲突，加入 namespace。

总之，不要提前抽象。而是合并抽象，类似于一种吸纳进标准库的流程。

Layout Store 来做 Layout Component 间通信。

One Single Store is also true. 因为，其实是 stream 的 merge。

从前端问题出发，做逻辑分类。便于精细处理。

> 细分会变复杂，组织要用标准原语。降低复杂度。

不要考虑性能，考虑性能瓶颈。

更多的考虑可读性。

其实，对 Component 再做下 Layout 和 Element 的逻辑分类。是映射于 CSS 的定位、布局相关属性和样式相关属性。

现在的分类标准是：

**如果要做 Loading 样式（就是线稿中间的默认空白显示），则分成 Layout 和 Element**。

因为没有必要特别细化空白显示的结构。所以，Layout 到很微笑时，就没必要了。代码逻辑可以 monolithic 一点。反正可控好维护。

当然，如果分类了，就纯粹些（指 CSS 属性的运用）。

比如，Layout 的 Empty Placeholder 如何被加载到的 Data 和 View 替换？

挂载点的替换。在 React 里其实就是 Children。这块逻辑看起来不属于 Layout，而是为 Layout 在这个 context 中一个必备功能。可以抽出。

现在还有一个 portal 的还需要想好如何处理？

相对定位，这种两个 box 状态的绑定，是浏览器原生提供的。自己做一套不必。

