# Java__笔面试知识点：线程池

**为什么要使用线程池？**使用线程池主要有以下一些原因：

1. **降低资源消耗**。创建/销毁线程需要消耗系统资源，线程池可以**复用已创建的线程**。
2. **控制并发的数量**。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃。（主要原因）
3. **可以对线程做统一管理**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
4. **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。



如果为每一个请求都新开一个线程，会有什么问题？

- **线程生命周期的开销非常高**。每个线程都有自己的生命周期，**创建和销毁线程**所花费的时间和资源可能比处理客户端的任务花费的时间和资源更多，并且还会有某些**空闲线程也会占用资源**。（降低资源消耗）
- 程序的稳定性和健壮性会下降，每个请求开一个线程。如果受到了恶意攻击或者**请求过多**(内存不足)，程序很容易就奔溃掉了。（统一管理、控制并发数量）



## Executor

Executor框架主要由三大部分组成：任务（Runnable、Callable）、任务的执行（Executor）、异步计算的结果（Future）

### 任务（Runnable、Callable）

执行任务需要实现的 **`Runnable` 接口** 或 **`Callable`接口**。**`Runnable` 接口**或 **`Callable` 接口** 实现类都可以被 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行。

#### Callable

Callable就是Runnable的扩展，**Runnable没有返回值，不能抛出受检查的异常，而Callable可以！**因此当我们希望任务有返回值的时候，可以使用Callable。

```java
public interface Runnable {
    public abstract void run();
}

public interface Callable<V> {
    V call() throws Exception;
}
```



### 任务的执行(`Executor`)

如下图所示，包括任务执行机制的核心接口 **`Executor`** ，以及继承自 `Executor` 接口的 **`ExecutorService` 接口。`ThreadPoolExecutor`** 和 **`ScheduledThreadPoolExecutor`** 这两个关键类实现了 **ExecutorService 接口**。



### 异步计算的结果(`Future`)

`Future` 接口以及 `Future` 接口的实现类 `FutureTask` 类都可以代表异步计算的结果。当我们把 **`Runnable`接口** 或 **`Callable` 接口** 的实现类提交给 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行之后，调用 `submit()` 方法时会返回一个 **`FutureTask`** 对象。

![image-20200812163958083](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200812163958083.png)

Future就是对于具体的Runnable或者Callable任务的执行结果进行**取消**、**查询是否完成**、**获取结果**。必要时可以通过**`get()`**方法获取执行结果，该方法会**阻塞直到任务返回结果（可以设定等待时间，如果超过则TimeoutException）。**

**`execute()` vs `submit()`**：

1. **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
2. **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。



### 基本使用流程

1. 创建实现`Runnable`或者`Callable`接口的任务对象
2. 使用`ExecutorService.execute()`方法将任务对象交给`ExecutorService`执行
3. 或者使用`ExecutroService.submit()`方法提交任务对象，返回一个实现`Future`接口的对象
4. 可以执行`Future.get()`来等待任务执行完成，以及返回结果；或者通过`Future.cancel()`来取消任务的执行



## ThreadPoolExecutor

Java中的线程池顶层接口是`Executor`接口，`ThreadPoolExecutor`是这个接口的实现类，也是最常用的线程池。

<img src="C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200812160453345.png" alt="image-20200812160453345" style="zoom:80%;" />

### 构造方法参数

五个必须参数：

- **int corePoolSize**：该线程池中**核心线程数最大值**

  > 核心线程：线程池中有两类线程，核心线程和非核心线程。核心线程默认情况下会一直存在于线程池中，即使这个核心线程什么都不干（铁饭碗），而非核心线程如果长时间的闲置，就会被销毁（临时工）。

- **int maximumPoolSize**：该线程池中**线程总数最大值** 。

  > 该值等于核心线程数量 + 非核心线程数量。

- **long keepAliveTime**：**非核心线程闲置超时时长**。

  > 非核心线程如果处于闲置状态超过该值，就会被销毁。如果设置allowCoreThreadTimeOut(true)，则会也作用于核心线程。

