# 一、Springboot源码

## 文档教程收集
鲁班学院
- [剑指Spring源码（一）](https://www.cnblogs.com/CodeBear/p/10336704.htm)
- [剑指Spring源码（二）](https://www.cnblogs.com/CodeBear/p/10374261.html)

## 1.很早之前通过配置
废弃，不要用了
- web.xml
ContextLoaderListener初始化springcontext
servlet给容器使用，tomcat注册一个servlet，会拦截搜有的请求
```xml
<listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
```xml
 <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
```
-  springContext.xml 扫描业务类
-  springmvc.xml 扫描Controller
替换为下面的方式
## 2. Java configuration registers and initializes the DispatcherServlet
注册和实例化
```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletCxt) {
        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```
> https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-servlet
## 3. Tomcat
Tomcat 是web容器，实现了Servlet标准【tomcat 8 => servlet > 3.0 】
servlet 标准中，在META-INF/services/javax.servlet.ServletContainerInitializer文件中定义MyWebApplicationInitializer包名
spring framework引入Tomcat包(tomcat-embed-core),调用start方法。tomcat 会去找到WebApplicationInitializer，并且调用类的onStartup方法（tomcat不依赖spring framework）
简写的示例：
```java
public class SpringApplication {
    public static void  run() {
    Tomcat tomcat = new Tomcat();
    tomcat.setPort(9876);
    try {
        tomcat.addWebapp("/", "/Proj/web");     // web项目
        // tomcat.addContext("/", "/Proj/be");
        tomcat.start();
        tomcat.getServer().await();
    } catch(Exception e) { // ....} 
    }
}
```
# 二、java并发解读
- [【JVM源码探秘】深入理解Thread.run()底层实现](https://hunterzhao.io/post/2018/06/11/hotspot-explore-inside-java-thread-run/)
## 1. 线程
java中的线程本质上是操作系统中的线程
thread.start(java) -> start0(native,C++) -> pthread_create(C++) 
pthread_create的主体方法中调用java中的run方法
## 2. Synchronized
- [深入理解多线程（一）——Synchronized的实现原理](https://www.hollischuang.com/archives/1883)
- [深入理解多线程（二）—— Java的对象模型](http://www.hollischuang.com/archives/1910)

对于同步方法，JVM采用ACC_SYNCHRONIZED标记符来实现同步。 对于同步代码块。JVM采用monitorenter、monitorexit两个指令来实现同步。
锁住的是对象而不是代码

### 2.1 **java对象**

```
「-----------堆内存中-----------
    对象头
    实例变量
    填充数据  
   ----------------------------」
```
可以通过jol-core包打印出来存储形式
```java
class L { }
L l = new L();
ClassLayout.parseInstance(l).toPrintable();
```
| OFFSET | SIZE | TYPE DESCRIPTION | VALUE|
| ---- | ---- | ---- | ---- |
| 0 | 4 | (object header) | 01 00 00 00 (00000001 00000000 00000000 00000000) (1)|
| 4 | 4 | (object header) | 01 00 00 00 (00000001 00000000 00000000 00000000) (0)|
| 8 | 4 | (object header) | 01 00 00 00 (00000001 00000000 00000000 00000000) (1)|
| 12 | 4 | (object header) | 01 00 00 00 (00000001 00000000 00000000 00000000) (1)|

64位操作系统
对象头有12 byte = 96bit

### 2.2 **对象头**

```
「----------------------------
    Mark Word
    指向类的指针klass pointer
    数组长度（只有数组对象才有）
   ----------------------------」
```
**Mark Word**

包括堆对象的布局、类型、GC状态、同步状态和哈希标示码的基本信息。

在32位JVM机器中

| 锁状态   | 25bit                | 4bit                 | 1bit (是否偏向锁) | 2bit (锁标示位 ) |
| -------- | -------------------- | -------------------- | ----------------- | ---------------- |
| 无锁     | 对象的HashCode       | 分代年龄(最大16次)   | 0                 | 01               |
| 偏向锁   | 线程ID(23) Epoch(2)  | 分代年龄             | 1                 | 01               |
| 轻量级锁 | 指向栈中锁记录的指针 | 指向栈中锁记录的指针 | 同左              | 00               |
| 重量级锁 | 指向重量级锁的指针   | 同左                 | 同左              | 10               |
| GC标记   | 空                   | 同左                 | 同左              | 11               |
在64位JVM机器中

| 锁状态   | 25bit                | 31bit | 1bit | 4bit                 | 1bit (是否偏向锁) | 2bit (锁标示位 ) |
| -------- | -------------------- | ----- | -------------------- | ----------------- | ---------------- | ---------------- |
| 无锁     | unused | hash  | unused | 分代年龄   | 0                 | 01               |
| 偏向锁   | 线程ID(54) Epoch(2) | 同左 | unused | 分代年龄             | 1                 | 01               |
| 轻量级锁 | 指向栈中锁记录的指针 |       |       | 指向栈中锁记录的指针 | 同左              | 00               |
| 重量级锁 | 指向重量级锁的指针   |       |       | 同左                 | 同左              | 10               |
| GC标记   | 空                   |       |       | 同左                 | 同左              | 11               |

