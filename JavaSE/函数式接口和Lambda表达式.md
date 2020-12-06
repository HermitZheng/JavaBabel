## 函数式接口

Java8最大的变化是引入了函数式思想，也就是说**函数可以作为另一个函数的参数**。

函数式接口，要求接口中**有且仅有一个抽象方法**，因此经常使用的Runnable，Callable接口就是典型的函数式接口。可以使用`@FunctionalInterface`注解，声明一个接口是函数式接口。如果一个接口满足函数式接口的定义，会**默认**转换成函数式接口。但是，最好是使用`@FunctionalInterface`注解**显式声明**。

该注解可用于一个接口的定义上，一旦使用该注解来定义接口，编译器将会**强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。**

  ```java
@FunctionalInterface
public interface MyFunction {
    
    void print(String s);
}
  ```

我们不能使用以下类型的方法来声明一个函数式接口：

- 默认方法
- 静态方法
- 从Object类继承的方法

一个函数式接口可以**重新声明Object类中的方法**，该方法不被视为抽象方法。因此，我们可以声明lambda表达式使用的另一种方法。

```java
package java.util;

@FunctionalInterface
public interface  Comparator<T> {
   // An  abstract method  declared in the functional interface 
   int compare(T o1, T o2);

   // Re-declaration of the equals() method in the Object class 
   boolean equals(Object obj);

   ...
}
```



## Lambda表达式

lambda表达式是函数式编程的核心，lambda表达式即**匿名函数**，是一段没有函数名的函数体，**可以作为参数直接传递给相关的调用者。**

### 基本语法

使用lambda表达式的一般语法是：

```java
(Parameters) -> { Body }		// 使用 -> 分割参数和表达式主体 
```

- **可选类型声明**：不需要声明参数类型（也可以声明），编译器可以统一识别参数值。如果我们选择省略参数类型，我们必须省略**所有**参数的类型。

  ```java
  (int x, int y) -> {x + y};
  (x, y) -> {x + y};
  ```

- **可选的参数圆括号**：一个参数无需定义圆括号，但多个参数需要定义圆括号。对于**没有参数**的lambda表达式，我们仍然需要括号。

  ```java
  x -> {x*x};
  () -> System.out.println("hi");
  ```

- **可选的大括号**：如果主体只包含了一个语句，就不需要使用大括号。

  ```java
  (x, y) -> x + y;
  ```

- **可选的返回关键字**：如果主体只有一个表达式返回值则编译器会自动返回值，如果使用了大括号（可能有多个语句），则需要使用`return`指定明表达式的返回值。

  ```java
  (int x, int y) -> { return x + y; }
  (int x, int y) -> x + y
  ```

- **final修饰符**：可以在参数声明中为lambda表达式使用` final `修饰符。

  ```java
  (final String str) -> str.length();
  ```

**使用Lambda表达式，实际就是创建出该接口的实例对象**。

根据上下文，一个lambda表达式可以映射到不同的**函数接口**类型。编译器会自动推断lambda表达式的类型。

```java
public static void main(String[] args) {  // 创建线程
    // 使用匿名内部类
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("使用匿名内部类！");
        }
    }).run();

    //使用Lambda
    new Thread(() -> System.out.println("使用Lambda表达式！")).run();
}
```

**可以使用break语句退出lambda表达式中的for循环，但是不能跳出到lambda表达式之外的for循环**。

```java
Function<String,String> func = y -> {
      for(int i=0;i<10;i++){
        System.out.println(i);
        if(i == 4){
          break;
        }
      }
```

以下代码以正常方式创建**递归函数**，然后使用递归函数作为**方法引用**来创建lambda表达式。

```java
public class Main {
  public static void main(String[] args) {
    IntFunction<Long> factorialCalc = Main::factorial;
    System.out.println(factorialCalc.apply(10));
  }
  public static long factorial(int n) {
    if (n == 0) {
      return 1;
    } else {
      return n * factorial(n - 1);
    }
  }
}
```



### Lambda行为参数化

可以将lambda表达式作为参数传递给方法。

```java
public class Main {
    
    public static void main(String[] args) {
        /**
         * Lambda表达式 (x, y) -> x + y 实际上就是创建出Calculate函数接口的实例对象，
         * 并作为参数传递给engine方法
         */
        int result = engine((x, y) -> x + y);   // 参数x, y为calculate的参数；方法体x+y为重写
        System.out.println(result);    // 输出 6
    }

    private static int engine(Calculate calculate) {
        int x = 2, y = 4;
        int result = calculate.calculate(x, y);
        return result;
    }
}
@FunctionalInterface
interface Calculate {
    int calculate(int x, int y);
}
```

**`engine `方法的结果取决于传递给它的lambda表达式。**通过其参数更改方法的行为称为行为参数化。

编译器并不总是可以推断lambda表达式的类型。

