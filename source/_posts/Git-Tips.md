title: Git Tips
date: 2016-01-20 13:49:17
tags:
---

## git submodule

[git1.8.4](https://github.com/git/git/blob/master/Documentation/RelNotes/1.8.4.txt#L38-L40) (July 2013) 之后不需要回到 . 目录。

直接到 submodule 期望所在的目录：

```
cd ./src/vendor/
git submodule add <git@github ...> <project-name>
```

参考链接 [How do I add a submodule to a sub-directory?](http://stackoverflow.com/questions/9035895/how-do-i-add-a-submodule-to-a-sub-directory)。

