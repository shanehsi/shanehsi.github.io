title: Java 知识点 20160229
date: 2016-02-29 16:30:04
tags:
---

区分目录：

- src/main/resources/META-INF/
- webapps下的那个META-INF


错误：

`java.lang.AbstractMethodError`

这个问题是因为接口中定义的方法名和实现类的的方法名相同，但是参数的数据类型却不一致，而且实现类的方法没有 `@Override` 造成的。

所以在实现接口或覆写父类方法时，最好的实践是加上 `@Override`。

详细：

```
http://www.google.com/search?q=java.lang.AbstractMethodError%3A+org.apache.xerces.dom.DocumentImpl.setXmlStandalone
```

