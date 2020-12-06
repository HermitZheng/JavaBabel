# Java__笔面试知识点（集合）

## 概述

集合框架主要由几个重要的接口组成：**Collection，Map和Iterator**。

**List**：**有序**集合，**允许重复**的元素，常用的实现类有ArrayList，LinkedList。
ListIterator是专门用来遍历List的，除了允许 Iterator 接口提供的正常操作外，该迭代器还允许元素插入和替换，以及双向访问。

**Set**：**无序**集合，**不允许重复**的元素，常用的实现类有HashSet，TreeSet(注意：TreeSet是有序的)
Set通过Iterator迭代器进行迭代

**Map**：是由**键映射值**构成的集合，一个映射**不能包含重复的键**；每个键最多只能映射到一个值， key是不能重复的，但是可以是null。常用的实现类：HashMap，TreeMap，Hashtable。

## 集合

### ArrayList

它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

ArrayList 实现了**RandomAccess 接口**， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 ArrayList 中，我们即可以通过**元素的序号**快速获取元素对象，这就是快速随机访问。 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。和 Vector 不同，**ArrayList 中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者 CopyOnWriteArrayList。

ArrayList是一个动态扩容的数组，使用指定的容量或者默认的10容量进行初始化。在插入时，如果容量超出了capacity，则执行**grow()**方法进行扩容（1.5倍）；如果扩容后还是不够，则扩容到minCapacity，也就是存放新数据所需的容量；最大扩容到Integer.MAX_VALUE。扩容后需要将原有的数据通过**`Arrays.copyOf(data, newCapacity)`**进行拷贝。

与扩容相关ArrayList的add方法底层其实都是`System.arraycopy()`来实现的，**该方法是由C/C++来编写的** native方法，效率较高。

由于支持随机访问，ArrayList的查找速度很快。而关于增删操作，如果是在数组的末端进行增删（类似于堆栈），push和pop不涉及**数据移动**操作，速度也是很快的。不适合作为队列，因为一定会涉及到整个数组的移动；但是可以用于实现环形队列，通过两个指针来记录读写的位置。

论遍历ArrayList要比LinkedList快得多，ArrayList遍历最大的优势在于**内存的连续性**，CPU的内部缓存结构会缓存连续的内存片段，可以大幅降低读取内存的性能开销。



### LinkedList

LinkedList底层是**双向链表**，**实现了Deque接口**，因此，我们可以**操作LinkedList像操作队列和栈一样**。

链表要检索任意一个值时，需要进行遍历（根据下标情况，选择从头开始或者从尾开始遍历），不支持随机访问，因而查找较慢。但是再进行增删操作时，可以只修改目标节点前后的指针，就能完成修改，而不需要对其余的数据进行移动，因而增删操作较快。



### HashMap

HashMap继承了**AbstractMap**，实现了**Map接口**。**无序，允许为null，非同步。**

**初始容量16，最大容量为2的31次方，默认装载因子0.75f。**转换链表为红黑树的**阈值为8**（且table_size > 64)，红黑树转回链表的阈值为6。

当桶中的链表要转为红黑树时，**散列表的最小Size为64**。每次**扩容**为原本容量的**两倍**，最大为`Integer.MAX_VALUE`。数组+链表--->散列表。

**loadFactor加载因子**如果太大则容易碰撞，查找效率低；如果太小则存放的数据会很分散，利用率低；且影响到扩容的频率。

**Hash碰撞**的解决方案（拉链法）：

- **java8之前是头插法**：原链表为 A ；插入 B 之后为 B -> A；**在java8之后使用尾插法**，解决了**多线程**环境下扩容死循环的问题
  - 扩容之前为( A -> B -> C)
  - 第一步( A )；( B -> C ) 。第二步( B -> A) ；( C )。
  - 但此时 A 中仍然保存有指向 B 的指针，从而 A 和 B 组成了一个“环形链表”，发生了死循环问题
