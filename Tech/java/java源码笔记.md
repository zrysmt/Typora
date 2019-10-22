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
> [【JVM源码探秘】深入理解Thread.run()底层实现](https://hunterzhao.io/post/2018/06/11/hotspot-explore-inside-java-thread-run/)

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

## 3.3 sleep + 自旋

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

## 4 ReentrantLock源码分析

> [JUC AQS ReentrantLock源码分析（一）](https://blog.csdn.net/java_lyvee/article/details/98966684)

使用示例

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

AQS(AbstractQueuedSynchronizer) 类

```java
private transient volatile Node head; // 队头
private transient volatile Node tail; // 队尾
private volatile int state; // 锁状体，加锁成功置为1， 重入+1，解锁值为0
```

Node类

```java
public class Node{
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
}
```

## 5. 线程池

> [Java并发编程：Callable、Future和FutureTask](https://www.cnblogs.com/dolphin0520/p/3949310.html)
>
> [Java并发编程：线程池的使用](https://www.cnblogs.com/dolphin0520/p/3932921.html)

- 线程创建方法如下，线程在运行结束后由虚拟机GC；当线程数量比较多时，频繁的创建和销毁很耗性能

1）继承`Thread`类； 2）实现Runnable接口

- Callable与Runnable

```java
public interface Runnable {
    public abstract void run();
}

public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result 返回值
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

- Future

Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

Future类位于java.util.concurrent包下，它是一个接口：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

- FutureTask

FutureTask是Future接口的一个唯一实现类

### 5.1 ThreadPoolExecutor构造函数

`Executor`是一个接口，它将任务的提交与任务的执行分离开来，定义了一个接收`Runnable`对象的方法`execute`。`Executor`是Executor框架中最基础的一个接口，类似于集合中的`Collection`接口。

`Executor`(接口) <---- ThreadPoolExecutor（核心类）

```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, 	BlockingQueue<Runnable> workQueue)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)

```

参数

corePoolSize - 即使空闲时仍保留在池中的线程数，除非设置 allowCoreThreadTimeOut  
maximumPoolSize - 池中允许的最大线程数  
keepAliveTime - 当线程数大于核心时，这是多余的空闲线程在终止之前等待新任务的最大时间。  
unit - keepAliveTime参数的时间单位  
workQueue - 在执行任务之前用于保存任务的队列。 该队列将仅保存execute方法提交的Runnable任务。  
threadFactory：线程工厂，用来创建线程，一般有三种选择策略。  

```
ArrayBlockingQueue;
LinkedBlockingQueue;
SynchronousQueue;
```


handler：拒绝处理策略，线程数量大于最大线程数就会采用拒绝处理策略，四种策略为

```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```

### 5.2 ThreadPoolExecutor方法

shutdown和submit，两者都用来关闭线程池，但是后者有一个结果返回。

### 5.3 线程池状态

线程池和线程一样拥有自己的状态，在ThreadPoolExecutor类中定义了一个volatile变量runState来表示线程池的状态，线程池有四种状态，分别为`RUNNING`、`SHURDOWN`、`STOP`、`TERMINATED`。

```
线程池创建后处于RUNNING状态

调用shutdown后处于SHUTDOWN状态，线程池不能接受新的任务，会等待缓冲队列的任务完成

调用shutdownNow后处于STOP状态，线程池不能接受新的任务，并尝试终止正在执行的任务

当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态
```

### 5.4 常用Executors的几个静态方法

```java
Executors.newCachedThreadPool();        //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
Executors.newSingleThreadExecutor();   //创建容量为1的缓冲池
Executors.newFixedThreadPool(int);    //创建固定容量大小的缓冲池
```

## 6. Redis

Redis 作缓存

缓存穿透：避免不存在的查询值，常使用布隆过滤器(Bloom)

## 7. Spring AOP

> https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-introduction-defn

概念：

- Aspect(切面): A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the [schema-based approach](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-schema)) or regular classes annotated with the `@Aspect` annotation (the [@AspectJ style](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj)).
- Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
- Advice: Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.
- Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
- Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an `IsModified` interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
- Target object: An object being advised by one or more aspects. Also referred to as the “advised object”. Since Spring AOP is implemented by using runtime proxies, this object is always a proxied object.
- AOP proxy: An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy.
- Weaving(环绕): linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.



































