title: Building command line tools with Node.js
date: 2016-06-08 10:43:56
tags:
---

- packaging new shell commands with npm
- parsing command line options
- reading text and passwords from stdin
- dodging callbacks with ES6 generators
- error output and codes
- coloring terminal output
- rendering an ASCII progress bar

## Packaging shell commands

初始化一个 node 项目：

`npm init`

index.js：

```diff
+ #!/usr/bin/env node
+ console.log('Hello, world!');
```

package.json

```diff
...
  "author": "Tim Pettersen",
  "license": "Apache-2.0",
+ "bin": {
+   "snippet": "./index.js"
+ }
}
```

安装项目到全局，可以测试：

```shell
$ npm install -g
$ snippet
Hello, world!
```

查看命令的路径：

```shell
$ which snippet
/usr/local/bin/snippet
$ readlink /usr/local/bin/snippet
../lib/node_modules/bitbucket-snippet/index.js
```


方便开发：

```shell
$ npm link
/usr/local/bin/snippet -> /usr/local/lib/node_modules/bitbucket-snippet/index.js
/usr/local/lib/node_modules/bitbucket-snippet -> /Users/kannonboy/src/bitbucket-snippet
```

最后 publish

```shell
npm publish
```

## Parsing command line options

```shell
$ npm install --save commander
```

```diff
#!/usr/bin/env node
- console.log('Hello, world!');
+ var program = require('commander');
+
+ program
+  .arguments('<file>')
+  .option('-u, --username <username>', 'The user to authenticate as')
+  .option('-p, --password <password>', 'The user\'s password')
+  .action(function(file) {
+    console.log('user: %s pass: %s file: %s',
+        program.username, program.password, file);
+  })
+  .parse(process.argv);
```

quick test：

```shell
$ snippet -u kannonboy -p correcthorsebatterystaple my_awesome_file
user: kannonboy pass: correcthorsebatterystaple file: my_awesome_file
```

自动生成一些 help 信息：

```shell
$ snippet --help

  Usage: snippet [options] <file>

  Options:

    -h, --help                 output usage information
    -u, --username <username>  The user to authenticate as
    -p, --password <password>  The user\'s password
```

## Prompting for user input

安装：

```shell
$ npm install --save co co-prompt
```

```diff
+ var co = require('co');
+ var prompt = require('co-prompt');
  var program = require('commander');
...
  .option('-u, --username <username>', 'The user to authenticate as')
  .option('-p, --password <password>', 'The user\'s password')
  .action(function(file) {
+    co(function *() {
+      var username = yield prompt('username: ');
+      var password = yield prompt.password('password: ');
       console.log('user: %s pass: %s file: %s',
-          program.username, program.password, file);
+          username, password, file);
+    });
  })
...
```

quick test：

```shell
$ snippet my_awesome_file
username: kannonboy
password: *************************
user: kannonboy pass: correcthorsebatterystaple file: my_awesome_file
```

## POSTing the snippet


## Handling error cases

So we're handling the happy case OK, but what if the upload fails or the user enters the wrong credentials? The UNIX-y way to handle it would be to write an message to standard error and exit with a non-zero code, so let's do that.

```diff
...
  request
    .post('https://api.bitbucket.org/2.0/snippets/')
    .auth(username, password)
    .attach('file', filename, file)
    .set('Accept', 'application/json')
    .end(function (err, res) {
+     if (!err && res.ok) {
        var link = res.body.links.html.href;
        console.log('Snippet created: %s', link);
+       process.exit(0);
+     }
+
+     var errorMessage;
+     if (res && res.status === 401) {
+       errorMessage = "Authentication failed! Bad username/password?";
+     } else if (err) {
+       errorMessage = err;
+     } else {
+       errorMessage = res.text;
+     }
+     console.error(errorMessage);
+     process.exit(1);
    });
```

## Adding a progress bar

```shell
npm install --save progress
```

```diff
+ var fs = require('fs');
+ var ProgressBar = require('progress');
  var chalk = require('chalk');
  var request = require('superagent');
...
  var username = yield prompt('username: ');
  var password = yield prompt.password('password: ');

+ var fileSize = fs.statSync(file).size;
+ var fileStream = fs.createReadStream(file);
+ var barOpts = {
+   width: 20,
+   total: fileSize,
+   clear: true
+ };
+ var bar = new ProgressBar(' uploading [:bar] :percent :etas', barOpts);
+
+ fileStream.on('data', function (chunk) {
+   bar.tick(chunk.length);
+ });

  request
    .post('https://api.bitbucket.org/2.0/snippets/')
    .auth(username, password)
-   .attach('file', file)
+   .attach('file', fileStream)
    .set('Accept', 'application/json')
...
```

## Summary

We've barely scraped the surface of what's possible with command line tooling in node. As per Atwood's Law, there are npm packages for elegantly handling standard input, managing parallel tasks, watching files, globbing, compressing, ssh, git, and almost everything else you did with Bash. Plus, there are nice APIs for forking child processes if you need to fall back on another shell script or command that you can't find a decent JavaScript implementation of.
The source code for the example we built above is liberally licensed and available on Bitbucket and, of course, published to npm. I've also implemented a couple of features that aren't shown here, like OAuth, so you don't have to keep typing in your username and password. You can start using it yourself with a simple:


