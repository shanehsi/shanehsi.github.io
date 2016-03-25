title: Webpack Less Loader
date: 2016-03-24 15:14:15
tags:
---

[webpack/less-loader](https://github.com/webpack/less-loader)

这里面的小细节还蛮多的。

开发时：

主要注意下 style loader 的作用。在 production 是不需要的。

```js
module.exports = {
  module: {
    loaders: [
      {
        test: /\.less$/,
        loader: "style!css!less"
      }
    ]
  }
};
```

关键 less 还有 options，具体参考 [LESS documentation](http://lesscss.org/usage/#command-line-usage-options)。

```js
module.exports = {
  module: {
    loaders: [
      {
        test: /\.less$/,
        loader: "style!css!less?strictMath&noIeCompat"
      }
    ]
  }
};
```

生产环境：

```js
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
    ...
    // must be 'source-map' or 'inline-source-map'
    devtool: 'source-map',
    module: {
        loaders: [
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract(
                    // activate source maps via loader query
                    'css?sourceMap!' +
                    'less?sourceMap'
                )
            }
        ]
    },
    plugins: [
        // extract inline css into separate 'styles.css'
        new ExtractTextPlugin('styles.css')
    ]
};

```