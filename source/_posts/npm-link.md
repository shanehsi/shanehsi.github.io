title: npm link
date: 2016-03-22 14:26:37
tags:
---

参考链接：[justjs: node.js tutorials](http://justjs.com/posts/npm-link-developing-your-own-npm-modules-without-tears)

**npm link: symbolic links to the rescue**

Fortunately npm provides a tool to avoid this tedium. And it's easy to use. But there's a catch.

Here's how it's supposed to work:

1. cd to src/appy

2. Run "**npm link**". This creates a symbolic link from a global folder to the src/appy folder.

3. cd to src/mysite

4. Run "**npm link appy**". This links "node_modules/appy" in this particular project to the global folder, so that "require" calls looking for appy wind up loading it from your development folder, src/appy.

Mission accomplished... almost. If you installed Node in a typical way, using MacPorts or Ubuntu's apt-get, then npm's "global" folders are probably in a location shared system-wide, like /opt/local/npm or /usr/lib/npm. And this is not good, because it means those "npm link" commands are going to fail unless you run them as root.

如何删除呢？

参考链接：[How do I uninstall a package installed using npm link?](http://stackoverflow.com/questions/19094630/how-do-i-uninstall-a-package-installed-using-npm-link)

```shell
sudo npm rm --global foo
npm ls --global foo # 检查下
```