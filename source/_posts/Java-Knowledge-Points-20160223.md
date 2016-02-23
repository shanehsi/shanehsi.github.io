title: Java 知识点 2016年02月23日
date: 2016-02-23 11:14:01
tags:
---

## Jetty 启动代码

### Bootstrap.java

#### 代码

```java
import com.*.cfg.Config;
import com.*.util.Log;

public class Bootstrap {
    public Bootstrap() {}

    public static void main(String[] args)
    throws Exception {
        Log.info("\n");

        initClasspath();
        loadConfig();

        // (){ 配置 logs 地址 } -> logs
        String logs = Config.get("jetty.logs");
        if (logs == null) {
            if (new File("/opt/logs/mobile").exists()) {
                logs = "/opt/logs/mobile";
            } else {
                logs = "./logs";
            }
        }
        if (!new File(logs).exists()) {
            new File(logs).mkdirs();
        }

        // (){ 配置 webroot } -> void
        String webroot = Config.get("jetty.webroot");
        if (webroot == null) {
            if (new File("webroot").exists()) {
                webroot = "./webroot";
            } else if (new File("src/main/webapp").exists()) {
                webroot = "./src/main/webapp";
            } else {
                webroot = ".";
            }
        }
        Config.set("jetty.webroot", webroot);

        // (){ 配置 context，并作 path normalize } -> void
        String context = Config.get("jetty.context");
        if ((context == null) || (context.isEmpty())) {
            context = "/";
        } else if (context.charAt(0) != '/') {
            context = "/" + context;
        }
        Config.set("jetty.context", context);

        // ? System.getProperty
        Log.info("user.dir=" + System.getProperty("user.dir"));
        Log.info("jetty.webroot=" + webroot);
        Log.info("jetty.context=" + context);

        // () { 加载 jetty.xml，或者使用 jetty8.xml } -> jettyConfig
        InputStream input = Bootstrap.class.getResourceAsStream("/jetty/jetty.xml");
        if (input == null) {
            input = Bootstrap.class.getResourceAsStream("/jetty8.xml");
            Log.info("Booting with /jetty8.xml");
        } else {
            Log.info("Booting with /jetty/jetty.xml");
        }
        XmlConfiguration jettyConfig = new XmlConfiguration(input);

        // ? XmlConfiguration#configure()
        Object server = jettyConfig.configure();

        // ? Pattern.compile
        Pattern p = Pattern.compile("jetty-[\\w]+.xml");
        // 数组 aggregate initialization 集初始化 [link](http://book.51cto.com/art/201304/388900.htm)
        // ? WEB-INF/classes/jetty 确认下 classes 目录的生成时间，内容
        File[] paths = { new File("src/main/resources/jetty"), new File("src/test/resources/jetty"), new File(webroot, "WEB-INF/classes/jetty") };
        for (File path: paths) {
            if ((path.exists()) && (path.isDirectory())) {
                File[] jettys = path.listFiles(new FileFilter() {
                    public boolean accept(File file) {
                        // ? this 是什么？
                        // ? this.val$p
                        // ? val$p.matcher().matches()
                        return (file.isFile()) && (this.val$p.matcher(file.getName()).matches());
                    }
                });
                // ? 奇怪为什么是 break？不应该是 continue？继续看逻辑
                if (jettys == null) {
                    break;
                }
                for (File jf: jettys) {
                    // ? File#getName()
                    Log.info("Plusing " + jf.getName());
                    XmlConfiguration cfg = new XmlConfiguration(new FileInputStream(jf));
                    // XmlConfiguration.#configure(XmlConfiguration#configure())
                    cfg.configure(server);
                }
                break;
            }
        }
        Log.info("\n\n");
        // ? Object#getClass().getMethod()
        // ? new Class[0]
        // getMethod().invoke(Object, new Object[0])
        server.getClass().getMethod("start", new Class[0]).invoke(server, new Object[0]);
    }
}

```

#### 业务逻辑

```
() { 
    创建 logs 目录，

    配置 jetty.webroot，jetty.context 到 System.setProperty，

    加载 jetty.xml，并尝试在 src/main/resources/jetty，src/test/resources/jetty，
    ${webroot}/"WEB-INF/classes/jetty 找 jetty-[\\w]+.xml 文件，
    找到之后会 XmlConfiguration.#configure(server)，
    其中 server 是 jettyConfig.configure()，

    "start"
} -> void

```

### 其中 `initClasspath()`:

注意，感觉这段代码并没有对应用状态产生系统。既没有返回值，也没有 Side Effects。

#### 代码

