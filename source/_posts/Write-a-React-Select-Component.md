title: Write a React Select Component
date: 2016-01-21 10:07:45
tags:
---

# 写在前面

今天写下 Select 组件。

通过写 React Upload 组件，发现其实理解了上传的本质之后，后面只是用 React 的方式包装。所以，组件需要自已研发，这样才能加深理解，做到心中无框架，回归编程的通用。

另外，在写 TypeScript 之后，类型的 mental modal 引入之后，前端开始变得有条理性。我相信设计是一项很重要的步骤，基于缜密的思维甚至依托严密的理论设计出来的事物才能响应变化。

所以，Select 组件也决定自己写。

## 目的

最终的形态分为几类：

- 基本
- 不可改，独立筛选框
- 邮箱提示类
- Typeahead
- 选项组

暂时不做多选。

# 步骤

我先实现下『不可改，独立筛选框』，『基本』类似。

## 结构

第一步还是先写结构， HTML。

```html
<div className={cx( "container")}>
    <div className={cx("select")}>
        <span>Select the country ...</span>
    </div>
    <div className={cx( "drop")}>
        <div className={cx( "search")}>
            <input type="text" autoComplete="off" />
        </div>
        <div className={cx( "option-list")}>
            <div className={cx( "option")}>China</div>
            <div className={cx( "option")}>China Taiwan</div>
        </div>
    </div>
</div>
```

如果不写 css，会发现除了没有边框外，已经达到了最终的效果（展开状态）。

## 样式

简单写点样式。只要稍微设置下 border，padding 和 color，backgroud-color，其他暂时不设置。

```css
.container {
  display: inline-block;
  position: relative;
  min-width: 15rem;
}

.select {
  display: block;
  position: relative;
  padding: 0.4rem 0.8rem;
  border: 1px solid #686868;
}

.select > span {
  color: #686868;
}

.drop {
  display: block;
  position: absolute;
  border: 1px solid #686868;
  width: 100%;
  margin-top: -1px;
  padding: 0.1rem;
}

.search {
  display: block;
  position: relative;
  padding: 0.2rem .4rem;
}

.search > input {
  display: inline-block;
  position: relative;
  width: 100%;
  line-height: 1.5rem;
}

.option-list {
  padding: 0.4rem 0;
}

.option {
  padding: 0.4rem 0.5rem;
}

.option:hover {
  color: white;
  background-color: #0b97c4;
}
```

写样式的时候，需要从设计的角度来想。比如这个组件的 display 一定是 `inline` 或者 `inline-block`。至于里面的结构的样式，其实已经不影响外面了。另外，`drop` 肯定是 `position: absolute`。

## 事件

直接上代码吧。

Select.tsx

