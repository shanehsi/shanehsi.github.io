title: 读一段 JavaScript 代码段
date: 2016-03-08 15:00:44
tags:
---

## 代码

```js
/* eslint no-var:0, vars-on-top:0 */
if (process.env.NODE_ENV === undefined) {
  process.env.NODE_ENV = 'development';
}

var fs = require('fs');
var path = require('path');
var url = require('url');
var http = require('http');
var crypto = require('crypto');
var cluster = require('cluster');

var httpHash = require('http-hash');
var ecstatic = require('ecstatic');
var pingme = require('pingme');
var log = require('npmlog');
var cookieParser = require('cookie-parser');

// 静态文件 root 目录
var staticRoot = path.join(process.cwd(), 'dist', process.env.NODE_ENV);

var serveStatic = ecstatic({
  root: staticRoot,
  cache: process.env.NODE_ENV === 'development' ? 'no-store' : 3600,
  showDir: false,
});
var serveHTML = ecstatic({
  root: staticRoot,
  cache: 'no-store',
  showDir: false,
});

// 拿到一些常量。包括
var PORT = require('./apps/@beauty/constants/serverPort');
var getEInHost = require('./apps/@beauty/host/getEInHost');
var getEPassportHost = require('./apps/@beauty/host/getEPassportHost');

var APP_PATHS = [
  '/',
  '/app/index',             // 首页
  '/app/home',              // 我的
  '/app/profile',           // 个人信息
  '/app/i',                 // 个人主页
  '/app/i/:beautyId',       // 个人主页
  '/app/feedback',          // 看评价
  '/app/product',           // 作品
  '/app/product/create',    // 作品添加
  '/app/product/edit/:id',  // 作品编辑
  '/app/poi/search',        // 门店选择
  '/app/qrcode',            // 用户晒图
  '/app/message',           // 消息
  '/app/statistics',        // 查人气
  '/app/tell/people',       // 告诉朋友
  '/app/about',             // 关于聚艺
];

var RELATION_PATHS = [
  '/relation',              // 关联
];

// mock server 是 express
var app = require('express')();

function mockMiddleware(req, res, next) {
    // T，外部变量
  var data = mock.get({
    uri: req.originalUrl,
    method: req.method,
  });
  if (data === null) return next();
  res.type('json');
  res.status(data.status);
  res.send(data.body);
}

// 开发环境使用 mock
if (process.env.NODE_ENV === 'development') {
  var Mock = require('monkeyjs');
  // 构造函数参数： 目录为 mock
  var mock = new Mock('mock');

  app
  .use(mockMiddleware)
  // .use(require('connect-slashes')(false));
}

function epassportMiddleware(req, res, next) {
    // 如果是 '/relation'，则验证 epassort
  if (req.path === '/relation') {
    // 如果是开发环境，则 beauty-sid:test
    if (process.env.NODE_ENV === 'development') {
      log.info('epassport', '测试环境 mock bsid');
      res.cookie('beauty-bsid', 'test');
      // 直接 return，T 这个代码的圈复杂度
      return next();
    }
    var bsid = req.cookies['beauty-bsid'];
    if (!bsid) return login();
    var request = require('request');
    var baRequest = require('@mtfe/basic-auth').wrapRequest(request, 'mtsalon', 'xoxoxo');

    // 验证 epassport epassport 的 host e-in.meituan.com
    baRequest('http://' + getEInHost(req.get('host')) + '/bizacct/ssobizacctinfo?bsid=' + bsid, function(error, response, body) {
      if (error) {
        log.error('epassport', error.message);
        // 跳转到登录页面
        return login();
      }
      // 解析访问 epassport 的 body
      var bodyJSON = JSON.parse(body);
      if (bodyJSON.error) {
        log.error('epassport', bodyJSON.error.message);
        return login();
      }
      log.info('epassport', '验证成功');
      next();
    });

  } else if (req.path === '/relation/token') { // 如果是 /relation/token，从 request 的 querystring BSID 中拿到值，给 beauty-sid
    var bsid = req.query.BSID;
    if (bsid != null) {
      res.cookie('beauty-bsid', bsid);
    }
    res.redirect('/relation');  // 再跳转
  } else {
    next();
  }

  function login() {
    res.redirect(url.format({
      protocol: 'http',
      host: getEPassportHost(req.get('host')),
      pathname: '/account/login',
      query: {
        service: 'mtsalon',
        loginsource: 38,
        continue: url.format({
          protocol: req.protocol,
          host: req.get('host'),
          pathname: '/relation/token',
        }),
      }
    }))
  }
}

function routeMiddleware(req, res, next) {
  log.info('server', req.method, req.url);
  var hash = httpHash();
  APP_PATHS.forEach(function(appPath) {
    hash.set(appPath, function(req_, res_) {
      req_.url = '/app/entry';
      return serveHTML(req_, res_);
    });
  });
  RELATION_PATHS.forEach(function(appPath) {
    hash.set(appPath, function(req_, res_) {
      req_.url = '/relation/entry';
      return serveHTML(req_, res_);
    });
  });
  var route = hash.get(req.url);
  if (route.handler) return route.handler(req, res);
  next();
}

app
.use(cookieParser())
.use(epassportMiddleware)
.use(routeMiddleware)
.use(serveStatic);

function onerror(req, res, err) {
  res.statusCode = 500;
  res.write(err.message);
  res.end();
}

function mainHandler(req, res) {
  var urlOpts = url.parse(req.url);
  // 简单处理，无应用检查，直接返回 OK
  // TODO 开 task 给 sre， 修改应用健康检查的 url
  // 健康检查
  if (urlOpts.pathname === '/api/monitor/alive') {
    res.setHeader('content-type', 'text/plain');
    res.end('OK\n');
  } else if (urlOpts.pathname === '/download') {
    // 特殊路径，/download
    fs.createReadStream(path.resolve(__dirname, 'apps/app/download.html'))
    // 使用 .bind 创建一个新的包装函数，和 new 类似了
    .on('error', onerror.bind(null, req, res))
    .pipe(res)
    .on('error', onerror.bind(null, req, res))
  } else {
    // app 是这块代码最大的作用域外变量
    app(req, res);
  }
}

module.exports = mainHandler;

if (require.main === module) {
  var server = http.createServer(mainHandler);
  pingme({
    server: server,
    // 简单处理，无应用检查，直接返回 OK
    ping: function(cb) { cb(null); },
    status: function(cb) { cb(null); }
  });

  server.listen(PORT, '0.0.0.0', function() {
    log.info('server', this.address());
  });

  // 开发方便使用
  // cluster fork 模式下不需要
  if (!cluster.isWorker && process.env.NODE_ENV === 'development') {
    // @TODO 必须等到 build 完成? DEV 环境不需要吧
    require('./bin/build');
  }
}
```

