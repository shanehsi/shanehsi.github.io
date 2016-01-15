title: Write a Webpack Plugin
date: 2016-01-14 17:56:38
tags:
---

Plugin 介入的是 Webpack build process。也就是说，在 Webpack build 时，做一些其他的工作（然后可以拿到 Webpack build 时的一些 context）。

Plugin 需要理解 Webpack 的一些 low-level 的内部机制，然后 hook into。要读一些 source code。

## Compiler and Compilation

开发 plugin 要知晓两个对象 `compiler`，`compilation`。

- `compiler` 相当于 Webpck 的 context，并且是配置过的，也就是说，可以拿到 Webpack 的配置比如，options，loaders，plugins。每一个 plguin 都会获得一个对 `compiler` 的引用。

- `compilation` 对应于一个版本的 assets 的 build。因为在运行 Webpack development middleware 时，每次文件变动被探测到，都会有一个新的 compilation，生成一次新的 compiled assets。所以，`compilation` 代表了 module resources，compiled assets，changed files 和 watched dependencies 的当前状态。还提供了很多 callback points，让 plugin 来做些自定义操作。

## Basic plugin architecture

Plugins 是实例化对象，prototype 上必须有 `apply` 方法，它会在 Webpack 安装这个 plugin 时调用。apply 会得到一个 Webpack compiler 的引用，并被赋权使用 compiler 的 callbacks。


```js
function HelloWorldPlugin(options) {
  // Setup the plugin instance with options...
}

HelloWorldPlugin.prototype.apply = function(compiler) {
  compiler.plugin('done', function() {
    console.log('Hello World!'); 
  });
};

module.exports = HelloWorldPlugin;
```

```js
var HelloWorldPlugin = require('hello-world');

var webpackConfig = {
  // ... config settings here ...
  plugins: [
    new HelloWorldPlugin({options: true})
  ]
};
```

## Accessing the compilation

如何访问 `compilation` 对象呢？通过使用 `compiler` 的回调。


```js
function HelloCompilationPlugin(options) {}

HelloCompilationPlugin.prototype.apply = function(compiler) {

  // 设置回调，访问 compilation
  compiler.plugin("compilation", function(compilation) {

    // 设置回调，访问 compilation 的 steps
    compilation.plugin("optimize", function() {
      console.log("Assets are being optimized.");
    });
  });
});

module.exports = HelloCompilationPlugin;
```

关于 `complier`，`compilation` 更多的信息参看 [plugins API](https://webpack.github.io/docs/plugins.html)。

## Async compilation plugins

异步这块暂时不看。

## PLUGINS 

以上是 high-level 的文档，这里是具体的细节。

Webpack 的很多对象都继承 Tabable class，后者会暴露一个 `plugin` 方法。通过 `plugin` 方法，plugins 会 inject custom build steps。比如：`compiler.plugin` 和 `compilation.plugin`。

具体说，对于 plguin 的调用会绑定在 build process 的特定步骤。

```js
/ MyPlugin.js

function MyPlugin(options) {
  // Configure your plugin with options...
}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin("compile", function(params) {
    // 开始 compile
    console.log("The compiler is starting to compile...");
  });

  compiler.plugin("compilation", function(compilation) {
    // 一个新的 compilation
    console.log("The compiler is starting a new compilation...");

    // optimize 步骤
    compilation.plugin("optimize", function() {
      console.log("The compilation is starting to optimize files...");
    });
  });

  compiler.plugin("emit", function(compilation, callback) {
    // 开始emit 生成的 assets，此时是 plugin 往 c.assets (c: Compilation) 数组增加 assets 的最后机会
    console.log("The compilation is going to emit files...");
    callback();
  });
};

module.exports = MyPlugin;

```


全部的 steps，参见 [THE COMPILER INSTANCE](https://webpack.github.io/docs/plugins.html#the-compiler-instance)。

## Example

该例子很简单，从文件系统拿到一个值，放入 compilation 的 assets。

```js
"use strict";

var fs = require('fs');
var path = require('path');

function HtmlPlugin(options) {
  this.options = options ? options : {};
}

HtmlPlugin.prototype.apply = function(compiler) {
  var that = this;

  compiler.plugin("emit", function(compilation, callback) {
    that.addFileToWebpackAsset(compilation);
    callback();
  });
};

HtmlPlugin.prototype.addFileToWebpackAsset = function(compilation) {
  var that = this;

  var template = path.join(__dirname, 'default-pc.html');
  var filename = path.resolve(template);
  compilation.fileDependencies.push(filename);

  var outFilename = path.basename(that.options.filename || 'index.html');
  compilation.assets[outFilename] = {
    source: function() {
      return fs.readFileSync(filename).toString();
    },
    size: function() {
      return fs.statSync(filename).size;
    }
  };
};

module.exports = HtmlPlugin;

```

