title: 一个 Maven 小技巧
date: 2016-03-10 17:51:16
tags:
---

> 2. 当修改了biz包时，需要deploy biz包，为了规避风险，线下统一发布snapshot, 线上发布release, 每次修改升级一个版本号，发布线上时把snapshot去掉(mvn versions:set -DnewVersion=1.1 -DgenerateBackupPoms=false)

另外有直接的插件，[maven-release-plugin](https://maven.apache.org/guides/mini/guide-releasing.html)。

附一则：

查找重复类: mvn enforcer:enforce

[Maven重复类的解决](http://www.cnblogs.com/zemliu/p/3277241.html)
