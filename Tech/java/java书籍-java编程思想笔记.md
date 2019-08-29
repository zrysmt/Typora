## 1. Java基本类型
- Java的基本类型主要分为整数型，浮点型，字符型，布尔型
```
         整数型：byte，short，int，long；
         浮点型：float，double
         布尔型：boolean
         字符型：char
```
- 基本类型的大小
```
         1byte：8bit，一个bit代表一个1或者0，是计算机的基本单位。
         byte：1byte short：2 byte int：4byte long：8byte
         float：4byte double：8个byte
         char：2byte        
         boolean：值只可以为true或者false ，理论上只占据一个bit，但是实际是占据了一个byte
```
## 2. this
- return this 返回句柄
- static方法中没有this
## 3. GC
- finalize() 观察垃圾收集的过程
- 为强制收尾，可以先调用`System.gc()`,再调用`System.runFinalize()`
## 4. 初始化
- 数组初始化 
```
int[] a1 = { 1, 2, 3, 4}
Integer[] a2 = new Integer[] {
    new Integer(1),
    new Integer(2),
    3
}
```
## 5.构造器
构造器实际上是static方法      
## 6. 多态   
```
Animal[] animals = new Animal(3)
animals[0] = new Cat()
animals[1] = new Dog()
animals[2] = new Lion()
```
条件： 
1）继承基类 2）重写 3）父类引用指向子类对象
定义： 
1、多态是允许你将父对象设置成为一个或更多的他的子对象相等的技术(可以理解为这是一种 向上转型) 
2、多态方法调用允许一种类型表现出与其他相似类型之间的区别，只要它们都是从同一基类导出而来。
**重载**与多态没有关系
在同一个类中，具有相同的方法名，但每个重载的方法具有一个独一无二的参数类型列表（可以是不同的类型，可以是不同的参数个数，也可以是相同的类型不同的顺序），但不能通过返回值去区分重载方法。

