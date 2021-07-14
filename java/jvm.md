---
sort: 2
---

# JVM

## 类加载器

Boostrap启动类加载器   ------ 加载lib/*.jar(包括rt.jar、jce.jar)
Extension扩展类加载器  ------ 加载jre/lib/ext/*.jar或者由-Djava.ext.dirs指定
Application应用程序类加载器 ---- 加载classpath指定内容
Custom自定义类加载器 ------ 自定义的ClassLoader指定的内容

双亲委派的好处: 稳定，防止一些主类(Object,String)被重写

### 打破双亲委派

SPI；例如JDBC,设置连接时的DriverManager的类，在IDE写了没有报错，因为这些是jdk定义了接口，mysql-connector-java.jar只是实现接口。
而jdk的类加载器是Boostrap启动类加载器，运行时报错是因为在Application应用启动类加载器
JVM启动的时候会自动初始化Launcher.class中会把线程上下文设置成AppClassLoader加载器(Thread.currentThread().getContextClassLoader)