- 重写**equals**保证当发生hash碰撞时，能够在链表中正确的存储（不发生重复）；同时重写hashCode保证相同的对象hash一定相同
- 链表中元素的个数超出阈值时，链表O(N)重构为红黑树O(logN)，以保证在数据量大的情况下能有较好的查询速度

**线程不安全问题**

- 头插法(jdk1.7)导致的扩容后**死循环**问题
- jdk1.7中扩容时可能发生**数据丢失**（略）
- **数据覆盖**问题：
  - 当put操作插入时，如果**没有发生hash碰撞**，会直接在table中插入元素
  - 两个线程插入两个hash值相同的元素时，都判定没有发生hash碰撞
  - A 线程插入后，轮到B线程的时间片，也直接插入，从而覆盖了A的记录，而不是插入链表，导致A的数据丢失

多线程的场景如何处理：

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合；
- Hashtable：Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null（null键放在table的第一个位置）
- ConcurrentHashMap



### TreeMap

实现了NavigableMap接口（继承自SortedMap接口），TreeMap是**有序**的。TreeMap底层是红黑树，它的方法的时间复杂度都不会太高：**log(n)**。可以使用**Comparator或者Comparable**来比较key是否相等与排序的问题；如果传入的comparator变量为null，则按照**自然顺序**排序。**非同步。**

**key不能为null，为null会抛出NullPointException。**

**Put**

- 如果红黑树为null，则新建红黑树
- comparator规则比较，找到合适的位置插入到红黑树中；如果comparator为**null**，则**使用key作为比较器进行比较（需实现Comparable接口）**
- 创建新节点，找到其父节点的位置，插入并**调整**红黑树

**getEntry**

根据comparator或者key自身进行查找，找到对应的位置。之后据此方法进行get()、remove()、set()

**Iterator**

**TreeMap遍历是使用EntryIterator这个内部类的**，其继承自PrivateEntryIterator，实现了Iterator接口



### Vector 和 SynchronizedList

SynchronizedList 和 Vector 可能会出现的问题：虽然**`size()`和`get()`以及`remove()`都是原子性的**，但是其内部的**`getLast()`和`deleteLast()`是可以交替进行的**，即会导致线程不安全。同时，在**遍历**Vector的时候，有别的线程修改了Vector的**长度**，那还是会**有问题**！

Java推荐使用`for-each`(迭代器)来遍历我们的集合，好处就是**简洁、数组索引的边界值只计算一次**。同时要保证线程安全，必须在循环前加锁。



### Collections.synchronizedMap

在Collections的内部类，SynchronizedMap的内部维护了一个**普通对象Map**，还有**排斥锁mutex**。

SynchronizedMap有两个构造器，如果传入了mutex参数，则将传入的参数赋值给锁；否则将this赋值给锁，即调用SynchronizedMap对象。

之后在操作Map时，所有方法都会上锁`synchronized(mutex) { ... }`





## JUC

### ConcurrentHashMap

存储结构和HashMap相似，也是数组+链表，是线程安全的。检索操作不加锁，即Get方法非阻塞，**Key 和 Value 值都不允许为 null**。

#### 1.7中的存储结构

由Segment(分段)数组`Segment<K,V> extends ReentrantLock`和HashEntry组成，采用了**分段锁**机制，即**理论上最多支持Segment数量（即容量，默认为16）的线程并发访问。**

**Put**

先定位到Segment，再进行put操作：先尝试获取锁，如果有竞争则通过`scanAndLockForPut()` **自旋**获取锁，如果重试次数达到了`MAX_SCAN_RETRIES` 则改为**阻塞锁**获取，保证能获取成功。

**Get**

将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。由于 HashEntry 中的 **value 属性**是用 **volatile** 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。get 方法是非常高效的，**因为整个过程都不需要加锁**。

#### 1.8中的存储结构