- **TimeUnit unit**：keepAliveTime的单位。
  
- TimeUnit是一个枚举类型 ，包括以下属性：
  
  > NANOSECONDS ： 1微毫秒 = 1微秒 / 1000 ；MICROSECONDS ： 1微秒 = 1毫秒 / 1000 ；MILLISECONDS ： 1毫秒 = 1秒 /1000 ；SECONDS ： 秒 MINUTES ： 分 ；HOURS ： 小时 ；DAYS ： 天
  
- **BlockingQueue workQueue**：**阻塞队列**，维护着**等待执行的Runnable任务对象**。当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

#### **常用的几个阻塞队列**：

  1. **LinkedBlockingQueue**

     链式阻塞队列，底层数据结构是**链表**，默认大小是`Integer.MAX_VALUE`，也可以指定大小。有界队列，但是当默认容量MAX时相当于是无界队列。

     

  2. **ArrayBlockingQueue**

     数组阻塞队列，是**有界队列**，底层数据结构是**数组**，需要指定队列的大小，且一旦初始化不能改变。构造方法中的fair表示控制对象的内部锁是否采用公平锁，默认是**非公平锁**。

     

  3. **SynchronousQueue**

     同步队列，**内部容量为0**，**每个put操作必须等待一个take操作**，反之亦然。

     一些方法的返回：

     - iterator() 永远返回空，因为里面没有东西
     - peek() 永远返回null
     - put() 往queue放进去一个element以后就一直wait直到有其他thread进来把这个element取走。
     - offer() 往queue里放一个element后**立即返回**，如果碰巧这个element被另一个thread取走了，offer方法返回true，认为offer成功；否则返回false。
     - take() 取出并且remove掉queue里的element，取不到东西他会一直等。
     - poll() 取出并且remove掉queue里的element，只有到**碰巧另外一个线程正在往queue里offer数据或者put数据的时候，该方法才会取到东西。否则立即返回null。**
     - isEmpty() 永远返回true
     - remove()&removeAll() 永远返回false

     

  4. **DelayQueue**

     延迟队列，该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。
     
     
     
  5. **PriorityBlockingQueue**

     基于优先级的**无界**阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），内部控制线程同步的锁采用的是公平锁。

     **PriorityBlockingQueue**不会阻塞数据生产者（因为队列是无界的），而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，**生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。**对于使用**默认大小**的**LinkedBlockingQueue**也是一样的。



**阻塞队列的实现原理：**

构造器除了初始化队列的大小和是否是公平锁之外，还对同一个锁（lock）初始化了两个**监视器Condition**，分别是**notEmpty**和**notFull**。这两个监视器的作用目前可以简单理解为标记分组，**当该线程是put操作时，给他加上监视器notFull，标记这个线程是一个生产者；当线程是take操作时，给他加上监视器notEmpty，标记这个线程是消费者。**

**Put**

```java
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            // 如果队列满了则阻塞该线程，同时将其标记为notFull（生产者）
            // 等待唤醒，唤醒之后继续while判断队列是否已满
            notFull.await();
        // 入队，在入队结束后使用notEmpty.signal()唤醒消费者（notEmpty）线程
        enqueue(e);
    } finally {
        lock.unlock();
    }
```

**Take**

```java
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            // 队列为空则阻塞并标记为notFull（消费者）
            notEmpty.await();
        // 出队中使用notFull.signal()唤醒生产者（notFull）线程
        return dequeue();
    } finally {
        lock.unlock();
    }
```





两个非必须参数：

- **ThreadFactory threadFactory**

  **创建线程的工厂** ，用于批量创建线程，统一在创建线程时设置一些参数，如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂。

