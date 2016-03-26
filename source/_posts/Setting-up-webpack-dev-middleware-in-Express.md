title: Express 设置 webpack-dev-middleware
date: 2016-03-22 09:21:32
tags:
---

需要了解几个概念：

1. Express 和 connect 的区别
2. Express middleware 的原理大概了解

Webpack dev 的几个概念：

**a. webpack dev server 完全可以 self-running 的 dev server，并包含 live reloading。**：

**b. webpack dev middleware，**：

是一个 simple wrapper middleware for webpack。 It serves the files emitted from webpack over a **connect server**.

和 bundling as files 相比的优势：

1. No files are written to disk, it handles the files **in memory**。
2. If files changed **in watch mode**, the middleware no longer serves the old bundle, but **delays requests until the compiling has finished**. You don’t have to wait before refreshing the page after a file modification.

**c. webpack hot middleware**

Webpack hot reloading using only webpack-dev-middleware. This allows you to add hot reloading into an existing server without webpack-dev-server.

以上两种 middleware 的区别可能要看代码。从文档和我的理解来说：

1. dev 是 in memory
2. hot 是依赖了 dev，并增加了 hot reload

因为已经有 Express 作为 server，所以，目的是：

**use webpack-dev-server as a middleware**