## 依赖

```js
var fs = require('fs');
var path = require('path');
var url = require('url');
var http = require('http');
var crypto = require('crypto');
var url = require('url');
var cluster = require('cluster');

var httpHash = require('http-hash');
var ecstatic = require('ecstatic');
var pingme = require('pingme');
var log = require('npmlog');
var cookieParser = require('cookie-parser');
```

以下是 Node.js 标准模块：

- fs
- path
- url
- http
- crypto
- cluster Nodejs多核处理模块

以下是第三方 npm 包：

- http-hash [npm](https://www.npmjs.com/package/http-hash) HTTP router based on a strict path tree structure
- ecstatic [npm](https://www.npmjs.com/package/ecstatic) A simple static file server middleware that works with both Express and Flatiron
- pingme [npm](https://www.npmjs.com/package/pingme) Super simple HTTP server that can be easily pinged so that Nagios et al can know your stuff's healthy.
- npmlog [npm](https://www.npmjs.com/package/npmlog) logger for npm
- cookie-parser [npm](https://www.npmjs.com/package/cookie-parser) cookie parsing with signatures
- monkeyjs [github](https://github.com/meituan/monkey) 来自美团 Data mapping system
- request [npm](https://www.npmjs.com/package/request) Simplified HTTP request client.

## ecstatic

```js
var serveStatic = ecstatic({
  root: staticRoot,
  cache: process.env.NODE_ENV === 'development' ? 'no-store' : 3600,
  showDir: false,
});
var serveHTML = ecstatic({
  root: staticRoot,
  cache: 'no-store',
  showDir: false,
});
```

非常直白。感谢 Option 配置式的强大的语法。

## http-hash

```js
function routeMiddleware(req, res, next) {
  log.info('server', req.method, req.url);

  var hash = httpHash();

  APP_PATHS.forEach(function(appPath) {
    hash.set(appPath, function(req_, res_) { // route.handler
        // 所有的 url 都是 /app/entry，and 不觉得这个有多好，过分抽象了，直接用 history 挺好
      req_.url = '/app/entry';
      return serveHTML(req_, res_);
    });
  });

  RELATION_PATHS.forEach(function(appPath) {
    hash.set(appPath, function(req_, res_) {
      req_.url = '/relation/entry';
      return serveHTML(req_, res_);
    });
  });

  // set 好后，get
  var route = hash.get(req.url);
  if (route.handler) return route.handler(req, res);
  next();
}
```

这个小 library，领域选择的很准确，职责单一。

## 一些冷门知识

```js
if (require.main === module) {
  var server = http.createServer(mainHandler);
  pingme({
    server: server,
    // 简单处理，无应用检查，直接返回 OK
    ping: function(cb) { cb(null); },
    status: function(cb) { cb(null); }
  });

  server.listen(PORT, '0.0.0.0', function() {
    log.info('server', this.address());
  });

  // 开发方便使用
  // cluster fork 模式下不需要
  if (!cluster.isWorker && process.env.NODE_ENV === 'development') {
    // @TODO 必须等到 build 完成? DEV 环境不需要吧
    require('./bin/build');
  }
}
```

参考链接：[Node.JS: Detect if called through require or directly by command line](http://stackoverflow.com/questions/6398196/node-js-detect-if-called-through-require-or-directly-by-command-line)

```js
if(require.main === module) 
   { console.log("called directly"); }
else
   { console.log("required as a module"); }
```

