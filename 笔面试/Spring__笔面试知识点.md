# Spring__笔面试知识点



## IOC

IoC（Inverse of Control：控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spirng 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。** **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。**

IoC 最常见以及最合理的**实现方式**叫做**依赖注入**（Dependency Injection，简称 DI）。

IOC的好处：

1. 对象之间的**耦合度**或者说依赖程度降低；
2. 资源变的**容易管理**；比如你用 Spring 容器提供的话很容易就可以实现一个单例。

Spring提供了好几种的方式来给属性赋值

- **1) 通过构造函数**
- **2) 通过set方法给属性注入值**
- 3) p名称空间
- 4)自动装配(了解)
- **5) 注解**

### BeanFactory、ApplicationContext







## AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，**实现了某个接口**，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于**没有实现接口**的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

<img src="C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200727172016401.png" alt="image-20200727172016401" style="zoom:50%;" />

Spring AOP 已经集成了AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

Spring AOP 和 AspectJ AOP的区别：**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** 如果我们的**切面比较少**，那么两者性能差异不大。但是，当**切面太多的话，最好选择 AspectJ** ，它比Spring AOP 快很多。

横切逻辑代码存在的问题（AOP解决的问题）：

- 代码重复问题
- 横切逻辑代码和业务代码混杂在一起，代码臃肿，不易维护





## Bean

在 Spring 中，那些组成应用程序的主体及由 Spring IOC 容器所管理的对象，被称之为 bean，**由 IOC 容器初始化、装配及管理**。**Spring中的bean默认都是单例的**，**Spring的单例是基于BeanFactory也就是Spring容器的，单例Bean在此容器内只有一个，Java的单例是基于 JVM，每个 JVM 内只有一个实例。**

### BeanDefinition

在Spring中，Bean本身以及一些其他信息一同被表示成一个BeanDefinition对象，这个对象有以下的一些信息：

- **全限定类名**：通常是Bean的一个指定的具体实现类
- Bean的**行为配置元素**：说明bean在容器中的行为方式（作用域、生命周期回调等等）
- Bean工作需要的**其他Bean的引用**：一般被称作协作者或者**依赖**
- 在新创建的对象中设置的其他配置设置：例如在管理连接池的Bean中限制池的大小或使用的连接的数量。



### Bean的作用域

使用**@Scope注解**来设置Bean的作用域，其中**request、session** 和 **global session** 三种作用域仅在基于web的应用中使用：

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是**单例**的。
- prototype : **每次请求**都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session：全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。

1. singleton：

   **当一个 bean 的作用域为 singleton，那么Spring IoC容器中只会存在一个共享的 bean 实例，并且所有对 bean 的请求，只要 id 与该 bean 定义相匹配，则只会返回bean的同一实例。** singleton 是单例类型(对应于单例模式)，就是在**创建起容器时就同时自动创建了一个bean的对象**，不管你是否使用，但我们可以指定Bean节点的 **`lazy-init=”true”`** 来**延迟初始化**bean，这时候，只有在**第一次获取bean**时才会初始化bean，即第一次请求该bean时才初始化。

2. prototype：

   **当一个bean的作用域为 prototype，表示一个 bean 定义对应多个对象实例。** **prototype 作用域的 bean 会导致在每次对该 bean 请求**（将其注入到另一个 bean 中，或者以程序的方式调用容器的 `getBean() `方法）时都会创建一个新的 bean 实例。prototype 是原型类型，它在我们创建容器的时候并没有实例化，而是当我们**获取bean的时候才会去创建一个对象**，而且我们每次获取到的对象都**不是同一个对象**。根据经验，**对有状态的 bean 应该使用 prototype 作用域，而对无状态的 bean 则应该使用 singleton 作用域。**

**单例Bean的线程安全问题：当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题**

- 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
- 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。



### Bean的生命周期

















## 事务





