title: Java 知识点 2016年02月24日
date: 2016-02-24 11:30:33
tags:
---

## MethodInvokingFactoryBean

从配置文件开始看。

```xml
<bean id="mnsInvokerBean" class="com.*.*.MnsInvoker" />
<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="mnsInvokerBean" />
    <property name="targetMethod">
        <value>registerHttpService</value>
    </property>
    <property name="arguments">
        <list>
            <value>${tesla_appkey}</value>
            <value>${jetty.port}</value>
        </list>
    </property>
</bean>
```

主要是这个 `MethodInvokingFactoryBean`。

参考：[Spring实战之org.springframework.beans.factory.config.MethodInvokingFactoryBean](http://blog.csdn.net/kongxx/article/details/6011441)。

讲解的很直观。

在用spring管理我们的类的时候有时候希望有些属性值是

- 来源于一些配置文件，
- 系统属性，
- 或者一些方法调用的结果

对于前两种使用方式可以使用spring的PropertyPlaceholderConfigurer类来注入，对于后一种则可以使用org.springframework.beans.factory.config.MethodInvokingFactoryBean类来生成需要注入的bean的属性。

通过MethodInvokingFactory Bean类，可注入方法返回值。 MethodInvokingFactoryBean用来 **获得某个方法的返回值**，该方法既可以是静态方法，也可以是实例方法。该方法的返回值 **可以注入bean实例属性**，**也可以直接定义成bean实例**。

另参考：[Spring Boot Hello MethodInvokingFactoryBean and MethodInvokingBean](http://blog.sina.com.cn/s/blog_72ef7bea0102wa0v.html)。

## bean 配置之 init-method 和 lazy-init。

```xml
<bean id="mtConfigClient" class="com.*.*.config.MtConfigClient" init-method="init" lazy-init="false">
```

关于 `init-method` 和 `lazy-init`。

其中关于 `init-method`，涉及到的概念是 [Spring bean 的生命周期](http://sexycoding.iteye.com/blog/1046775)。Spring 允许 Bean 在初始化完成后以及销毁前执行特定的操作。

常用的三种指定特定操作的方法：

- 通过实现InitializingBean/DisposableBean 接口来定制初始化之后/销毁之前的操作方法；
- 通过 <bean> 元素的 init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法；
- 在指定方法上加上@PostConstruct或@PreDestroy注解来制定该方法是在初始化之后还是销毁之前调用。

这几种方法的先后顺序为（参考：[Spring容器中的Bean几种初始化方法和销毁方法的先后顺序](http://blog.csdn.net/caihaijiang/article/details/8629725))：

**Bean在实例化的过程中：Constructor > @PostConstruct >InitializingBean > init-method**

**Bean在销毁的过程中：@PreDestroy > DisposableBean > destroy-method**

其中关于 `lazy-init`，默认是 false。可以在 applicationContext.xml 里配置：

```xml
<beans default-lazy-init="true" >   
      <bean class ="org.xxxx.bean" >   
      ......
</beans> 
```

这个应该只在开发环境中可以使用。

> ApplicationContext实现的默认行为就是在启动时将所有singleton bean提前进行实例化。提前实例化意味着作为初始化过程的一部分，ApplicationContext实例会创建并配置所有的singleton bean。通常情况下这是件好事，因为这样在配置中的任何错误就会即刻被发现（否则的话可能要花几个小时甚至几天）。

