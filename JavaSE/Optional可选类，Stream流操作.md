# Optional可选类，Stream流操作

## Optional类

在Java中经常会遇到NullPointerException，而为了避免出现空指针异常，我们经常会写这样的语句：

```java
User user = getUserById(id);
if (user != null) {     // 避免NPE
    String username = user.getUsername();
    System.out.println("Username is: " + username);
}
```

Java 8引入了一个java.util.Optional类来优雅地处理NullPointerException。

Optional是一个**可以包含或不包含非空值**的容器。**可能返回null的方法应返回Optional，而不是null。**

![image-20200318103446607](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200318103446607.png)

JDK 提供三个**静态方法**来构造一个 `Optional`：

1. `Optional.of(T value)`，该方法通过一个非 `null` 的 *value* 来构造一个 `Optional`，返回的 `Optional` 包含了 *value* 这个值。对于该方法，传入的参数**一定不能为 `null`**，否则便会抛出 `NullPointerException`。

```java
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);    // value不能为null
}
```



2. `Optional.ofNullable(T value)`，该方法和 `of` 方法的区别在于，传入的参数**可以为 `null`** —— 但是前面 javadoc 不是说 `Optional` 只能包含非 `null` 值吗？我们可以看看 `ofNullable` 方法的源码：

```java
public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);   // 当value为null，返回Optional.empty()
}	 
```



3. `Optional.empty()`，该方法用来构造一个空的 `Optional`，即该 `Optional` 中不包含值 —— 其实底层实现还是 **如果 `Optional` 中的 *value* 为 `null` 则该 `Optional` 为不包含值的状态**，然后在 API 层面将 `Optional` 表现的不能包含 `null` 值，使得 `Optional` 只存在 **包含值** 和 **不包含值** 两种状态。

```java
public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
private final T value;
private static final Optional<?> EMPTY = new Optional<>();   
private Optional() {this.value = null;}   // 当empty()时，返回一个空的Optional
```



- 如果值存在则 `isPresent()`方法会返回 `true`，调用 `get()` 方法会返回该对象。

```java
public boolean isPresent() {
    return value != null;
}
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

- 如果值不存在，即在一个`Optional.empty` 上调用 `get()` 方法的话，将会抛出 `NoSuchElementException` 异常。



```java
Optional<User> user = Optional.ofNullable(getUserById(id));
if (user.isPresent()) {     // 只是这样使用的话依旧复杂
    String username = user.get().getUsername();
    System.out.println("Username is: " + username); 
}
```

###  Optional 提供的一些方法

1. **ifPresent()**：如果 `Optional` 中有值，则对该值调用 `consumer.accept`，否则**什么也不做**。

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
// 修改上面的例子
Optional<User> user = Optional.ofNullable(getUserById(id));
user.ifPresent(u -> System.out.println("Username is: " + u.getUsername()));
```

2. **orElse()**：如果 `Optional` 中有值则将其返回，否则返回 `orElse` 方法传入的参数。

```java
public T orElse(T other) {
    return value != null ? value : other;
}
User user = Optional
        .ofNullable(getUserById(id))
        .orElse(new User(0, "Unknown"));
System.out.println("Username is: " + user.getUsername());
```

3. **orElseGet()**：`orElseGet` 与 `orElse` 方法的区别在于，`orElseGet` 方法**传入的参数为一个 `Supplier `接口的实现** —— 当 `Optional` 中有值的时候，返回值；当 `Optional` 中没有值的时候，返回从该 `Supplier` 获得的值other.get()。

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
User user = Optional
        .ofNullable(getUserById(id))
        .orElseGet(() -> new User(0, "Unknown"));
System.out.println("Username is: " + user.getUsername());
```

4. **orElseThrow()**：`orElseThrow` 与 `orElse` 方法的区别在于，`orElseThrow` 方法当 `Optional` 中有值的时候，返回值；没有值的时候会**抛出异常**，抛出的异常由传入的 *exceptionSupplier* 提供。

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
User user = Optional
        .ofNullable(getUserById(id))
        .orElseThrow(() -> new EntityNotFoundException("id 为 " + id + " 的用户没有找到"));
```

例如：在 SpringMVC 的控制器中，我们可以**配置统一处理各种异常**。查询某个实体时，如果数据库中有对应的记录便返回该记录，否则就可以抛出 `EntityNotFoundException` ，处理 `EntityNotFoundException` 的方法中我们就给客户端返回Http 状态码 404 和异常对应的信息

5. **map()**：如果当前 `Optional` 为 `Optional.empty`，则依旧返回 `Optional.empty`；否则返回一个新的 `Optional`，该 `Optional` 包含的是：**函数 *mapper* 在以 *value* 作为输入时的输出值**。

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
Optional<String> username = Optional.ofNullable(getUserById(id))
                        .map(user -> user.getUsername())
                        .map(name -> name.toLowerCase())
                        .map(name -> name.replace('_', ' '))   // 可以进行多次map操作
                        .orElse("Unknown")
                        .ifPresent(name -> System.out.println("Username is: " + name));
```

6. **flatMap()**：`flatMap` 方法与 `map` 方法的区别在于，`map` 方法参数中的函数 `mapper` 输出的是值，然后 `map` 方法会使用 `Optional.ofNullable` 将其包装为 `Optional`；而 `flatMap` 要求**参数中的函数 `mapper` 输出的就是 `Optional`**。

```java
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
Optional<String> username = Optional.ofNullable(getUserById(id))
                        .flatMap(user -> Optional.of(user.getUsername()))
                        .flatMap(name -> Optional.of(name.toLowerCase()))
                        .orElse("Unknown")
                        .ifPresent(name -> System.out.println("Username is: " + name));
```

7. **filter()**：`filter` 方法**接受一个 `Predicate` 来对 `Optional` 中包含的值进行过滤**，如果包含的值满足条件，那么还是返回这个 `Optional`；否则返回 `Optional.empty`。

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}
Optional<String> username = Optional.ofNullable(getUserById(id))
                        .filter(user -> user.getId() < 10)
                        .map(user -> user.getUsername());
                    	.orElse("Unknown")
                        .ifPresent(name -> System.out.println("Username is: " + name));
```



## Stream流

Java8 Stream 使用的是函数式编程模式，如同它的名字一样，它可以被用来对**集合**进行**链状流式的操作**。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
    int sum = 0; 
    for (int n : numbers) {  // 在外部迭代中，我们为每个循环使用for，或者为序列中的集合和过程元									素获取迭代器。
      if (n % 2 == 1) {
        int square = n * n;
        sum = sum + square;
      }
    }
```

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()   // 使用stream流重写，从内部执行循环，效果完全一样
        .filter(n -> n % 2  == 1)    	// 过滤
        .map(n  -> n * n)				// 映射
        .reduce(0, Integer::sum);		// 求和
```

外部迭代通常意味着顺序代码，顺序代码只能由一个线程执行。流被设计为**并行处理**元素。

```java
int sum = numbers.parallelStream()  // 使用parallelStream进行并行流处理
        .filter(n -> n % 2  == 1)
        .map(n  -> n * n)
        .reduce(0, Integer::sum);
```

**中间操作**（如filter）会再次返回一个流，所以我们可以链接多个中间操作；**终端操作**是对流操作的一个结束动作，一般返回 `void` 或者一个**非流**的结果，如上面的reduce返回求和值。