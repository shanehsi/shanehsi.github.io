title: Java Knowledage Points 2016-03-01
date: 2016-03-01 11:56:17
tags:
---

在将两家公司的基础中间件合并时出现的错误。

[ClassNotFoundException和NoClassDefFoundError的区别](http://my.oschina.net/jasonultimate/blog/166932)。

Java支持使用Class.forName方法来动态地加载类，任意一个类的类名如果被作为参数传递给这个方法都将导致该类被加载到JVM内存中，如果这个类在 **类路径中没有被找到** ，那么此时就会在运行时抛出ClassNotFoundException异常。

要解决这个问题很容易，唯一需要做的就是要确保所需的类连同它依赖的包存在于类路径中。当Class.forName被调用的时候，类加载器会查找类路径中的类，如果找到了那么这个类就会被成功加载，如果没找到，那么就会抛出ClassNotFountException，除了Class.forName，ClassLoader.loadClass、ClassLOader.findSystemClass在动态加载类到内存中的时候也可能会抛出这个异常。

另外还有一个导致ClassNotFoundException的原因就是：当一个类已经某个类加载器加载到内存中了，此时 **另一个类加载器** 又尝试着动态地从同一个包中加载这个类。

> 两个类加载器动态加载冲突。

NoClassDefFoundError产生的原因：

如果JVM或者ClassLoader实例尝试加载（可以通过正常的方法调用，也可能是使用new来创建新的对象）类的时候却找不到类的定义。要查找的类 **在编译的时候是存在的，运行的时候却找不到了**。这个错误往往是你使用new操作符来创建一个新的对象但却找不到该对象对应的类。这个时候就会导致NoClassDefFoundError.

由于NoClassDefFoundError是有JVM引起的，所以不应该尝试捕捉这个错误。

解决这个问题的办法就是：**查找那些在开发期间存在于类路径下但在运行期间却不在类路径下的类**。

- 加载时从外存储器找不到需要的class就出现ClassNotFoundException 
- 连接时从内存找不到需要的class就出现NoClassDefFoundError

