# Object类的方法

Object类是Java中所有类的基类。位于java.lang包中，一共有13个方法。

| 方法                    | 简要说明                                                     |
| ----------------------- | ------------------------------------------------------------ |
| Object()                | 默认无参构造方法                                             |
| void registerNatives()  | 注册本地函数                                                 |
| Object clone()          | 创建与该对象的类相同的新对象                                 |
| boolean equals(Object)  | 比较两对象是否相等                                           |
| int hashCode()          | 返回该对象的散列码值                                         |
| String toString()       | 返回该对象的字符串表示                                       |
| Class getClass()        | 返回一个对象运行时的实例类                                   |
| void notify()           | 激活等待在该对象的监视器上的一个线程                         |
| void notifyAll()        | 激活等待在该对象的监视器上的全部线程                         |
| void wait()             | 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待 |
| void wait(long timeout) |                                                              |
| void wait(long, int )   |                                                              |
| void finalize()         | 当垃圾回收器确定不存在对该对象的更多引用时，对象垃圾回收器调用该方法 |

### 1. Object() 即Object的构造方法

Java中规定：在类定义过程中，对于未定义构造函数的类，默认会有一个无参数的构造函数，作为所有类的基类，Object类自然要反映出此特性，在源码中，未给出Object类构造函数定义，但实际上，此构造函数是存在的。

当然，并不是所有的类都是通过此种方式去构建，也自然的，并不是所有的类构造函数都是public。

### 2. registerNatives() 

```java
public class Object {
    private static native void registerNatives();
    static {
        registerNatives();
    }
}
```

**向JVM注册native方法**（本地方法，由JVM实现，底层是C/C++实现的） ，当有程序调用到native方法时，JVM找到再去找这些底层方法进行调用。

### 3. clone()

```java
protected native Object clone() throws CloneNotSupportedException;
```

**此方法返回当前对象的一个副本。**

这是一个protected方法，提供给子类重写。但需要实现Cloneable接口，这是一个标记接口，如果没有实现，当调用object.clone()方法，会抛出CloneNotSupportedException。

**clone的对象是一个新的对象，但是二者的引用都是指向同一个对象，进行是浅拷贝，引用类型还是指向原来的对象。**

### 4. equals()

```Java
public boolean equals(Object obj);
```

用于比较当前对象与目标对象是否相等，**默认是比较引用是否指向同一对象**。为public方法，子类可重写。

```java
public class Object{
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```

为了在比较两个对象时，当它们的某些属性值相同我们就认为它们相同，一般会重写equals()方法。而hashCode方法中有“**相同对象必须有相同哈希值**”的约定，因而也有必要将hashCode()方法也进行重写（例如需要将对象存放至map或set中时）。

**重写equals方法的几条约定：**

