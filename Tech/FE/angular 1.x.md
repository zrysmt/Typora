ng-app 指令告诉Angular.js 使用 YIJIEBUYI 模块来启动应用.

模块启动经历2个阶段.

1.Config 代码,AngularJS会连接并注册好所有数据源,不确定的服务或者数据不会注入进来.

2.Run 代码,启动你的应用,在注射器创建完成之后开始执行,只有实例和常量可以被注入到Run代码块,这时Run代码块类似其他语言的main方法.

模块可以通过API来实例化控制器,指令,过滤器和服务.

常用的API如下:

1.config(configFn)

​    利用此方法可以做一些注册工作,这些工作需要在模块加载时完成.

比如 ui.router 模块,完全可以在此方法中注册.

2.constant(name, object)

​    此方法会首先运行，所以你可以用它来声明整个应用范围内的常量，并且让它们在所有配置（config方法）和实例（后面的所有方法，例如controller、service等）方法中可用.

3.controller(name,constructor)

​    它的基本作用是配置好控制器方便后面使用.

4.directive(name,directiveFactory)

​    可以使用此方法在应用中创建指令.

5.filter(name,filterFactory)

​    允许你创建命名的AngularJS过滤器，就像前面章节所讨论的那样.

6.run(initializationFn)

​    如果你想要在注射器启动之后执行某些操作，而这些操作需要在页面对用户可用之前执行，就可以使用此方法.

7.value(name,object)

​    允许在整个应用中注射值.

8.factory(name,factoryFn)

​    如果你有一个类或者对象，需要首先为它提供一些逻辑或者参数，然后才能对它初始化，那么你就可以使用这里的factory接口。factory是一个函数，它负责创建一些特定的值（或者对象）.

9.service(name,object)

​    factory和service之间的不同点在于，factory会直接调用传递给它的函数，然后返回执行的结果；而service将会使用"new"关键字来调用传递给它的构造方法，然后再返回结果.

10.provider(name,providerFn)

provider是这几个方法中最复杂的部分（显然，也是可配置性最好的部分）.

provider中既绑定了factory也绑定了service，并且在注入系统准备完毕之前，还可以享受到配置provider函数的好处（也就是config块）.