```java
private static void initClasspath() {
    // ? System#getProperty
    // ? java.class.path
    String projectClassPath = System.getProperty("java.class.path");

    // () { 字符串去除 '/jre' } -> javaHome
    // ? java.home
    String javaHome = System.getProperty("java.home");
    if (javaHome.endsWith("/jre")) {
        javaHome = javaHome.substring(0, javaHome.length() - 4);
    }
    Log.info("JAVA_HOME=" + javaHome);
    Log.info("CLASSPATH=" + projectClassPath);
    // ProjectClassLoader 是项目代码
    List < String > classpaths = ProjectClassLoader.getClasspaths();
    boolean logger = true;
    if (projectClassPath != null) {
        StringBuffer excludedString = new StringBuffer();
        // ? File.pathSeperatorChar
        String[] tokens = projectClassPath.split(String.valueOf(File.pathSeparatorChar));
        for (String entry: tokens) {
            // ? String 赋值是 clone？
            String path = entry;
            // 删除 '-y-'，或 '-n-'
            if ((path.startsWith("-y-")) || (path.startsWith("-n-"))) {
                path = path.substring(3);
            }
            // 如果开始于 '-n-' 或者 javaHome
            if ((entry.startsWith("-n-")) || (entry.startsWith(javaHome))) {
                if (logger) {
                    excludedString.append((excludedString.length() > 0 ? "\n" : "") + "Excluded entry=" + path);
                }
            } else {
                if (logger) {
                    Log.info("ProjectClassLoader: entry=" + path);
                }
                classpaths.add(path);
            }
        }
        Log.info(excludedString.toString());
    }
}

```

#### 业务逻辑

```
() {
    获得 java.class.path -> projectClassPath，和 java.home -> javaHome，
    然后 split projectClassPath，并根据 path 的开头来增加到 classpaths.add(path)
} -> void
```

### 其中 `loadConfig` 代码：

#### 代码

```java
private static void loadConfig()
throws IOException {
    // System.getProperties.load()
    InputStream input = Bootstrap.class.getResourceAsStream("/jetty/boot.properties");
    if (input != null) {
        System.getProperties().load(input);
        Log.info("Boot Loaded...");
    } else {
        Log.info("No /jetty/boot.properties found to load...");
    }
    // 项目代码
    Config.reload();
}
```

#### 业务逻辑

```
() {
    拿到 /jetty/boot.properties，并 System.getProperties().load() 进来，

    Config.reload()
} -> void
```

### 整体业务逻辑

1. jetty 配置与启动
2. 自定义的初始化 classpath 逻辑
3. 加载 jetty 的 boot 属性配置

### 整体需要学习的知识点 

#### `System.getProperty()`

