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
servlet 标准中，在`META-INF/services/javax.servlet.ServletContainerInitializer`文件中定义`MyWebApplicationInitializer包名`
spring framework引入Tomcat包(tomcat-embed-core),调用`tomcat.start`方法。tomcat 会去找到系统的和用户自定义的WebApplicationInitializer，并且调用类的onStartup方法（tomcat不依赖spring framework）
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
pthread_create的主体方法中调用(JNI)java中的run方法

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
| 8 | 4 | (object header) | 43 c1 00 20 (01000011 11000001 00000000 01000000) (536920387) |
| 12 | 4 | (loss due to the next object alignment) |  |

64位操作系统，对象头有12 byte = 96bit

### 2.2 **对象头**

```
「-----------对象头--------------
    Mark Word
    指向类的指针klass pointer
    数组长度（只有数组对象才有）
   ----------------------------」
```
**Mark Word**

包括堆对象的布局、类型、GC状态、同步状态和哈希标示码的基本信息。

在32位JVM机器中

| 锁状态   | 25bit                       | 4bit               | 1bit (是否偏向锁) | 2bit (锁标示位 ) |
| -------- | --------------------------- | ------------------ | ----------------- | ---------------- |
| 无锁     | 对象的HashCode              | 分代年龄(最大16次) | 0                 | 01               |
| 偏向锁   | 线程ID(23) Epoch(2)         | 分代年龄           | 1                 | 01               |
| 轻量级锁 | [<---指向栈中锁记录的指针   | -----------------  | ------------>]    | 00               |
| 重量级锁 | [<---指向重量级锁的指针     | -----------------  | ------------>]    | 10               |
| GC标记   | [<---空-------------------- | -----------------  | ------------>]    | 11               |
在64位JVM机器中

| 锁状态   | 25bit                    | 31bit   | 1bit    | 4bit     | 1bit (是否偏向锁) | 2bit (锁标示位 ) |
| -------- | ------------------------ | ------- | ------- | -------- | ----------------- | ---------------- |
| 无锁     | unused                   | hash    | unused  | 分代年龄 | 0                 | 01               |
| 偏向锁   | [<---线程ID(54) Epoch(2) | --->]   | unused  | 分代年龄 | 1                 | 01               |
| 轻量级锁 | 指向栈中锁记录的指针     | ------- | ------- | -------  | ----->]           | 00               |
| 重量级锁 | 指向重量级锁的指针       | ------- | ------- | -------  | ----->]           | 10               |
| GC标记   | [<---空----------------- | ------- | ------- | -------  | ----->]           | 11               |

分析打印出来的对象头信息, hash code 没有经过JVM计算，所以前几位都为0

```
01 00 00 00 (00000001 00000000 00000000 00000000) (1)  

01 00 00 00 (00000001 00000000 00000000 00000000) (0)  

43 c1 00 20 (01000011 11000001 00000000 01000000) (536920387)
```

```java
l.hashCode(); // 计算hashCode = 0x1540e19d（Little-Endian: 低地址存放低位）
```

其中【】扩起来的是25bit没使用的空间

```
01 9d e1 40 (00000001 10011101 11100001 01000000) (1088527617)  

15 00 00 00 (【0】0010101 【00000000 00000000 00000000】) (21)  

43 c1 00 20 (01000011 11000001 00000000 01000000) (536920387)
```

### 2.3 锁

- jdk 1.6前： 互斥是在OS中进行(mutex 比较耗性能)

- jdk 1.6后： 锁的在jdk中进行，

  1）没有资源竞争的同步代码 ---- 偏向锁，一个线程调用

  2）有资源竞争（多个线程执行）

  ​    2.1） 多个线程交替执行 ---- 轻量级锁

  ​    2.2） 线程竞争 ---- 重量级锁(OS级别)

JVM偏向锁延迟了(启动main方法前，会先启动jdk)，可以使用参数关闭偏向锁延迟`VM options`: -XX: BiasedLockingStartupDelay=0

## 3. 多线程同步内部实现

模拟

### 3.1 自旋

```java
volatile int status = 0; // 标示 是否有线程上锁成功
void lock {
  while(!compareAndSet(status, 1)){ 
    // 循环执行，status==1时无限循环=>Lock住， status==0，将status置为1往下执行
    // --------------------------------------------------------- (1)
  }
  // 逻辑代码
}
void unlock{ status = 0; }
  
boolean compareAndSet(int except, int newValue) {
  // CAS操作，修改status成功则返回ture
}
```

缺点： 耗费CPU

## 3.2 yield + 自旋

```java
yield(); //----------------------------------------------------- (1)
```

yield就是让出CPU，当只有两个线程时yield时有效的

## 3.3 sleep

```java
sleep(10); //---------------------------------------------------- (1)
```

睡眠时间不能控制

## 3.4 park + 自旋

```java
volatile int status = 0; 
Queue parkQueue;

void lock {
  while(!compareAndSet(status, 1)){ 
    park();
  }
  unlock();
}
void unlock{ 
  lock_notify();
}
  
void park() {
  parkQueue.add(currentThread); // 将当前线程加入到等待队列
  releaseCpu(); // 释放CPU
}
void lock_notify() {
  Thread t = parkQueue.header(); // 得到要唤醒的头部线程
  unpark(t); // 唤醒线程
}
```

这里的parkQueue类似于AQS

AQS(AbstractQueuedSynchronizer) 类主要代码

```java
private transient volatile Node head; // 队头
private transient volatile Node tail; // 队尾
private volatile int state; // 锁状体，加锁成功置为1， 重入+1，解锁值为0
```

示例：

```java
import java.util.concurrent.locks.LockSupport;

public class Example {
  public static void main(String[] args) throws InterruptedException{
    System.put.Println('m1');
    Thread t1 = new Thread("t1") {
      @Override
      public void run(){
      	System.put.Println('t1');
        LockSupport.park();
				System.put.Println('t2');
			}
    }
    t1.start();
    Thread.sleep(5000);
    System.put.Println('m2');
    LockSupport.unpark();
  }
}
```

打印结果

```
m1 
t1
m2 
t2
```

## 3.5 ReentrantLock

```java
public class Example2 {
  public static void main(String[] args) throws InterruptedException{
    final ReentrantLock lock = new ReentrantLock(true);

    Thread t1 = new Thread("t1") {
      @Override
      public void run(){
      	lock.lock();
        logic();
        lock.unlock();
			}
    }
    t1.start();
    Thread.sleep(10);
  }
  
  public static void logic(){
    try{
      Thread.sleep(2000);
    } catch(InterruptedException e) {
      e.printStatckTrace();
    }
  }
}
```











