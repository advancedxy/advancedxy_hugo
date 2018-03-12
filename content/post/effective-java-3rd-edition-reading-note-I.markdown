---
author: "Ye, Xianjin"
title: "Effective Java 3rd Edition - Reading Note I"
description: "Java's Type System"
date: 2018-03-01T15:28:49+08:00
lastmod: 2018-03-12T23:34:13+08:00
---

# 前言
最近购得 < Effective Java 3rd Edition > 一书, 本文是其读书笔记系列的第一部分, 主要介绍 Java 的类型系统 (Type System) 及书中提供的建议的思考.

# Introduction to Java's Type System
介绍 Java 的类型系统之前, 先介绍两个基本概念.

> 何为类型 (Type), 亦被称之为数据类型 (Data Type)?

根据 [WikiPedia](https://en.wikipedia.org/wiki/Data_type) 的定义, 类型确定了数据的取值范围, 同时定义了数据可进行的操作, 数据的含义以及数据的值是如何被存储的.
也即类型确定了数据 + 操作.

>何为类型系统 (Type System)?

类型系统是类型的申明和应用的规则集合, 通常内置于编程语言进行帮助程序员减少异常程序状态的产生. 不同的编程语言有不同的类型系统实现. 


### Types in Java
Java 的类型由两大类组成: 原始类型(primitive types)和引用类型(reference types).
其中原始类型由如下几个类型组成: boolean, char, byte, short, int, long, float, double.
引用类型为除原始类型之外的其他所有类型, 包括但不限于: T[](Array 为特殊的引用类型), List/Queue/Set 等由 Class/Interface 定义的类型.

### Primitive types
#### Item61: Prefer primitive types to boxed primitives
介绍本 Item 之前, 先介绍一下 Java 的自动装箱技术(Autoboxing). 该技术的存在是为了程序更简洁和易读.

Autoboxing 指的是 Java 编译器对原始类型到对应的引用包装类型的自动转换. 从引用类型到原始类型的自动转换则被称之为 Unboxing.

来看一个简单的例子就知道自动装箱是如何工作的:

``` java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i += 2)
    li.add(i); // translate to li.add(Integer.valueOf(i));
```
下表列出了原始类型和对应的引用包装类型的映射:

| Primitive Type | Wrapper Class |
|----------------|---------------|
| boolean        | Boolean       |
| char           | Character     |
| byte           | Byte          |
| short          | Short         |
| int            | Integer       |
| long           | Long          |
| float          | Float         |
| double         | Double        |


虽然自动装箱技术减少了原始类型和包装类型的区别, 但原始类型和包装类型仍存在实际的区别. 这也就意味这我们在使用时需保持警觉, 并谨慎选择合适的类型.

原始类型和对应的包装类型区别如下:

1. 原始类型有且仅有取值(value)进行标识, 但包装类型除了取值外仍有自己的标识(Identity). 也即存在两个包装类型的实例取值一样但对应不同的实例.
2. 原始类型仅有有效值(functional value), 但包装类型还有一个额外的无效值: null
3. 原始类型更加高效: 时间(相关操作消耗更少 CPU) + 空间(占据更少的内存空间).

下面我们来看几个没有区分原始类型和包装类型的实例:

``` java
// Broken comparator.
Comparator<Integer> naturalOrder = 
    (i, j) -> if (i < j) ? -1 : (i == j ? 0 : 1);
// Apply == to boxed primitives are almost always wrong
```

``` java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42) // unboxing -> NPE
            System.out.println("Unbelievable");
    }
}
```

``` java
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i; // Autoboxing here
    }
    System.out.println(sum);
}
```
总结来讲:

1. 尽可能地使用原始类型
2. 如果不得不选择包装类型(用于泛型), 则需要慎之又慎:
   * 在包装类型中使用了 `==` 基本可以确定是错误的.
   * 原始类型和包装类型混合使用时, 包装类型将会进行拆箱, 拆箱时可能出现 NPE.
   * 原始类型被装箱时, 将会造成额外的对象创建. 该对象创建相对于原始类型, 代价高昂.

### Reference Types
介绍完原始类型后, 我们再看看引用类型. 根据[JLS-SE8](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.3), 引用类型包含4种类型: class, interface, type variables 和 array. 

其中 type variables 由泛型类, 接口, 方法和构造器(Generic class, interface, method, constructor)的类型参数引入, 在上述泛型中标识不完整类型.

数组类型是特殊的引用类型, 由要素类型(element type) + []进行标识: T[], 其中 T 可以为任意类型, 如:

1. 原始类型: int[], byte[]
2. 引用类型: 
   1. Integer[], Long[], Object[]
   2. Collection<?>[] // array of collection of any type
   3. int[][] // element type: int[] which is a reference type
   4. I[] // I: interface type. An element of such an array may have as its value a null reference or an instance of any type that implements the interface.
   5. Abs[] // Abs: abstract class. An element of such an array may have as its value a null reference or an instance of any subclass of the abstract class that is not itself abstract.


类和接口是 Java 中使用最多的类型, 在 < Effective Java 3rd Edition > 提供了下述的 items.
#### Item20: Prefer interfaces to abstract classes
Java 提供了两种多实现方方式: 接口和抽象类. 自 Java8[[JLS 9.4.3]](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.4.3) 起, 接口和抽象类均可以提供默认实现.
接口和抽象类的最大区别为: 抽象类只能由子类来实现, 而接口可以由任何满足接口定义的类来实现. 这也造成了为啥我们应倾向于接口而非抽象类的原因:

1. 已有的类可以很轻松添加新的接口实现: 只需实现新接口定义的方法即可. 而抽象类的子类通常不能添加新的抽象类, 除非将两个抽象类合并. 但这个造成了额外的破坏: 新的抽象类并不一定适合提供给所有的子类.
2. 基于1, 接口更易于定义 mixin. mixin 指的是某个类型在主体实现之外额外提供的行为. 比如, `Comparator` 就是一个 mixin 接口, 提供了原类型之外的比较含义.
3. 接口可以更好地进行组合而非继承: Interface A + Interface B => Interface AB

但接口存在下述限制:

1. Object's 的 `equals` 和 `hashCode` 行为只能进行说明而不能提供实现
2. 接口不能包含实例字段和非公开静态成员(私有静态方法除外)
3. 对于没有控制权的接口, 无法提供默认方法

而对于上述的限制, 一般的做法是实现一个骨架类 + 接口. 具体做法为:

1. 接口对外定义类型
2. 骨架类提供额外的实现以减少子类实现的负担

#### Item 21: Design interfaces for posterity 
在 Java8 之前, 我们无法为接口添加新方法而不破坏已有的实现. 这是因为当接口添加了一个新的方法后, 已有的实现通常会缺少该方法从而在编译期就发生错误(只破坏了 source compatibility).
在 Java8 及之后, 我们可以为接口提供默认实现方法, 但这并不意味着我们可以随意地添加新方法. 往已有的接口中添加新方式仍然充满着危险, 这意味着我们在设计接口时, 需三思而后行.

为什么存在默认方法实现后, 添加新方法仍然是危险的?

1. 我们提供的默认方法并不能满足所有子类的特性要求. 比如说 Java8 的 `Collection` 接口新增加了 `removeIf` 方法, 其实现大致如下:

    ``` java
    // Default method added to the Collection interface in Java 8
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean result = false;
        for (Iterator<E> it = iterator(); it.hasNext(); ) {
            if (filter.test(it.next())) {
                it.remove();
                result = true;
            }
        }
        return result;
    }
    ```
   上述实现大概是最好的通用实现了, 但却不能满足实践中某个特定的类, 比如 `org.apache.commons.collections4.-collection.SynchronizedCollection`: 因为我们的实现中并没有保证同步.
2. 我们提供的默认方法可能不会编译错误或者警告, 但却有可能导致运行时错误.

因此, 除非绝对必要, 向已有接口添加新的方法应被避免. 这也意味着我们在设计接口时就应该慎之又慎.

#### Item 22: Use interfaces only to define types
接口应只用于定义类型. 其中一个反例便是 `constant interface`. `constant interface` 存在的问题如下:

1. 由于接口要求所有的字段为 public, 这意味着这些常数将会泄漏到实现子类的 API 中.
2. 由于1, 我们没法删除不再需要的常数字段(删除将破坏 binary compatibility).

那么如何避免 `constant interface` 带来的问题:

1. 如果常数和类或者接口强相关, 那么应该将常数加入到类或者接口中. 比如, `Integer.MAX_VALUE`
2. 如果常数是某种枚举, 那么应使用枚举类型.
3. 其他的情况下使用不可构造的工具类. 另外, 可以用静态导入(static import)避免工具类的重复书写. 具体示例如下:

    ``` java
    // Constant utility class
    package com.effectivejava.science;

    public class PhysicalConstants {
      private PhysicalConstants() { }  // Prevents instantiation

      public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
      public static final double ELECTRON_MASS    = 9.109_383_56e-31;
    }
    ```

#### Java Generics and type variance
前面提到了 type variables, 这个在 Java 中最多的使用场景便是泛型(Generics). 在介绍< effective java 3rd edition > 中涉及到泛型的 item 之前, 先简单介绍一下 Java 的泛型的概念和类型变形(Type Variance).

> What is Java Generics?

`Java Generics` 通指在 Java 中定义和使用泛型类型(generic type) 和泛型方法(generic method)这一项语言特性. 泛型类型/方法和普通方法最大的区别就是参数类型的使用, 如下述的代码定义了一个泛型类.

``` java
class Box<T> {
    private T value;
    public Box(T t) {
        value = t;
    }
}
```
其中, `Box<T>` 是一个泛型类型, 它使用了参数 T 表示装箱里面的要素类型. 在实际使用中, 我们通常指定参数类型, 比如 `Box<Integer>` 表示是 Integer 的装箱.

这里只对 Java 的泛型做最简单的介绍引入, 更多的详细细节可以参考 [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)

> What about type variance then ?

介绍类型变形(type variance)之前, 我们应该先重新复习一下子类替换原则([Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)): 如果 `S <: T` (S 是 T 的子类), 那么 `T` 实例可以被子类 `S` 实例替换也不影响相关特性.

类型变形则指的是子类替换在复杂情况下(如: 方法返回类型(method return type), 集合类型(collection type))是如何工作的. 由于`Generics` 涉及到了参数类型, 一般也伴随出现了类型变形.

类型变形有三种情况, 具体示例如下表:

| Variance            | Meaning                          | Java Example                           |
|---------------------|----------------------------------|----------------------------------------|
| Covariant(协变)     | S <: T -> C[S] <: C[T]           | Array[String] <: Array[Object]         |
| Contravariant(逆变) | S <: T -> C[T] <: C[S]           | List[? super String] <: List[String]   |
| Invariant(不变)     | S <: T -> C[S], C[T] not related | List[Object], List[String] not related |

更详细地对类型变形进行说明将会超出本文的范围, 后续会考虑用另一篇文章进行详细说明以及 JVM 其他语言(Kotlin, Scala)对比.

#### Item 26: Don’t use raw types

先介绍几个相关的概念:

1. 泛型类型(generic type): 类和接口在定义时, 包含了一个或者多个参数类型, 如 `List<E>`
2. 参数化类型(parameterized type): 类或者接口+\<实际要素类型\>组成了参数化类型, 比如 `List<String>, List<Long>`. 泛型类型定义了对应的参数化类型集合.
3. 裸类型(raw type): 只有泛型类型的类或者接口. 如 `List` 被称之为裸类型. 裸类型的要素类型被擦除(erased), 主要是为了和出现泛型之前的代码兼容.

那么不使用裸类型的理由是? __使用裸类型将会丢失泛型带来的类型安全和表达能力__. 具体示例如下:

``` java
// Raw collection type - don't do this!
// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;

// Erroneous insertion of coin into stamp collection
stamps.add(new Coin( ... )); // Emits "unchecked call" warning

// Raw iterator type - don't do this!
for (Iterator i = stamps.iterator(); i.hasNext(); )
    Stamp stamp = (Stamp) i.next(); // Throws ClassCastException at runtime
        stamp.cancel();
```

那当我们需要表达 `List of any type` 或者 `List of some type` 时, 可以使用 `List<Object> || List<?>`.
必须使用裸类型的两个场景:

1. 类型字面量: `List.class`(`List<String>.class` 非法)
2. 用于 `instanceof` 判断语句时: 由于类型信息在运行时被擦, `someObject instanceof List<String>` 是非法的程序

#### Item 28: Prefer lists to arrays

`Array` 和 `List` 是 Java 中常用的线性数据结构(存储一组数据). 在实际使用时, 应该选择哪种数据结构存储数据呢?

在回答上述问题之前, 先来看下 Java 中的 `Array` 和 `List` 的不同点.

| Type  | Variance  | Reification     |
|-------|-----------|-----------------|
| Array | Covariant | Reifiable       |
| List  | Invariant | `Non-reifiable` |

先来看 Variance. Java 的 `Array` 是 covariant, 意味着 `String[] <: Object[]`. 而 `List` 是 invariant: `List<String>` 和 `List<Object>` 不存在类型替换关系.
由于 `Array` 是可变的, 和 `Array` 的协变特性组合在一起, 便相当于在类型系统中打了个洞, 一些原本在编译器可以检查出来的错误只能在运行时抛出.

具体示例如下:

``` java
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException

// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

再来看 [Reification(具象化)](https://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html)

| Refication      | Meaning                                                               | Example                                                          |
|-----------------|-----------------------------------------------------------------------|------------------------------------------------------------------|
| Reifiable       | type information is fully <br> available at runtime                   | primitives, non-generic types, <br> raw types, unbound wildcards |
| `Non-reifiable` | some type information is removed <br> at compile-time by type erasure | `List<String>` or `List<Integer>`                                |

具象化的区别意味着:

1. `Array` 在运行时拥有元素的类型信息, 可以在运行时进行检查.
2. `List` 由于被类型擦除, 类型检查只能在编译期类型擦除之前进行. 这也是 `List` 是 invariant 的原因.
3. `Array` 和 `Generic Type` 不能很好地共存. 因为 `Generic Type` 的部分类型信息在运行时被擦除, 没法被 `Array` 用来做类型检查.

我们来看个具体的示例: 为啥在涉及到泛型时, 应该更倾向于 `List`

``` java
// A generic chooser
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        // choiceArray = choices.toArray(); // won't compile
        choiceArray = (T[]) choices.toArray(); // has compiler warning
    }
    
    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

