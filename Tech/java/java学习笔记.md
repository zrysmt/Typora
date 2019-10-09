## 1.项目结构
- 开放接口层：可直接封装Service方法暴露成RPC接口；通过Web封装成http接口；进行网关安全控制、流量控制等。
- 终端显示层：各个端的模板渲染并执行显示的层。当前主要是velocity渲染，JS渲染，JSP渲染，移动端展示等。 
- Web层：主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。 
- Service层：相对具体的业务逻辑服务层。 
- Manager层：通用业务处理层，它有如下特征： 
    1） 对第三方平台封装的层，预处理返回结果及转化异常信息； 
    2） 对Service层通用能力的下沉，如缓存方案、中间件通用处理； 
    3） 与DAO层交互，对多个DAO的组合复用。 
- DAO层：数据访问层，与底层MySQL、Oracle、Hbase等进行数据交互。 
- 外部接口或第三方平台：包括其它部门RPC开放接口，基础平台，其它公司的HTTP接口。

```
org.spring.springboot.controller - Controller 层
org.spring.springboot.dao - 数据操作层 DAO
org.spring.springboot.domain - 实体类
org.spring.springboot.service - 业务逻辑层
Application - 应用启动类
application.properties - 应用配置文件，应用启动会自动读取配置
```
## 2. eleme项目
- 1. 下载maven
```
brew install maven
```
## 3. spring资源
- [spring新手指南](https://spring.io/guides)
- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
- [系列学习文档](http://www.spring4all.com/article/246)
## 4. 依赖注入和控制反转
Spring框架控制反转（IOC）组件通过提供一系列的标准化的方法把完全不同的组 件组合成一个能够使用的应用程序来解决这个问题。
更合适的名字应该叫**依赖注入**
## 5.  自定义属性
### spring配置优先级
Spring Boot 不单单从 application.properties 获取配置，所以我们可以在程序中多种设置配置属性。按照以下列表的优先级排列：
- 1.命令行参数
- 2.java:comp/env 里的 JNDI 属性
- 3.JVM 系统属性
- 4.操作系统环境变量
- 5.RandomValuePropertySource 属性类生成的 random.* 属性
- 6.应用以外的 application.properties（或 yml）文件
- 7.打包在应用内的 application.properties（或 yml）文件
- 8.在应用 @Configuration 配置类中，用 @PropertySource 注解声明的属性文件
- 9.SpringApplication.setDefaultProperties 声明的默认属性
可见，命令行参数优先级最高。这个可以根据这个优先级，可以在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用。
###  java对象配置
@Component 作为 Bean 注入到 Spring 容器中
@ConfigurationProperties(prefix = "home")   //将配置文件中以 home 前缀的属性值自动绑定到对应的字段中
### random.* 属性 ==> 生产随机数
## 6. Springboot 整合 Mybatis , 使用xml配置SQL
- [博客地址](http://www.spring4all.com/article/145)
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="org.spring.springboot.dao.CityDao">
    <resultMap id="BaseResultMap" type="org.spring.springboot.domain.City">
        <result column="id" property="id" />
        <result column="province_id" property="provinceId" />
        <result column="city_name" property="cityName" />
        <result column="description" property="description" />
    </resultMap>
    <parameterMap id="City" type="org.spring.springboot.domain.City"/>
    <sql id="Base_Column_List">
        id, province_id, city_name, description
    </sql>
    <select id="findByName" resultMap="BaseResultMap" parameterType="java.lang.String">
        select
        <include refid="Base_Column_List" />
        from city
        where city_name = #{cityName}
    </select>
</mapper>
```
## 7. 监控与管理
spring-boot-starter-actuator
## 8. java.util.concurrent
- CountDownLatch 给定计数器初始化(1). await 方法阻塞，直到由于countDown()方法的调用而导致计数器达到0，之后所有等待线程被释放

```java
  CountDownLatch latch = new CountDownLatch(1); // 1
  latch.await();  // 2
  latch.countDown()； // 3 
```
## 8. Bean
Full @Configuration 
Lite @Bean : 当@Bean方法在没有使用@Configuration注解的类中声明时，它们被称为 以“lite”进行处.通常，在“lite”模式下一个@Bean方法不应该调用其他的@Bean方法
```java
@Configuration 
public class AppConfig { 
    // 要定义一个bean，只需在一个方法上使用@Bean注
    // 默认情况下，bean名称与 方法名称相同
     @Bean
     public TransferService transferService() { 
        return new TransferServiceImpl(); 
    } 
}
```
@Bean(initMethod = "init")
@Bean(destroyMethod = "cleanup") //可禁用默认 （推测）模式
@Bean(name = "myFoo") // 自定义名称
## 9. request
```java
import org.apache.http.impl.client.HttpClients;
 CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet(url);
String responseBody = httpclient.execute(httpget, new HttpResponseHandler())
```
post
```
HttpPost httpPost = new HttpPost(url);
CloseableHttpClient httpclient = HttpClients.createDefault();
String responseBody = httpclient.execute(httpPost, new HttpResponseHandler());
```
## 10. fastJSON
```java
String text = JSON.toJSONString(obj); //序列化
VO vo = JSON.parseObject("{...}", VO.class); //反序列化
```
## 11. SpringBoot 过滤器、监听器、拦截器
- http://www.importnew.com/29401.html
#### 过滤器
过滤器Filter，是Servlet的的一个实用技术。可通过过滤器，对请求进行拦截，比如读取session判断用户是否登录、判断访问的请求URL是否有访问权限(黑白名单)等。主要还是可对请求进行预处理。接下来介绍下，在springboot如何实现过滤器功能。
#### 监听器
Listen是servlet规范中定义的一种特殊类。用于监听servletContext、HttpSession和servletRequest等域对象的创建和销毁事件。监听域对象的属性发生修改的事件。用于在事件发生前、发生后做一些必要的处理。一般是获取在线人数等业务需求。
#### 拦截器
以上的过滤器、监听器都属于Servlet的api，我们在开发中处理利用以上的进行过滤web请求时，还可以使用Spring提供的拦截器(HandlerInterceptor)进行更加精细的控制。
## 12. 同步/异步 & 阻塞/非阻塞
要注意这两个是不同纬度的概念：同步/异步指的是程序跟I/O操作的关系，如果由程序自己控制I/O操作则是同步，如果将I/O操作委托给其他（例如操作系统），则是异步；阻塞/非阻塞指的是I/O操作时对于CPU的消耗，如果CPU停下来等待I/O操作完，再进行其他操作，则为阻塞，否则是非阻塞。
根据组合，I/O交互可分为同步阻塞、同步非阻塞、异步阻塞、异步非阻塞四种。其中BIO为同步阻塞的，NIO为同步非阻塞，而AIO（也称为NIO2，书中没有讲到AIO）是异步非阻塞的。
## 13. NIO多人聊天
> https://github.com/2368001770/NIO-Chat-Room


BIO：阻塞式IO模型、弹性收缩能力差、多线程消耗资源

NIO：非阻塞式IO模型、弹性收缩能力强、单线程节省资源

通道（Channel）：双向性、非阻塞性、操作唯一性（只能通过Buffer进行操作）
> * 文件类————FileChannel
> * UDP类————DatagramChannel
> * TCP类：————ServerSocketChannel、SocketChannel

缓冲区（Buffer）：一块内存区域、读写Channel中的数据
> * 属性————容量（Capacity）、上限（Limit）、位置（Position）、标记（Mark）

多路复用器（Selector）：I/O事件就绪选择、NIO网络编程基础之一

选择键SelectionKey（）：四种就绪状态常量、有价值的属性

NIO网络编程基础步骤：
> * 1、创建Selector
> * 2、创建ServerSocketChannel，并绑定监听端口
> * 3、将Channel设置为非阻塞模式
> * 4、将Channel注册到Selector上，监听连接事件
> * 5、循环调用selector的select方法，检测就绪情况
> * 6、调用selectorKeys方法获取就绪Channel集合
> * 7、判断就绪事件的种类，调用不同业务处理方法
> * 8、重复第四步

NIO网络编程缺陷：
> * 1、NIO类库和API繁杂
> * 2、可靠性能力补齐、工作量和难度非常大
> * 3、Selector空轮询（bug），CPU 100%

## 14. Spring Cloud
《day01-Spring Cloud 微服务入门.docx》
- 注册中心Eureka
- 负载均衡 Ribbon
- 容错保护：Hystrix
- 声明式的REST调用:  Feign
-  服务网关: Zuul
- 配置管理中心： Config
- 消息总线： Bus

**rest请求**
```java
public Item queryItemById(Long id) {
        return this.restTemplate.getForObject("http://127.0.0.1:8081/item/"+ id, Item.class);
}
```
**使用eureka注册中心**
```java
public Item queryItemById(Long id) {
        String serviceId = "ruyi-microservice-item";
        List<ServiceInstance> instances = this.discoveryClient.getInstances(serviceId);
        if(instances.isEmpty()){
            return null;
        }
        // 为了演示，在这里只获取一个实例
        ServiceInstance serviceInstance = instances.get(0);
        String url = serviceInstance.getHost() + ":" + serviceInstance.getPort();
        return this.restTemplate.getForObject("http://" + url + "/item/" + id, Item.class);
    }
```
**简化+容错Hystrix**
```java
   @HystrixCommand(fallbackMethod = "queryItemByIdFallbackMethod")
    public Item queryItemById(Long id) {
        String serviceId = "ruyi-microservice-item";
        return this.restTemplate.getForObject("http://" + serviceId + "/item/" + id, Item.class);
    }
    public Item queryItemByIdFallbackMethod(Long id){ // 请求失败执行的方法
        return new Item(id, "查询商品信息出错!", null, null, null);
    }
```
**Feign**
ItemFeignClient.java
```java
@FeignClient(value = "ruyi-microservice-item")
public interface ItemFeignClient {
    // 这里定义了类似于SpringMVC用法的方法，就可以进行RESTful的调用了
    @RequestMapping(value = "/item/{id}", method = RequestMethod.GET)
    public Item queryItemById(@PathVariable("id") Long id);
}
```
```java
    private ItemFeignClient itemFeignClient;

    @HystrixCommand(fallbackMethod = "queryItemByIdFallbackMethod")
    public Item queryItemById(Long id) {
        return itemFeignClient.queryItemById(id);
    }
```

Feign： 是netflix提供的服务间基于http的rpc调用框架
在springcloud体系中实现rpc的组件有2个，一个是ribbon，另一个是feign，而且feign在底层封装了ribbon
## 15. Java agent
Java Agent也是一个Jar包，只是启动方式和普通Jar包有所不同，对于普通的Jar包，通过指定类的main函数进行启动，但是Java Agent并不能单独启动，必须依附在一个Java应用程序运行，有点像寄生虫的感觉
## 16. Classloader
双亲委派模型

Java语言系统自带有三个类加载器:
`Bootstrap ClassLoader` 最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。另外需要注意的是可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。比如java -Xbootclasspath/a:path被指定的文件追加到默认的bootstrap路径中。我们可以打开我的电脑，在上面的目录下查看，看看这些jar包是不是存在于这个目录。
`Extention ClassLoader` 扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件。还可以加载-D java.ext.dirs选项指定的目录。
`Appclass Loader`也称为SystemAppClass 加载当前应用的classpath的所有类。

## 17.Spring-retry

两个方法的返回值必须相同

```java
@Retryable(value = {Exception.class}, maxAttempts = 5, backoff = @Backoff(delay = 5000l, multiplier = 1))
    public boolean actCallback(ActCallbackParams params, String type) {}
```

```java
@Recover
public boolean recover(Exception e) {}
```

## 18. ==和equals

> [ATA java中的==和equals](https://www.atatech.org/articles/150993?spm=ata.home.0.0.11fd7536WvsJEk&flag_data_from=home_algorithm_article)

**基本使用**

1) 基本数据类型：`byte,short,char,int,long,float,double,boolean`的比较使用 "==",比较的是值

2) 引用数据类型：类，接口 ，数组等，使用 == 和 equals比较的是他们在内存中的存放地址，所以，除非是同一个new出来的对象，他们的比较后的结果为true，否则比较后结果为false。

```java
Student a = new Student("100","test","test");
Student b = new Student("100","test","test");
System.out.println(a == b); //false
System.out.println(a.equals(b));//false
```

**进阶**

实际在项目中可能会遇到，根据类中某一些属性判断类是否相等。这时候需要考虑重写equals方法。

jdk的规范中定义：如果两个对象相同，那么它们的hashCode值一定要相同。所以 重写equals方法时，对应的`hashCode`方法也需要重写









