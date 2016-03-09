title: React Component 规格
date: 2016-03-09 11:48:20
tags:
---

- × 组件小而美，互相独立，不要搞大而全的 ui 框架，但是需要统一的架构（css，demo，test 等）
- × 拥抱开源，多利用开源组件，在其基础上做一些本地化，二次开发，最好可以提 pr 反向回馈开源组件
- √ 开源组件实在无法满足需求的情况，需要我们自己开发组件
- √ 样式提供 css 文件，不同业务可以有不同的引入方式
- √ 组件的输出需要是完整的编译后的代码（可以考虑只提供 CommonJS 风格），而不是编译前的 es6 代码，不应该把编译过程交给使用方

**样式提供 css 文件，不同业务可以有不同的引入方式**：

这也就是为什么样式要和 HTML 分离的原因了。目前蚂蚁用的是 namespace。

引申下来，其实就是给个基础的样式，完全放开权限可以微调（如果想重写也是没问题的）。反正就是 HTML 结构是固定了，除了这个限制，爱怎么写 CSS 完全没问题。（所以，HTML 结构设计要合理，不要浮夸）。

这样感觉，less 很好的（讨厌 sass，嫌他复杂，依赖还有个 C 扩展，每次安装，每次改 npm 都一丢丢问题）。

**拥抱开源，多利用开源组件，在其基础上做一些本地化，二次开发，最好可以提 pr 反向回馈开源组件**：

目前 JavaScript 社区，能把接口定义好，再回馈。目前还是以 Monolithic 为主。

