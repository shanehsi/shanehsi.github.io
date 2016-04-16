title: JavaScript existential operator alternative syntax
date: 2016-04-16 16:05:06
tags:
---

[链接](http://stackoverflow.com/questions/15260732/does-typescript-support-the-operator-and-whats-it-called)

妥协的方法：

```js
var thing = foo && foo.bar || null;
var thing = foo && foo.bar && foo.bar.check && foo.bar.check.x || null;
```
