## 1.多态

- 类

```

Animal[] animals = new Animal(3)

animals[0] = new Cat()

animals[1] = new Dog()

animals[2] = new Lion()

```

- 参数和返回类型



## 2. 继承

合约： 覆盖的规则不能改变参数， 返回值类型要兼容， 不能降低存取权限



## 3.方法的重载

与多态没有关系

- 返回类型可以不同

- 不能只改变返回类型

- 可以更改存取权限



## 3. 深入多态

抽象类，无法初始化，用来extends

接口是100%纯抽象的类

抽象的方法: 一定要被覆盖，没有方法体

   就算只有一个抽象的方法，此类也必须标记为抽象的

   对于具体的类，你必须实现所有抽象的方法

## 4.对象

任何从ArrayList<Object>取出的东西都被当作Object类型的引用而不管它原来是什么。




  编译器根据引用类型来判断有哪些method可以调用，而不是根据Object确实的类型。

  强制类型转换 

  Dog d = (Dog) x.getObject();

## 5.接口 interface

```java

public interface Pet{

    public abstract void beFrindly();

    public abstract void play();

}



public class Dog extends Canine implements Pet{

    @override

    public void beFrindly(){ ...}



    

    @override

    public void play(){ ...}

}

```

- super可以调用父类的方法



## 6.构造器与垃圾收集器

### 6.1 栈与堆

栈： 方法调用和局部变量


堆： 所有的对象(包括实例对象和局部对象)，又称为回收的堆



new 出来的对象才会在堆上占用空间



构造函数： 没有写的话，编译器会写（默认的是没有参数的）。

    构造函数没有返回类型，一定要与类名相同。 不会被继承， 可以有多个重载(参数不同)
    
    抽象的类也有构造函数
    
    私有的构造函数（private）只能在该类中调用（如Math防止被初始化）

### 6.2 new 

创建新对象时，所有继承下来的构造函数都会执行（即使父类是抽象的）。

构造函数链



构造函数没有写super(调用父类构造函数)，编译器会帮我们加上。



### 6.3 this

this() 调用对象本身的构造函数

this是对象本身，和super不能同时被调用



### 6.4 存活时间

最后一个对象引用不存在了，对象也就被GC了

- 引用永久性离开它的范围

- 对象被赋值到其他对象上

- null删去引用



## 7. 静态 static

静态方法： 不能调用非静态的变量，不能调用非静态的方法

    不用实例化， [类名].[静态方法名]

静态变量： 被同类的所有实例共享的变量

静态的final变量是常数，大写字母

final: 变量值不能改变

       方法不能被覆盖
    
       类不能被继承

如果类只有静态的方法，可以将构造函数标记为private避免被初始化



static代码块:
　　static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，

类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。
为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次



## 8.格式化

```java

String.format('%,d', 10000)

String.format('%tc',new Date())   // 完整的日期和时间

String.format('%tr',new Date())   // 只有时间

String.format('%tA, %<tB %<td',new Date())  // < 格式化程序重复利用之前用过的参数

```

## 9.日期函数

java.util.Date 停用

java.util.Calendar 抽象的

```java

import java.util.Calendar;

Calendar c = Calendar.getInstance()

```

静态import, 少写点

```

import static java.lang.System.out;

out.println(c.DATE);

```



## 10. 异常

Exception对象

RuntimeException异常，编译器不会注意，不需要声明或被包在try/catch块中

```java

// 抛出异常的方法

public void takeRisk throws BadException{ // duck掉

    if(){

        throw new BadException()

    }

}

```

try/catch块有return， finally 也会执行


异常也是多态

有多个catch块时要从小排到大

对于执行有异常的方法，要么try/catch, 要么申明（duck掉，例如 public void takeRisk **throws** BadException）



## 11. 序列化

当对象被实例化时，被该对象引用的实例变量/对象也会被序列化，这些操作时自动进行的。

`Serializable接口`被称为标记用接口，没有任何方法需要实现， 序列化类必须实现该接口。

`serialVersionUID` 标示类id

```java

import java.io.*;


public class Box implements Serializable{
    private int width;
    private int height;

    public void setWidth(int w) {
        width = w;
    }
    public  void setHeight(int h) {
        height = h;
    }

    public static void main(String[] args) {
        Box b = new Box();
        b.setWidth(100);
        b.setHeight(20);

        try {
            FileOutputStream fs = new FileOutputStream("foo.ser");
            ObjectOutputStream os = new ObjectOutputStream(fs);
            os.writeObject(b);
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

`tansient`实例变量，不会被序列化

静态变量不会被序列化



## 12. 解序列化(Deserialization)

解序列化，`tansient`变量 会被解为初始值

```java

FileInputStream fs = new FileInputStream("foo.ser");
ObjectInputStream os = new ObjectInputStream(fs);
Box one = (Box) os.readObject();
os.close();

```

## 13. java.io.File

File对象代表磁盘上的文件或目录的路径名称

```java

File f = new File('[文件名/路径]')

```

## 14. 缓冲区

```java

BufferedWriter writer = new BufferedWriter(new FileWriter(aFile))

```



```java

import java.io.*;

public class ReadFile {