前端如何服务化，做到 micro，需要思考。[Linux 为什么还要坚持使用宏内核？](http://www.zhihu.com/question/20314255)。

**组件小而美，互相独立，不要搞大而全的 ui 框架，但是需要统一的架构（css，demo，test 等）**：

是从 Design（UI）推动 Component ？还是反之？

我目前的想法是，要从 Design 开始。出现业务需求是，也要从 Design Standard 出发进行设计增强。然后组件实现。

组件的实现要轻量。

从功能上划分，

- 如果只是改动了 HTML 结构，放在一个组件
- 如果是样式上，由 CSS 负责。

Design 的增强映射成 DOM 结构的增加（如果是修改，要是微调）。我相信从功能点划分出发，子类不会有 DOM 的巨大差异。只可能是增加。

组件化只是开发方便，从用户出发，还是 Design。

## 更多想法

类似于 ant.design 和 react-component 的关系。

后者提供 HTML DOM 结构，一份基础的样式。

HTML/CSS 之外的功能。这块功能的逻辑性是够强的，可以使用 OO 组合（and 我现在对 function 不求甚解，不甚感冒）。


## css 的讨论

```
我觉得还是有好处的。说一些自己的经历和由此产生的想法。
如果每个组件都有单独的 CSS 文件，那么对于复杂的可能引入很多组件的项目来说，这个问题该怎么解决呢？因为如果是用 React 做项目的话，肯定要把基础的 HTML 结构放到顶层 React Component 里提供复用，如果使用最简单的方式，把组件的 CSS 以 形式写在 里，可能会有几个问题：1. 真正引用我们的组件的 React Component 并不能直接接触到 ，如果需要 inject 的话，就得用 document.head.appendChild(linkElement) 这种比较脏的形式，而且会导致 server rendering 的结果和浏览器端结果 checksum 不一致（会有 warning）。如果引用我们组件的 React Component 自身是会被重复引用的组件，很可能 元素会被重复写入 ；或者当组件的生命周期结束的时候，相应的 又没有从 DOM 被移除。如果不能妥善处理 元素，会有几个潜在问题：1. 里边出现大量重复的 ; 2. 大量重复无意义的请求; 3. CSS 很难合并; 4. CSS 文件自身变得难以管理，组件更新乃至替换组件变成了复杂、有风险的事情。为了避开这些问题，对方可能会选择使用 Reducer 这种集成化的解决方案，然而 Reducer 是在 Browserify 之上的封装（抱歉不是特别了解 Reducer，只看过源码，可能理解有误），选择 Reducer 也就意味着牺牲了自由度，对方需要放弃原来的 Browserify / WebPack 技术栈，也就是放弃大量的开源插件，这是有风险的事情，更何况让对方接受一个不是非常知名的集成解决方案并不是十分容易的事情。原本使用 React 组件只是类似调用一个函数那么简单的事情，为什么对方应该考虑这么多呢？
对于我们之前做项目来说，因为使用 Browerify 打包，CSS 地址必须 hardcode 到想要引用这个 CSS 的地方去，而且，不能使用常量，地址必须以某种非常特殊且明确的形式写在那里（或者写个简单的 transform 可以稍微解决得好一点，至少做 rebase 会简单些），不仅开发体验差、CSS 资源难管理，而且仍然会有前边提到的 checksum 不一致的问题。
之前在聚艺项目里的做法是，使用 Less 做开发，所有组件的样式都写在各自单独的样式文件里，然后统一从 entry.less 中 import 进来，实现 concat。这样就避开了在各种 React Component 里 inject 样式很麻烦的问题，也避开了 inline style 形式可能出现的 CSS 重复打包的问题。但是仍然有问题：各个组件必须有自己的命名空间，对于写 Less 来说还好，但是写组件就会麻烦很多；而且因为各种样式共享全局作用域，要想知道某个样式文件依赖了另外某一个样式文件是很难的事情。这里边有很多潜在的问题。
之前曾经非常看好把 CSS Style 以 JS Object 的形式写在 .jsx 里，但是这种形式开发体验并不好，语法也比原生的 CSS 啰嗦、失去了 CSS Preprocessor 的各种优势，而且因为不能原生支持 :hover 等形式，限制很多。如果依赖 Radium 等第三方库去做转换的话，仍然有问题：1. :hover, :active 等非常常用的伪类要用 JS 模拟，这在性能上会有问题，而且也无法保证模拟效果和原生 CSS 绝对一致；2. 原生 CSS 的很多可供选择的开源工具会废掉，比如个人比较常用的 Autoprefixer 在开发效率上会带来很大帮助，可以拓宽一些 CSS 新属性的兼容范围（比如最典型的 flex-box），而且对于 CSS 的可维护性也有改善（随着浏览器的更新、浏览器市场份额的变化，我们可以随时调整 vendor-prefix 的添加策略），但是如果写成 CSS in JS 的话，因为是一个全新的领域，原来这些很有用的工具就不存在了；3. CSS 全部写在 JS 里，可以想象会有多少重复的样式被 server render 出来。
个人认为比较理想的方式仍然是把 CSS 写在外部文件里，我们不仅可以使用各种 CSS Preprocessor，还可以同时使用 PostCSS 做更多事情。但是前边提到的引入 CSS 文件的问题怎么解决？我们自己开发一套解决方案不是一个很容易让人接受的方式（比如我一直一直都不太能接受 FES，总觉得一旦开始用它的框架，以后整个项目的发展空间就被框在它提供的功能范围里了，这是很大的风险）；其次，CSS 可以提供很好的可复用性，比如说 className，它本来就是一个 Class。我们可以利用一些工具在 build 阶段把 CSS 中的各种 className 替换成有规则的 ID，然后提供一个方法帮助组件在 JS 里调用这些被替换掉的 className；如果 className 样式提供的还不够的话，组件自身可以再在元素的 style 里进行扩展。而基于 PostCSS 的 Autoprefixer 可以让我们的 CSS 拥有更广泛的适用范围，甚至阿里开发的 CSSGrace 可以让我们的 CSS 兼容到 IE6，但只需要写最新的无前缀的 CSS 属性。
那么怎么把 CSS 加载到页面中呢，甚至最好还能实现 server rendering？
比如可以实现这样一个 React Component：
import fs from 'fs';
import React, { PropTypes } from 'react';

class Style extends React.Component {
    render() {
        const css = fs.readFileSync(this.props.cssFile, { encoding: 'utf-8' });

        return (
            <style dangerouslySetInnerHTML={{ __html: css }} />;
        );
    }
}

Style.propTypes = {
    cssFile: PropTypes.string.isRequired,
};

export default Style;
然后在写组件的时候：

import React, { PropTypes } from 'react';
import Style from '@mtfe/react-tools/utils/Style.js';
import classNameUtils from '@mtfe/react-tools/utils/className.js';

const CSS_PATH = '../assets/style.css';
const { classNames, ComponentStyle } = classNameUtils(<Style cssFile={CSS_PATH} />);

class MyAwesomeComponent extends React.Component {
    render() {
        return (
            <section>
                <ComponentStyle />
                <article className={classNames('my-awesome-article')}>
                    <h2 className={classNames('my-awesome-article-title')>Title</h2>
                    <p>blablablabla</p>
                </article>
            </section>            
        );
    }
}

export default MyAwesomeComponent;
最后当别人调用我们的组件的时候：
import React, { PropTypes } from 'react';
import AwesomeComponent from 'my-awesome-component';

class App extends React.Component {
    render() {
        return (
            <html>
            <head>
                <title>Fancy App</title>
            </head>
            <body>
                <header>
                    <h1>My site</h1>
                </header>
                <AwesomeComponent />
            </body>
            </html>
        );
    }
}
无需关心样式，组件和样式就都渲染出来了。CSS 部分可以做预处理和后处理，className 允许禁止转换，保持原来的 className，同时添加命名空间，方便用户自己对组件样式进行二次定制，甚至干脆允许禁止加载组件自己的样式，样式部分完全由用户自己去处理；组件的样式 <ComponentStyle />会在内部统计被加载的次数，将只被渲染一次（对于 server rendering 来说很有好处），而且当组件生命周期结束时，会自动判断是否需要把相应的样式从 DOM 中移除。因为 className 是被替换过的，因此没有命名空间的问题，组件变成了真正的组件，是可以随时调用的资源。如果用户觉得这些配置有些麻烦，可以封装一下组件再用。
当然，实现上面这些，困难在于：CSS 样式必须在组件 build 的时候就打包进去（这其实并不难），而且我们需要把对 CSS 类名解析的结果一并打包进去，提供可定制性。build 后的目录结构可能是这样的：
| ...
| lib
| -- index.js
| -- assets
| ---- css
| ------ style.js
| ------ style.classNames.js
| ---- images
| ------ icon.js
| assets
| -- css
| ---- style.less
| -- img
| ---- icon.png
| ...
只把资源相关的东西 build 进去，其他依赖并没有 build 进去，这和 WebPack / Browserify 打包是不一样的。
因为急着出门，先写到这里……很多地方比较乱，请大家多提问题和意见
```