1. **自反性：即x.equals(x)返回true，x不为null；**
2. 对称性：即x.equals(y)与y.equals(x）的结果相同，x与y不为null；
3. 传递性：即x.equals(y)结果为true, y.equals(z)结果为true，则x.equals(z)结果也必须为true；
4. 一致性：即x.equals(y)返回true或false，在未更改equals方法使用的参数条件下，多次调用返回的结果也必须一致。x与y不为null。
5. **如果x不为null, x.equals(null)返回false。**



以下是String对equals的重写（对比两个字符串中的所有字符是否都相同）：

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n-- != 0) {
                if (v1[i++] != v2[j++])
                    return false;
            }
            return true;
        }
    }
    return false;
}
```

### 5. hashCode()

```java
public native int hashCode();
```

**将对象的地址值映射为Integer类型的哈希值。**

hashCode方法有几条规范：

1. 一个对象多次调用它的hashCode方法，应当返回相同的integer（哈希值）。
2. 两个对象如果通过**equals**方法判定为相等，那么就应当返回相同integer（哈希值）。
3. 两个地址值不相等的对象调用hashCode方法不要求返回不相等的integer，但是要求拥有两个不相等integer的对象必须是不同对象。**（相同的对象哈希值不一定相等，但是哈希值不相等的两个对象一定不相同）**



String类中的hashCode方法：

```java
public int hashCode() {
    int h = hash;    //Default to 0
    if (h == 0 && value.length > 0) {    //private final char value[]
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

### 6. toString()

```java
public String toString()；
```

建议重写。默认的toString方法，只是将当前类的全限定性类名+@+十六进制的hashCode值。

```java
public class Object{
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
}
```

用于自定义地返回类中的相关信息。

### 7. getClass()  

```java
public final native Class<?> getClass();
```

获取这个对象，包含了对象在运行时的所有方法、属性等信息，即获取这个类型类。

只有对象才能使用这个方法，即**对象.getClass()**或**类.class**。

可以用于反射，在运行时再获取一个对象类型信息并使用。

### 8. **wait() / wait(long) / wait(long, int)**

```java
public final void wait() throws InterruptedException { wait(0);} //实际上是wait(long)
public final native void wait(long timeout) throws InterruptedException;
public final void wait(long timeout, int nanos) throws InterruptedException;
```



这三个方法是用来 **线程间通信用** 的，作用是 **阻塞当前线程** ，等待其他线程调用notify()/notifyAll()方法将其唤醒。

在同步代码块中调用该方法时，当前线程立即释放锁并等待，直到有其他线程调用；当前线程必须是该对象的拥有者。

wait() 方法一直等待，直到获得锁或者被中断。

wait(long timeout) 设定一个超时间隔，如果在规定时间内没有获得锁就返回。

wait(long timeout, int nanos) 此方法类似于一个参数的 wait 方法，但它允许更好地控制在放弃之前等待通知的时间量（timeout为毫秒，nanos为毫微秒，更加精确）。用毫微秒度量的实际时间量可以通过以下公式计算：

```
1000000*timeout+nanos
```

在其他所有方面，此方法执行的操作与带有一个参数的 wait(long) 方法相同。需要特别指出的是，wait(0, 0) 与 wait(0) 相同。



注意事项：

1. 此方法只能在当前线程获取到对象的锁监视器之后才能调用，否则会抛出IllegalMonitorStateException异常。
2. 调用wait方法，线程会将锁监视器进行释放；而Thread.sleep，Thread.yield()并不会释放锁 。
3. wait方法会一直阻塞，直到其他线程调用当前对象的notify()/notifyAll()方法将其唤醒；而wait(long)是等待给定超时时间内（单位毫秒），如果还没有调用notify()/nofiyAll()会自动唤醒；waite(long,int)如果第二个参数大于0并且小于999999，则第一个参数+1作为超时时间；

### 9. notify() / notifyAll()

```java
public final native void notify();     //随机唤醒一个
public final native void notifyAll();  //唤醒所有
```

如果当前线程获得了当前对象锁，调用wait方法，将锁释放并阻塞；

这时另一个线程获取到了此对象锁，并调用此对象的notify() / notifyAll()方法将之前的线程唤醒。

**注意：调用notify()后，阻塞线程被唤醒，可以参与锁的竞争，但可能调用notify()方法的线程还要继续做其他事，锁并未释放，所以我们看到的结果是，无论notify()是在方法一开始调用，还是最后调用，阻塞线程都要等待当前线程结束才能开始。**

### 10. finalize()

（不推荐使用）

```java
protected void finalize() throws Throwable;
```

**此方法是在垃圾回收之前，JVM会调用此方法来清理资源。此方法可能会将对象重新置为可达状态，导致JVM无法进行垃圾回收。**

**finalize()方法具有如下4个特点：**

1. 永远不要主动调用某个对象的finalize()方法，该方法由垃圾回收机制自己调用；
2. finalize()何时被调用，是否被调用具有不确定性；
3. 当JVM执行可恢复对象的finalize()可能会将此对象重新变为可达状态；
4. 当JVM执行finalize()方法时出现异常，垃圾回收机制不会报告异常，程序继续执行。