    public static void main(String[] args) {
        try {
            File f = new File("text.txt");
            FileReader fr = new FileReader(f);
            BufferedReader reader = new BufferedReader(fr);

            String line = null;

            while((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

## 15. 网络java.net

**客户端**

```java

import java.io.*;
import java.net.*;

public class ChatClient {
    public void go() {
        try {
            Socket socket = new Socket("127.0.0.1", 5000 );
            InputStreamReader streamReader = new InputStreamReader(socket.getInputStream());
            BufferedReader reader = new BufferedReader(streamReader);
            String advice = reader.readLine();
            System.out.println("Today, you should " + advice);
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ChatClient chat = new ChatClient();
        chat.go();
    }
}

```

**服务端**

```java

import java.io.*;
import java.net.*;

public class ChatServer {

    String[] adviceList = {"say", "hello" , "world", "abc"};

    public void go() {
        try {
            ServerSocket serverSocket = new ServerSocket(5000);
            while (true) {
                Socket socket = serverSocket.accept(); // 等待要求到达时间
                PrintWriter writer = new PrintWriter(socket.getOutputStream());
                String advice = getAdvice();
                writer.println(advice);
                writer.close();
                System.out.println(advice);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    private String getAdvice() {
        int random = (int) (Math.random() * adviceList.length);
        return adviceList[random];
    }

    public static void main(String[] args) {
        ChatServer chatServer = new ChatServer();
        chatServer.go();
    }
}

```

但是这是单线程的，只能一次服务一个客户端



## 16.线程 Thread

- 主线程： java虚拟机调用main

- main启动新的线程，新的线程启动期间main的线程会暂时停止执行

```java

Runnable r = new MyRunnable();  // 任务

Thread t = new Thread(r);      // 工人

t.start();                     // 工人启动任务

```



- java虚拟机会在线程与原来的主线程间切换直到两者都完成为止



`Runnable`接口，只有一个方法需要实现，并且没有参数 `public void run`



执行/可执行/ 被挡住



有时候主线程先结束，有时候新建的线程会先结束。



### 线程调度

### `synchronized`同步，修饰方法使它每次只能被单一的线程存取。

```java

class TestSync implements Runnable{
    private int balance;

    public void run() {
        for (int i = 0; i < 50; i++) {
            increment();
            System.out.println("balance is: " + balance);
        }

    }

    public synchronized void increment() {
        int i = balance;
        balance = i + 1;
    }

}

public class TestSyncTest{
    public static void main(String[] args) {
        TestSync job = new TestSync();

        Thread  a = new Thread(job);
        Thread  b = new Thread(job);

        a.start();
        b.start();
    }
}

```

### 死锁

两个线程相互等待

### 线程的优先级

Java 默认的线程优先级是父线程的优先级，而非普通优先级Thread.NORM_PRIORITY，因为主线程默认优先级是普通优先级

`Thread.NORM_PRIORITY`，所以如果不主动设置线程优先级，则新创建的线程的优先级就是普通优先级`Thread.NORM_PRIORITY`



`setPriority`设置优先级



## 17. 数据结构

- `ArrayList`不是唯一的且无序的集合

- `TreeSet` 有序的**可防止重复**的集合

- `HashMap` 可成队的name/value来保存与取出

- `LinkedList` 链表，经常插入/删除高效率操作的结合

- `HashSet` 可快速查找**防止重复**的集合

- `LinkedHashMap`



ArrayList数据排序 Collections.sort(lists)

### 范型


**extends**同时代表继承和实现

```java

public <T extends Animal> void takeThing(ArrayList<T> list) // 1)

public void takeThing(ArrayList<? extends Animal> list)  // 2)

```

执行是一样的，感觉使用1）比较常见



- Comparable接口

ArrayList<Song> 具有sort的方法，那么Song类要实现Comparable接口

```java

class Song implements Comparable<Song> {

    public int compareTo(Song s) {

        return title.compareTo(s.getTitle());

    }

}

```



- Comparator(较合适的方法)

```java

class ArtistCompare implements Comparator<Song> {

    public intcompare(Song one, Song Two){

        return one.getArtist().compareto(two.getArtist())

    }

}



ArtistCompare artistCompare = new ArtistCompare();

Collections.sort(songList, artistCompare)

```

### Collection API

map虽然没有继承Collection,但也被当作其中的一分子



### 相等性

引用相等性--堆上同一对象的两个引用   == 判断即可

对象相等性--堆上的两个不同对象在意义上是相同的 

```java

if(foo.equals(bar) && foo.hashCode() == bar.hashCode()){

    // 两个引用指向同一个对象，或者两个对象是相等的

}

```

### hashSet如何检查重复

通过hashCode(), 如果相同，调用equals()继续检查

```java

class Song implements Comparable<Song> {

    public boolean equals(Object aSong) {

        Song s = (Song) aSong;

        return getTitle().equals(s.getTitle());

    }

    public int hashCode() {

        return title.hashCode();

    }

    public int compareTo(Song s) {

        return title.compareTo(s.getTitle());

    }

}

```

equals()被覆盖过，则hashCode() 也必须被覆盖

a.equals(b) 与 a.hashCode() == b.hashCode()等值，反过来就不行


### TreeSet的元素必须是Comparable

集合中的元素必须是有实现的Comparable的类型 或 重载Comparator参数的构造函数来创建



## 18. jar
- 1.

```bash

javac -d ../classes *.java   // javac文件全部到classes中

```

- 2.classes文件夹新增`manifest.txt`文件，写入

```

Main-Class: Main

```

- 3.执行命令，产生了app1.jar文件

```bash

jar -cvmf manifest.txt app1.jar *.class

```

运行

```

java -jar app1.jar

```

## 19. 包

```java

package me.ele;

```

一个包内的不同文件类可以相互引用，不用import, 但是javac编译的时候，引用的文件要先编译，或者使用`javac *.js`全部编译



进入classes文件夹执行

```

java me.ele.Test

```

打包

```

jar -cvmf manifest.txt app1.jar me   // me是文件夹

```

```

jar -tf app1.jar 列出

jar -xf app1.jar 解压

```

## 20. 分布式计算

Clinet对象 --> 客户端辅助设施(stub) --> 服务端辅助设施(skeleton) --> Service对象



## 21. Servlet

HttpServlet