``` java
// List-based Chooser - typesafe
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

总结来讲: 混用泛型和数组时, 我们应优先选择`List`. 而在其他情况下, 出于性能和内存占用的考量, 应优先选择数组(比如说 `int[]`).

#### Item 29/30: Favor generic types/methods

当提供出来的 API 要求用户在客户端进行强制转换时(`cast`), 应使用更加安全和易用的泛型类型. 我们在设计新类型时, 应确保用户无需做额外的类型转换, 这通常意味着需要使用泛型.

下面来看一下实例, 如何实现一个 `Generic Stack`.

``` java
// Object-based collection - a prime candidate for generics
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) { 
        // Any type can be pushed, which will fail at runtime casting
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() { // Require cast in client code
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

将其泛型化如下:

```
// Initial attempt to generify Stack - won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // Cannot create generic array
    }

    // The elements array will contain only E instances from push(E).
    // This is sufficient to ensure type safety, but the runtime
    // type of the array won't be E[]; it will always be Object[]!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    
    // Appropriate suppression of unchecked warning
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push requires elements to be of type E, so cast is correct
        @SuppressWarnings("unchecked") E result = (E) elements[--size];

        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    ... // no changes in isEmpty or ensureCapacity
}
```

和泛型类型类似, 方法也可以是泛型的. 接受参数化类型的静态实用(utility)方法通常是泛型的. 
申明泛型方法的方式是: 在方法的装饰器(modifiers)和返回类型之间, 添加参数列表`<E1, E2, E3...>`, 如下述的泛型方法:

``` java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

在定义类型参数时, 有一个比较少见的情况: 类型的边界定义和被定义的类型有关. 这个称之为递归类型边界(`resursive type bound`). 
最常见的一个示例便是`Comparator`, 如下例:

``` java
// Returns max value in a collection - uses recursive type bound
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

泛型方法相对于需要客户端进行显式转换的版本更加安全和易用, 因此优先使用泛型方法来实现通用工具函数.

#### Item31: Use bounded wildcards to increase API flexibility

如前所述, Java 的 `Collection` 是 invariant 的, 这在逻辑上是合理的. 但有时候, 我们需要更加灵活的类型特性. 考虑前述的 `Stack` 类, 需新加一个方法:

``` java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

该方法能够正常工作, 但由于类型不变, 导致了客户端在调用 `pushAll` 时, 传入的参数只能是 `Stack` 的类型参数 `E`, 这限制了 `pushAll` 的作用范围.
相较于 `push` 方法, 客户端可以对 `Stack<Number>` 调用 `push(intVal)`, 但 `pushAll(intVals)` 这个调用由于类型不变(invariant)的限制, 将不能编译通过.

如何解决呢? 出路便是边界化通配类型(`bounded wildcards`). 我们重新定义 `pushAll` 的签名便可解决.
```
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

相对应的, 考虑新增一个 `popAll` 的方法:

``` java
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```
该方法和最开始的 `pushAll` 存在同样的限制, 只能接受和当前 `Stack` 类型参数一致的 `dst`. 更合理的签名是:

``` java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

总结来讲, 为了最大化提升 API 的灵活度, 应考虑合理使用通配符类型参数. 其中最基本的规则是 PECS:

>PECS stands for producer-extends, consumer-super.