`System.getProperty()`，参考链接 [Java 中System里getProperty 方法获得系统参数](http://www.cnblogs.com/sigh-differ/archive/2012/12/25/java-system-getproperty.html)

#### `org.eclipse.jetty.xml.XmlConfiguration`

`configure()` 方法

```java
Object server = jettyConfig.configure();

XmlConfiguration cfg = new XmlConfiguration(new FileInputStream(jf));
cfg.configure(server);

server.getClass().getMethod("start", new Class[0]).invoke(server, new Object[0]);
```

参考链接：[深入Jetty源码之XmlConfiguration实现](http://www.blogjava.net/DLevin/archive/2014/05/24/414061.html)

> XmlConfiguration可以使用一个XML文件初始化一个Object实例，它支持属性设置、方法调用、新建新实例等

> 可以使用带Object实例的参数或不带参数两种方式调用configure方法，带参数表示将XML中内容配置实例中的属性，不带参数则先在内部创建class属性指定的类实例，然后使用XML的内容配置该新创建的实例的属性。

#### `System.getProperties().load()`

使用Properties的load方法，将这个文件先加载进来，之后可以使用getProperty方法将对应键的值得到。

## 其他知识点

### spring 配置

#### PropertyPlaceholderConfigurer

```xml
<bean id="propertyConfigurer" 
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath:*-common-config.properties</value>
            <value>classpath:*-custom-config.properties</value>
        </list>
    </property>
</bean>
```

参考链接：[浅析Spring框架下PropertyPlaceholderConfigurer类](http://blog.csdn.net/zh417/article/details/1728874)


> 首先要弄清楚一个概念：bean factory post-processor。
> A bean factory post-processor is a java class which implements the
org.springframework.beans.factory.config.BeanFactoryPostProcessor interface. It is executed manually  (in the case of the BeanFactory) or automatically (in the case of the ApplicationContext) to apply changes of some sort to an entire BeanFactory, after it has been constructed.
> 1. 首先bean factory post-processor实现了org.springframework.beans.factory.config.BeanFactoryPostProcessor接口。
> 2. 在BeanFactory的情况下它被手动的执行。
> 3. 在ApplicationContext的条件下它会自动的执行。
> 4. 最关键的一点是，是在一个类的实例被构造出来之后，对整个BeanFactory进行修改。

那么PropertyPlaceholderConfigurer类就是bean factory post-processor的一种，它的作用是一个资源属性的配置器，能够将BeanFactory的里定义的内容放在一个以.propertis后缀的文件中。

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
   <property name="driverClassName"><value>${driver}</value></property>
   <property name="url"><value>jdbc:${dbname}</value></property>
</bean>
```

而 jdbc.propertis：

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

引用就是：

```xml
<bean id="propertyConfigurer"  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>/WEB-INF/jdbc.properties</value>
            </list>
        </property>
 </bean>
```

将上边一段配置注册在web.xml中就可以了：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring-context.xml</param-value>
</context-param>
```

当然，不要忘了spring的监听器注册：

```xml
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

ContextLoaderListener监听器的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。来源：[ContextLoaderListener作用详解](http://blog.csdn.net/ysughw/article/details/8992322)。

### web.xml

#### WEB-INF

WEB-INF是Java的WEB应用的安全目录。所谓安全就是客户端无法访问，只有服务端可以访问的目录。如果想在页面中直接访问其中的文件，必须通过web.xml文件对要访问的文件进行相应映射才能访问。来源：[web-inf](http://baike.baidu.com/link?url=lXauen8uR9fVVAOBticfuWrBNz0s65cto_raI_B9ugUUB0uCuEENR3_rjApc2LeEcTkynqrphIEHOwN9e3xTlq)。

#### web.xml 详解

##### 加载顺序

参考：[web.xml文件详解](http://www.cnblogs.com/hellojava/archive/2012/12/28/2835730.html)

web.xml主要用来配置 Filter、Listener、Servlet 等。

WEB容器的加载顺序是： ServletContext -> context-param -> listener -> filter -> servlet。并且这些元素可以配置在文件中的任意位置。

加载过程顺序如下：

1. 启动一个 WEB 项目的时候，WEB 容器会去读取它的配置文件 web.xml，读取 <listener> 和 <context-param> 两个结点。 
2. 紧接着，容创建一个 ServletContext（servlet上下文），这个web项目的所有部分都将共享这个上下文。 
3. 容器将 <context-param> 转换为键值对，并交给servletContext。 
4. 容器创建 <listener> 中的类实例，创建监听器。 

> 所以，是现有 ServletContext，然后将 context-param 交给 ServletContext，然后创建 listener 实例

##### 文件元素详解

**1.** schema

**2.** <display-name> Web应用名称

提供GUI工具可能会用来标记这个特定的Web应用的一个名称

**3.** <context-param>上下文参数

声明应用范围内的初始化参数。它用于向 ServletContext提供键值对，即应用程序上下文信息。我们的listener, filter等在初始化时会用到这些上下文中的信息。在servlet里面可以通过getServletContext().getInitParameter("context/param")得到。

**4.** <filter>过滤器

**5.** <listener>监听器

**6.** <servlet>

<servlet></servlet> 用来声明一个servlet的数据，主要有以下子元素：

- <servlet-name></servlet-name> 指定servlet的名称
- <servlet-class></servlet-class> 指定servlet的类名称
- <init-param></init-param> 用来定义参数，可有多个init-param。在servlet类中通过getInitParamenter(String name)方法访问初始化参数
- <load-on-startup></load-on-startup>指定当Web应用启动时，装载Servlet的次序。当值为正数或零时：Servlet容器先加载数值小的servlet，再依次加载其他数值大的servlet。当值为负或未定义：Servlet容器将在Web客户首次访问这个servlet时加载它。

<servlet-mapping></servlet-mapping> 用来定义servlet所对应的URL，包含两个子元素

- <servlet-name></servlet-name> 指定servlet的名称
- <url-pattern></url-pattern> 指定servlet所对应的URL

其他参考文章：[web.xml 中的listener、 filter、servlet 加载顺序及其详解](http://zhxing.iteye.com/blog/399668)

### jetty 配置

```xml
<context-param>
    <param-name>org.eclipse.jetty.servlet.SessionIdPathParameterName</param-name>
    <param-value>none</param-value>
</context-param>

```

参考：[Session 管理](http://ykgarfield.github.io/jetty-9.2.3.v20140905-zh/session-management.html)

> Session URL 参数名称.默认为 jsessionid,但是可以使用这个 context 参数为一个特定 webapp 进行设置. 设置为 "none" 禁用 URL 重写.

### 自己写 listener

例子：

配置 xml：

```xml
<listener>
    <listener-class>com.meituan.service.mobile.tesla.core.listener.ContextListener</listener-class>
</listener>
```

代码：

```java

public class ContextListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        WebApplicationContext webApplicationContext = WebApplicationContextUtils.getWebApplicationContext(sce.getServletContext());
        // 服务启动时，初始化context
        LogService.register();
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }

}
```

### 另外一些高级用法

#### 反射

```java
// ? Object#getClass().getMethod()
// ? new Class[0]
// getMethod().invoke(Object, new Object[0])
server.getClass().getMethod("start", new Class[0]).invoke(server, new Object[0]);
```

#### Pattern.compile

```java
Pattern p = Pattern.compile("jetty-[\\w]+.xml");
return (file.isFile()) && (this.val$p.matcher(file.getName()).matches());
```

完全没有遇见过 `this.val$p` 这种用法。