抛弃了原有的 Segment 分段锁，而采用了 **`CAS + synchronized`** 来保证并发安全性。**把之前的HashEntry改成了Node**，但是作用不变，把**值和next采用了volatile去修饰**，保证了可见性，并且也引入了**红黑树**，在链表大于一定值的时候会转换（默认是8）。**理论上也是最多支持桶数量的并发访问**，因为在对空桶进行put的时候不需要加syn锁，而是直接使用CAS写入。

**Put**

- 对key进行散列，获取hash值
- 当表为null时，初始化表
- 如果hash值定位到的Node为空，则可以直接写入不需要加锁，利用 **CAS** 尝试写入，失败则**自旋**保证成功。
- 如果当前位置的 `hashcode == MOVED == -1`，则需要进行**扩容**，帮助当前线程扩容。
- 散列冲突，利用 **synchronized 锁**写入数据。：
  - 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

**Get**

get方法是**不用加锁**的，是非阻塞的。Node节点是重写的，设置了**volatile**关键字修饰，致使它每次获取的都是**最新**设置的值。

```java
    volatile V val;
    volatile Node<K,V> next;
```

根据key计算hash值，查找指定位置之后，根据当前位置是头结点、链表、红黑树用不同的方式进行查找。



### CopyOnWriteArrayList

CopyOnWriteArrayList底层就是**数组**，加锁就交由**ReentrantLock**来完成。

```java
	/** 可重入锁对象 */
    final transient ReentrantLock lock = new ReentrantLock();
    /** CopyOnWriteArrayList底层由数组实现，volatile修饰 */
    private transient volatile Object[] array;	
```

**Add、Set**

在添加或修改的时候就上锁`lock.lock()`，并**复制一个新数组，操作在新数组上完成，将array指向到新数组中**，最后解锁`lock.unlock()`。**写加锁，读不加锁**。

**Iterator**

CopyOnWriteArrayList在使用迭代器遍历的时候，操作的都是**原数组**。

- **内存占用**：如果CopyOnWriteArrayList经常要**增删改**里面的数据，经常要执行`add()、set()、remove()`的话，都需要复制一个数组出来，那是比较耗费内存的。
- **数据一致性**：CopyOnWrite容器**只能保证数据的最终一致性，不能保证数据的实时一致性**。



### CopyOnWriteSet

CopyOnWriteArraySet的原理就是CopyOnWriteArrayList。

```java
    private final CopyOnWriteArrayList<E> al;

    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```



## 并发相关

### CAS算法

CAS（Compare And Swap）是乐观锁的一种实现方式，有**3个**操作数：

- **内存值V**
- **旧的预期值A**
- **要修改的新值B**

**当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做**。先**比较**是否相等，如果相等则**替换**。

当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值(**A和内存值V相同时，将内存值V修改为B)**，而其它线程都失败，失败的线程**并不会被挂起**，而是被告知这次竞争中失败，并可以再次尝试**(否则什么都不做)**

使用版本号、时间戳来防止ABA问题。

### Volatile

volatile经典总结：**volatile仅仅用来保证该变量对所有线程的可见性，但不保证原子性**。

- 保证**该变量对所有线程的可见性**
- - 在多线程的环境下：当这个变量修改时，**所有的线程都会知道该变量被修改了**，也就是所谓的“可见性”
- **不保证原子性**
- - 修改变量(赋值)**实质上**是在JVM中**分了好几步**，而**在这几步内(从装载变量到修改)，它是不安全的**。只能保证对**单次读/写**的原子性。i++ 这种操作不能保证原子性。
- 禁止进行指令重排序。（实现**有序性**）

### COW（CopyOnWrite）

如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取**相同的指针指向相同的资源**，直到某个调用者**试图修改**资源的内容时，系统才会**真正复制一份专用副本**（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。**优点**是如果调用者**没有修改该资源，就不会有副本**（private copy）被建立，因此多个调用者只是读取操作时可以**共享同一份资源**。