- **RejectedExecutionHandler handler**

  **拒绝处理策略**，线程数量大于最大线程数就会采用拒绝处理策略，四种拒绝处理的策略为 ：

  1. **ThreadPoolExecutor.AbortPolicy**：**默认拒绝处理策略**，丢弃任务并抛出RejectedExecutionException异常。
  2. **ThreadPoolExecutor.DiscardPolicy**：丢弃新来的任务，但是不抛出异常。
  3. **ThreadPoolExecutor.DiscardOldestPolicy**：丢弃**队列头部（最旧的）**的任务，然后重新尝试执行程序（如果再次失败，重复此过程）。
  4. **ThreadPoolExecutor.CallerRunsPolicy**：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，**如果执行程序已关闭，则会丢弃该任务**。因此这种策略会**降低对于新任务提交速度**，影响程序的整体性能。如果应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。



### 线程池的状态

变量ctl定义为**AtomicInteger**，**记录了“线程池中的任务数量”和“线程池的状态”两个信息**：

- **RUNNING**：线程池**能够接受新任务**，以及对新添加的任务进行处理。
- **SHUTDOWN**：线程池**不可以接受新任务**，但是可以对已添加的任务进行处理。
- **STOP**：线程池**不接收新任务，不处理已添加的任务，并且会中断正在处理的任务**。
- **TIDYING**：当**所有的任务已终止**，ctl记录的"任务数量"为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
- **TERMINATED**：线程池**彻底终止的状态**。

**状态的转换**：

- 线程池创建后处于**RUNNING**状态。
- 调用`shutdown()`方法后处于**SHUTDOWN**状态，线程池不能接受新的任务，清除一些空闲worker,会**等待阻塞队列的任务完成**。
- 调用`shutdownNow()`方法后处于**STOP**状态，线程池不能接受新的任务，**中断所有线程**，阻塞队列中没有被执行的任务**全部丢弃**。此时，poolsize=0,阻塞队列的size也为0。
- 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为**TIDYING**状态。接着会执行`terminated()`函数。
- 线程池处在**TIDYING**状态时，**执行完terminated()方法之后**，就会由 **TIDYING 转变为 TERMINATED**， 线程池被设置为**TERMINATED**状态。



### 对任务的处理流程

1. 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量快速达到corePoolSize，在核心线程数量 < corePoolSize时）。**注意，这一步（新建线程`addWorker()方法中`）需要获得全局锁（ReentrantLock）。**
2. 线程总数量 >= corePoolSize时，新来的线程任务会进入**任务队列**中等待，然后空闲的核心线程会依次去缓存队列中取任务来执行（体现了**线程复用**）。
3. 当缓存队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建**非核心线程**去执行这个任务。**注意，这一步需要获得全局锁。**
4. 缓存队列满了， 且总线程数达到了maximumPoolSize，则会采取上面提到的**拒绝策略**进行处理。

![image-20200812223406463](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200812223406463.png)



### Worker线程复用

ThreadPoolExecutor在创建线程时，会将线程封装成**工作线程worker**,并放入**工作线程组**中，然后这个worker反复从阻塞队列中拿任务去执行。

`Worker`类实现了`Runnable`接口，所以`Worker`也是一个线程任务。在`Worker`的构造方法中，创建了一个线程，线程的任务就是自己。故`addWorker`方法中调用了`t.start`，会触发`Worker`类的`run`方法被JVM调用。

首先去执行创建这个worker时就有的任务，当执行完这个任务后，worker的生命周期并没有结束，在`while`循环中，worker会不断地调用`getTask`方法从**阻塞队列**中获取任务然后调用`task.run()`执行任务,从而达到**复用线程**的目的。只要`getTask`方法不返回`null`,此线程就不会退出。

当然，核心线程池中创建的线程想要拿到阻塞队列中的任务，先要判断线程池的状态，如果**STOP**或者**TERMINATED**，返回`null`。

**核心线程**的会一直卡在`workQueue.take`方法，被**阻塞并挂起**，不会占用CPU资源，直到拿到`Runnable` 然后返回（当然如果**allowCoreThreadTimeOut**设置为`true`,那么核心线程就会去调用`poll`方法，因为`poll`可能会返回`null`，所以这时候核心线程满足超时条件也会被销毁）。

**非核心线程**会`workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS)` ，如果超时还没有拿到，下一次循环判断**compareAndDecrementWorkerCount**就会返回`null`,Worker对象的`run()`方法循环体的判断为`null`，任务结束，然后线程被系统回收 。



