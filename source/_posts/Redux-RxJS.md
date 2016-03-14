title: Redux RxJS
date: 2016-03-14 15:53:21
tags:
---

使用 RxJS 的目的是为了引入 stream 的操作。

- actions 作为 mutations 保留，并且 view 对 actions 是 imperative。
- store 监听 action。

- stream 在进入 store 之前 merge 一次。
- store 到 view，最简单的是通过 throttle，交给 requestAnimationFrame。
- （但是如果有 animation store，可以在加入一层 merge）