```java
@FunctionalInterface
interface IntCalculator{
  int calculate(int x, int y);      // 两个同名方法拥有不同的参数类型
}
@FunctionalInterface
interface LongCalculator{           // 这种情况下Lambda表达式会出现歧义
  long calculate(long x, long y);
}
// 解决方法一：指定参数类型以匹配函数接口的要求
engine((long x, long y)-> x * y);
engine((int x,int y)-> x / y);
// 解决方法二：使用cast将表达式转化为所需的函数接口类型
engine((IntCalculator) ((x,y)-> x + y));
// 解决方法三：避免直接使用Lambda表达式作为参数
IntCalculator iCal = (x,y)-> x + y;
engine(iCal);
```



### Java 交叉类型

Java 8引入了一种称为交集类型的新类型。交叉类型是多种类型的交叉。

在两种类型之间使用`Type1 & Type2`，以表示类型1，类型2的交集的新类型。

```java
public class Main {  
  public static void main(String[] args) {
	// 以这种方式，我们使一个lambda表达式可序列化。
    java.io.Serializable ser = (java.io.Serializable & Calculator) (x,y)-> x + y;
  }  
}

@FunctionalInterface
interface Calculator{
  long calculate(long x, long y);
}
```



## 方法引用

lambda表达式表示在函数接口中定义的匿名函数。方法引用**使用现有方法创建lambda表达式**。

方法引用的一般语法是：

```java
Qualifier::MethodName
```

两个连续的冒号充当分隔符。

`MethodName `是方法的名称。`限定符Qualifier`告诉在哪里找到方法引用。

例如，我们可以使用` String::length `从` String `类引用length方法。这里` String `是限定符，` length `是方法名。

有六种类型的方法引用。

- TypeName::staticMethod - 引用类的**静态方法，接口或枚举**
- objectRef::instanceMethod - 引用**实例方法**
- ClassName::instanceMethod - **从类中**引用实例方法
- TypeName.super::instanceMethod - 从对象的**父类型**引用实例方法
- ClassName::new - 引用一个类的**构造函数**
- ArrayTypeName::new - 对指定**数组类型**的构造函数的引用

### 静态方法引用

静态方法引用允许我们使用静态方法作为lambda表达式。静态方法可以在类，接口或枚举中定义。

```java
// Using  a  lambda  expression
Function<Integer, String> func1  = x -> Integer.toBinaryString(x);

// Using  a  method  reference
Function<Integer, String> func2  = Integer::toBinaryString;
```

我们可以在静态方法引用中使用**重载**的静态方法。

```java
// Uses  Integer.valueOf(int)
Function<Integer, Integer> func1  = Integer::valueOf;

// Uses  Integer.valueOf(String)
Function<String, Integer> func2  = Integer::valueOf;

// Uses  Integer.valueOf(String, int)
BiFunction<String, Integer, Integer> func3  = Integer::valueOf;
```

### 实例方法引用

我们可以通过两种方式获得一个实例方法引用，从**对象实例**或从**类名**。

基本上我们有以下两种形式：

- instance::MethodName
- ClassName::MethodName

这里`实例`表示任何对象实例。` ClassName `是的名称类，例如` String `，` Integer `。

`实例`和` ClassName `称为接收器。更具体地说，` instance `被称为有界接收器，而` ClassName `被称为无界接收器。

**绑定实例方法引用**，绑定接收器接收器具有以下形式：

```java
instance::MethodName
Supplier<Integer> supplier = "It is a String".length(); 
```

**未绑定实例方法引用**，未绑定的接收器使用以下语法：

```java
ClassName::instanceMethod
Function<String,  Integer> strLenFunc = String::length; 
int len = strLenFunc.apply("A String"); 
```

**超类型实例方法引用**，关键字` super `仅在实例上下文中使用，引用覆盖的方法。

下面是指对**父类型**的实例方法的方法引用。

```java
ClassName.super::instanceMethod
BiFunction<String,  String,String> strFunc = this::append;   // 省略创建父子类
strFunc = Student.super::append;    // Student extends Person
```



### 构造方法引用

使用构造函数引用的语法是：

```java
ClassName::new
Supplier<String> func = String::new;
```

关键字new指的是类的构造函数。编译器根据上下文选择一个构造函数。

**数组构造函数引用**，我们可以使用数组构造函数创建一个数组如下：

```java
ArrayTypeName::new
IntFunction<int[]> arrayCreator = int[]::new;
```

`int [] :: new `是调用` new int [] `。` new int [] `需要一个` int `类型值作为数组长度，因此` int [] :: new `需要一个` int `类型输入值。



### 通用方法引用

我们可以通过指定实际的类型参数来在方法引用中使用通用方法。

语法如下：

```java
ClassName::methodName
ClassName::new   // 通用构造函数引用的语法
```

```java
public class Main{
  public static void main(String[] argv){
    Function<String[],List<String>> asList = Arrays::<String>asList;
      
    System.out.println(asList.apply(new String[]{"a","b","c"}));
  }
}
```

