title: JavaScript 正则包含子字符串
date: 2016-01-21 16:07:36
tags:
---

## 场景

一般是在 typeahead 时使用。

```js
new RegExp(<substring>, 'i').test(<string>);
```

其中 `'i'` 代表忽略大小写。

也可以使用 `indexOf`，`<string>.indexOf(<substring>) !== -1`。可以看出不能方便的通过类似 `'i'` 设置忽略大小写。

至于性能，根据浏览器实现不同性能有所差异。参考：[Fastest way to check a string contain another substring in Javascript?](http://stackoverflow.com/questions/5296268/fastest-way-to-check-a-string-contain-another-substring-in-javascript)。