## 默认实现的线程池

### FixedThreadPool

`FixedThreadPool` 被称为可重用固定线程数的线程池。核心线程数量和总线程数量相等，都是传入的参数nThreads，所以只能创建核心线程，不能创建非核心线程。因为LinkedBlockingQueue的默认大小是Integer.MAX_VALUE，故如果核心线程空闲，则交给核心线程处理；如果核心线程不空闲，则入列等待，直到核心线程空闲。

```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {	// 传递线程数和工厂
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),	// 无界队列，容量为MAX
                                      threadFactory);
    }
```

**`FixedThreadPool` 使用无界队列 `LinkedBlockingQueue`（队列的容量为 Intger.MAX_VALUE）作为线程池的工作队列会对线程池带来如下影响 ：**

1. 当线程池中的线程数达到 `corePoolSize` 后，新任务将在无界队列中等待，因此**线程池中的线程数不会超过 corePoolSize**；
2. 由于使用无界队列时 `maximumPoolSize` 将是一个无效参数，因为不可能存在任务队列满的情况。所以，通过创建 `FixedThreadPool`的源码可以看出创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 被设置为同一个值。
3. 由于 1 和 2，使用无界队列时 `keepAliveTime` 将是一个无效参数；
4. 运行中的 `FixedThreadPool`（未执行 `shutdown()`或 `shutdownNow()`）**不会拒绝任务**，**在任务比较多的时候会导致 OOM**（内存溢出）。



### CachedThreadPool

`CachedThreadPool` 是一个会根据需要创建新线程的线程池。对于新的任务，如果此时线程池里没有空闲线程，**线程池会毫不犹豫的创建一条新的线程去处理这个任务**。`CachedThreadPool` 的`corePoolSize` 被设置为空（0），**`maximumPoolSize`被设置为 Integer.MAX.VALUE**，即它是无界的，这也就意味着如果主线程提交任务的速度高于 `maximumPool` 中线程处理任务的速度时，`CachedThreadPool` 会**不断创建新的线程**。极端情况下，**大量创建线程会导致OOM**。

```java
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
```

`CacheThreadPool`的**运行流程**如下：

1. 首先执行 `SynchronousQueue.offer(Runnable task)` 提交任务到任务队列。如果当前 `maximumPool` 中有闲线程正在执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`，那么**主线程执行 offer 操作与空闲线程执行的 `poll` 操作配对成功**，主线程把任务交给空闲线程执行，`execute()`方法执行完成，否则执行下面的步骤 2；
2. 当初始 `maximumPool` 为空，或者 `maximumPool` 中没有空闲线程时，将没有线程执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`。这种情况下，步骤 1 将失败，此时 `CachedThreadPool` 会**创建新线程执行任务**，execute 方法执行完成；

当需要执行很多**短时间**的任务时，CacheThreadPool的线程复用率比较高， 会显著的**提高性能**。而且线程60s后会回收，意味着即使没有任务进来，CacheThreadPool并不会占用很多资源。



### SingleThreadExecutor

**使用单个worker线程的Executor**。使用了LinkedBlockingQueue（Integer.MAX_VALUE），所以，**不会创建非核心线程**。所有任务按照**先来先执行**的顺序执行。如果这个唯一的线程不空闲，那么新来的任务会存储在任务队列里等待执行。

```java
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```



### ScheduledThreadPoolExecutor

**`ScheduledThreadPoolExecutor` 主要用来在给定的延迟后运行任务，或者定期执行任务。**



### 使用构造函数创建！

**《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 构造函数的方式**，为什么？

Executors 返回线程池对象的弊端如下：

- **`FixedThreadPool` 和 `SingleThreadExecutor`** ： 允许请求的队列长度为 Integer.MAX_VALUE，可能**堆积大量的请求**，从而导致 OOM。
- **`CachedThreadPool `和 `ScheduledThreadPool`** ： 允许创建的线程数量为 Integer.MAX_VALUE，可能会**创建大量线程**，从而导致 OOM。