```js
import { Option } from './Option';

// 设置 drop 是否显示
const hidden = {
  "display": "none"
};

const show = {
  "display": "block"
};

export class Select extends React.Component<any, any> {

  private clickOutsideHandler: any;

  constructor(props) {
    super(props);
    // 常用的 state
    this.state = {
      selected: {
        display: "Select the country ...",
        value: undefined
      },
      open: false
    };
    this.onClickSelect = this.onClickSelect.bind(this);
    this.onClickOption = this.onClickOption.bind(this);
    this.onClickDocument = this.onClickDocument.bind(this);
    this.onChangeFilter = this.onChangeFilter.bind(this);
  }

  // 组件在 Update 的时候做些事情，这可能是这个组件（复杂的组件）的特殊之处，
  // 1. focus 到 filter 框
  // 2. 注册 clickOutsideHandler，并注意在 open=false 时回收
  componentDidUpdate() {
    if (this.state.open) {
      const filterNode: any = findDOMNode(this.refs["filter"]);
      filterNode.focus();

      if (!this.clickOutsideHandler) {
        this.clickOutsideHandler = addReactDomEventListener(document, "mousedown", this.onClickDocument);
      }
      return;
    }
    if (this.clickOutsideHandler) {
      this.clickOutsideHandler.remove();
      this.clickOutsideHandler = null;
    }
  }

  private onClickSelect() {
    this.setState({
      open: !this.state.open
    });
  }

  private onClickOption({ value, display }) {
    this.setState({
      selected: {
        display: display,
        value: value
      },
      filter: undefined,
      open: false
    });
  }

  // 关闭 drop
  private onClickDocument(event: Event) {
    const target: EventTarget = event.target;
    const root = findDOMNode(this);
    const dropNode = findDOMNode(this.refs["drop"]);
    // 如果不包含
    if (!containsNode(root, target) && !containsNode(dropNode, target)) {
      this.setState({open: false});
    }
  }

  private onChangeFilter(event) {
    const value = event.target.value;
    this.setState({
      filter: value
    });
  }

  render() {
    let optionListV;

    // 筛选功能
    const filteredOptionList = _.filter(_.values(this.props.options), (o: any) => {
      if (_.isEmpty(this.state.filter)) return true;
      return new RegExp(this.state.filter, 'i').test(o.desc);
    });

    if (_.isEmpty(filteredOptionList)) {
        // Option 设置了一个 disabled 的状态
      optionListV = <Option disabled
                            display='无匹配项'/>;
    } else {
      optionListV = _.map(filteredOptionList, (item: any) => {
        return (<Option key={item.value}
                        value={item.value}
                        display={item.display}
                        desc={item.desc}
                        onClick={this.onClickOption}/>);
      });
    }

    return (
      <div className={cx("container")}>
        <div className={cx("select")} onClick={this.onClickSelect}>
          <span>{this.state.selected.display}</span>
        </div>
        <div ref="drop" className={cx("drop")} style={this.state.open? show : hidden}>
          <div className={cx("search")}>
            <input ref="filter"
                   type="text" autoComplete="off"
                   value={this.state.filter}
                   onChange={this.onChangeFilter}/>
          </div>
          <div className={cx("option-list")}>
            {optionListV}
          </div>
        </div>
      </div>
    );
  }

}
```

Option.tsx

```js
/// <reference path="../../../typings/ttsd.d.ts" />
import * as React from "react";
import { findDOMNode } from "react-dom";
const styles = require("./Option.css");
import classNames from "classnames/bind";
const cx = classNames.bind(styles);

export class Option extends React.Component<any, {}> {
  constructor(props) {
    super(props);
    this.onClick = this.onClick.bind(this);
  }

  private onClick() {
    this.props.onClick({
      value: this.props.value,
      display: this.props.display
    });
  }

  render() {
    if (this.props.disabled) {
      return (
        <div className={cx("disable")}>{this.props.display}</div>
      );
    }
    return (
      <div className={cx("option")} onClick={this.onClick}>{this.props.display}</div>
    );
  }
}
```

# 思考

在写的过程中，发现很多 Select 其实可以更细化的拆分，比如 Option 可以泛化成 menu。这是其二。

另外写组件的过程其实包含了很多事情，包括：

- 结构
- 样式
- 事件
- 值的处理
- 甚至包括从总体来看组件的设计

Select 的确有不同的形态，我发现很多组件尝试用 if-else，在同一个 `<Select>` 里做到统一。我认为这样是不好的。感性的原因是这是一种很弱的抽象，圈复杂度非常高，不利于代码维护。还有一个原因，也是我这两天自己研发组件得到的经验：开始不要想很多，一个场景对应一套实现；之后再考虑提取公用代码。

这才是 **设计之道**。

所以，暂时不考虑细节的部分，我认为 Select 组件已经开发完成了。代码都在那儿，如何重构是之后的事情。这种时候似乎应该 test-driven。考虑到 test-driven DSL 的恶心程度，在不是复合场景下（复合比如：样式+事件+数据流）不如手动测试。

明天开发 trigger，popup 和 table。

即使这些开发完毕了，也不要急着重构，而是上生产环境。慢慢抽出公用组件。刚开始都是丑陋的代码，慢慢变得合理高效。一下次输入很多知识点会非常低效。
