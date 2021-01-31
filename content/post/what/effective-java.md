+++
title = "Effective Java"
description = "3rd Edition, 12 chapters, 90 items"
tags = [
    "java"
]
date = "2020-12-24"
topics = [
    "java"
]
toc=true
+++

90个条目，每天读一篇，3个月完成。从今天的第8条开始。

找到一位[法国人](https://github.com/Ahdak/blog)学习[Effective-Java的笔记](https://ahdak.github.io/blog/tags/#Effective-Java)

## 1 Creating and Destroying Objects

### Item 08: Avoid finalizers and cleaners 

参考[A Guide to the finalize Method in Java](baeldung.com/java-finalize)

- `finalize()`方法调用 Finalizer，属于Object类的方法
- 不要在此方法中执行耗时动作或用来确保“状态” 的操作
- 不能确保被正确地执行（给予GC算法实现，在不同的JVM下有不同的实现）
- 即使手动调用`System.gc`和`System.runFinalization`也无法确保finalize方法被调用
- finalize中的执行如果有异常会被忽略，不会有任何打印
- 比cleaner更耗时('AutoCloseable'接口的`close`方法)
- 还可能被利用进行攻击（不同理解原理）
- 如果无法正常回收，容器出现OOM问题。参考上面链接中的测试代码

有这么多缺点，为啥还存在？仅作为“安全网”(safety net)用来“兜底”，例如 
- `FileInputStream`
- `FileOutputStream`
- `ThreadPoolExecutor`
- ` java.sql.Connection` 

或者针对Native层的对象（how？）

```

import java.lang.ref.ReferenceQueue;
import java.lang.reflect.Field;

public class CrashedFinalizable {

    public static void main(String[] args) throws ReflectiveOperationException {
        for (int i = 0; ; i++) {
            new CrashedFinalizable();
            // other code
            if ((i % 1_000_000) == 0) {
                Class<?> finalizerClass = Class.forName("java.lang.ref.Finalizer");
                Field queueStaticField = finalizerClass.getDeclaredField("queue");
                queueStaticField.setAccessible(true);
                ReferenceQueue<Object> referenceQueue = (ReferenceQueue) queueStaticField.get(null);

                Field queueLengthField = ReferenceQueue.class.getDeclaredField("queueLength");
                queueLengthField.setAccessible(true);
                long queueLength = (long) queueLengthField.get(referenceQueue);
                System.out.format("There are %d references in the queue%n", queueLength);
            }
        }
    }

    @Override
    protected void finalize() {
        System.out.print("");
    }
}
```

本地输出——

```
There are 0 references in the queue
There are 0 references in the queue
There are 0 references in the queue
There are 102719 references in the queue
There are 0 references in the queue
There are 464705 references in the queue
There are 0 references in the queue
There are 0 references in the queue
There are 0 references in the queue
There are 0 references in the queue
There are 723325 references in the queue
There are 1505566 references in the queue
There are 2244080 references in the queue
```

![](https://upload-images.jianshu.io/upload_images/3296949-72130351f7ab647c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/3296949-584408e67e6e1b98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Item 09: Prefer try-with-resources to try-finally

今天这个item只有两页，说的内容也简单

在Java 7 中引入的 [14.20.3. try-with-resources](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20.3)，通常会被简写为 `JSL 14.20.3` 

JSL: [Java Language Specification](https://docs.oracle.com/javase/specs/jls/se7/html/index.html)

>A try-with-resources statement is parameterized with variables (known as resources) that are initialized before execution of the try block and closed automatically, in the reverse order from which they were initialized, after execution of the try block. catch clauses and a finally clause are often unnecessary when resources are closed automatically.

- 不再需要手动进行关闭操作。会自动进行“倒序”关闭。所以catch和finally语句都不是必须的了

```
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
        return br.readLine();
    }
}
```

- 使用try-with-resources，不要使用try-finally。后者看起来更难看，而且抛出的异常可能不准确（因为try语句快和finally语句块中都可能抛出异常）。前者是目前的最佳实践

## 2 Methods Common to All Objects

### Item 10: Obey the general contract when overriding `equals`

不重写——  

- 最简单的方法是不重新此方法
- 如果类的每个实例都是唯一的时也不需要，例如每个`Thread`实例
- 或者说不需要进行“逻辑相等”（logical equality）的测试，例如`java.util.regex.Pattern`的类实例也不需要重新`equals`方法
- 父类已经重新了此方法并且适用子类的场景，例如大部分`Set`类的实现继承自`AbstractSet`， `List`继承自`AbstractList`
- 类是私有或包内私有并且equals方法不会被调用

要重写——

- 值类型的类，例如`Integer`,`String`，或者用户启动两个实例"逻辑地相等"(logically equivalent)，而不是指“同一个对象”

所谓`equals`的“一般格式/通用合同”(general contract)包括——

- 反射性(Reflexive)， 对于非null引用， x.equals(x)总为true
- 对称性(Symmetric)，如果 x.equals(y)为真，那么y.equals(x)必为true
- 传递性(Transitive)， 如果x等于y，y等于z，那么x也等于z
- 一致性(Consistent)，无论调用多少次，结果都一样
- ~~分空性~~(Non-nullity)任何非null引用都不等于null， x.equals(null)结果false


>To paraphrase John Donne, no class is an island. 
[meaning](https://www.phrases.org.uk/meanings/no-man-is-an-island.html)  human beings do badly when isolated from others and need to be part of a community in order to thrive.

传递性很容易被违反。尤其是实例化一个类并且增加了一个组件的情况下
> There is no way to extend an instantiable class and add a value component while preserving the equals contract

The [Liskov substitution principle](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle) says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes.

JDK中的 `java.util.Timestamp`扩展了`java.util.Date`并且增加了一个组件`nanoseconds`，Timestamp的`equals`方法就违背了对称性原则。

> The Timestamp.equals(Object) method never returns true when passed an object that isn't an instance of java.sql.Timestamp, because the nanos component of a date is unknown. As a result, the Timestamp.equals(Object) method is not symmetric with respect to the java.util.Date.equals(Object) method.

equals的实现不需要先判断nul，根据[JLS, 15.20.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.20.2)定义，null进行`instanceof`调用时始终返回false

`equals`方法实现——

- 使用`==`检查是否是同一类引用；(考虑性能情况下的简化操作)
- 使用`instanceof`判断
- 进行类型转换cast 
- 对关键field进行对比
- 同时重写 `hashCode`方法
...

最后灵魂三性问：是否满足对称性、传递性、一致性？

建议使用[google's AutoValue](https://github.com/google/auto/tree/master/value)，[使用方式](https://github.com/google/auto/blob/master/value/userguide/index.md)， [Introduction to AutoValue](https://www.baeldung.com/introduction-to-autovalue)

### Item 10: Obey the general contract when overriding `equals`

不重写——  

- 最简单的方法是不重新此方法
- 如果类的每个实例都是唯一的时也不需要，例如每个`Thread`实例
- 或者说不需要进行“逻辑相等”（logical equality）的测试，例如`java.util.regex.Pattern`的类实例也不需要重新`equals`方法
- 父类已经重新了此方法并且适用子类的场景，例如大部分`Set`类的实现继承自`AbstractSet`， `List`继承自`AbstractList`
- 类是私有或包内私有并且equals方法不会被调用

要重写——

- 值类型的类，例如`Integer`,`String`，或者用户启动两个实例"逻辑地相等"(logically equivalent)，而不是指“同一个对象”

所谓`equals`的“一般格式/通用合同”(general contract)包括——

- 反射性(Reflexive)， 对于非null引用， x.equals(x)总为true
- 对称性(Symmetric)，如果 x.equals(y)为真，那么y.equals(x)必为true
- 传递性(Transitive)， 如果x等于y，y等于z，那么x也等于z
- 一致性(Consistent)，无论调用多少次，结果都一样
- ~~分空性~~(Non-nullity)任何非null引用都不等于null， x.equals(null)结果false


>To paraphrase John Donne, no class is an island. 
[meaning](https://www.phrases.org.uk/meanings/no-man-is-an-island.html)  human beings do badly when isolated from others and need to be part of a community in order to thrive.

传递性很容易被违反。尤其是实例化一个类并且增加了一个组件的情况下
> There is no way to extend an instantiable class and add a value component while preserving the equals contract

The [Liskov substitution principle](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle) says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes.

JDK中的 `java.util.Timestamp`扩展了`java.util.Date`并且增加了一个组件`nanoseconds`，Timestamp的`equals`方法就违背了对称性原则。

> The Timestamp.equals(Object) method never returns true when passed an object that isn't an instance of java.sql.Timestamp, because the nanos component of a date is unknown. As a result, the Timestamp.equals(Object) method is not symmetric with respect to the java.util.Date.equals(Object) method.

equals的实现不需要先判断nul，根据[JLS, 15.20.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.20.2)定义，null进行`instanceof`调用时始终返回false

`equals`方法实现——

- 使用`==`检查是否是同一类引用；(考虑性能情况下的简化操作)
- 使用`instanceof`判断
- 进行类型转换cast 
- 对关键field进行对比
- 同时重写 `hashCode`方法
...

最后灵魂三性问：是否满足对称性、传递性、一致性？

建议使用[google's AutoValue](https://github.com/google/auto/tree/master/value)，[使用方式](https://github.com/google/auto/blob/master/value/userguide/index.md)， [Introduction to AutoValue](https://www.baeldung.com/introduction-to-autovalue)


### Item 11: Always override `hashCode` when you override `equals`

- 重写了`equals`方法时必须也要重写`hashCode`方法
- 需要满足一致性
- 如果两个对象的equals，调用hashCode也不许返回相同的值
- 如果两个对象不equals，不强调调用各自的hashcode方法时也必须不同——所以“极限”操作，hashCode总返回42

一般做法——：  
- 定义一个result值；可以进行lazy初始化
- 计算关键属性的hash值：
- 返回result值
- 非关键属性可以忽略

需要考虑性能影响；懒初始化；使用“奇质数”`odd prime`

```
// hashCode method with lazily initialized cached hash code
private int hashCode; // Automatically initialized to 0
@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```
可以利用IDE的自动处理功能：

- 选择一个模板，例如 Guava’s`com.google.common.hash.Hashing`
- 选择关键属性
- 生成对应的equals或hashCode方法

例如： `return Objects.hashCode(pcId, emailAddress, reporterId, numberInfo, idNum);`

### Item 12: Always override `toString`

这条大概日常用得最多了，“一致，有效”

- 默认的实现是`类名@十六进制`hashCode的十六进制表示
- 这个实现不“强制”，只是从实际使用角度更“方便”
- 最好返回类中所有的“有效”信息
- 返回格式“不一定”需要固定：不固定可能造成测试问题（输出顺序），但保持灵活性；固定可以提高确定性，可读性，但丧失灵活性，提供给第三方使用时，对方可能已写了对应的parser，如果后续有变更……
- 无论哪一种情况，最好做文档说明
- 无论是否采取固定方式，都需要提供访问“有效”信息的方法

对于值类型，最好重写此方法；静态工具类、枚举类不需要；再次推荐AutoValue

### Item 13: Override `clone` judiciously

一句话总结，绝大数情况下，不需要重写`clone`方法；需要复制的话，提供一个copy方法更好，类似于——

```
// Copy constructor
public Yum(Yum yum) { ... };
// Copy factory
public static Yum newInstance(Yum yum) { ... };
```

- 如果不实现空接口`Cloneable`，调用`clone`方法时会抛出`CloneNotSupportedException`异常（Object的clone方法是protected级别）
- clone方法的“general contract”：x.clone() != x 为真；x.clone().getClass() == x.getClass() 为真；x.clone().equals(x)为真(不强求？)
- 重写方式：先调用`super.clone`；然后对必要的组件进行处理；最后转换为需要的对象
- 如果需要确保线程安全，clone方法需要加锁
- “必要的组件”如果不处理，clone后的新对象中的组件“可能”与原始对象的组件指向了相同的引用，然后就……

```
    //Clone method for class with references to mutable state
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            // 必要的组件进行clone
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

示例—— 包含了私有静态Entry类的HashTable类 


```
public class HashTable implements Cloneable {
        private Entry[] buckets = ...;
        private static class Entry {
            final Object key;
            Object value;
            Entry next;
            Entry(Object key, Object value, Entry next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }
        }
    // Remainder omitted
    }
```
直接进行clone无效——

```
    // Broken clone method - results in shared mutable state!
    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

需要先为Entry对象提供deepCopy方法——
```
    // Recursively copy the linked list headed by this Entry
    Entry deepCopy() {
        return new Entry(key, value,
                next == null ? null : next.deepCopy());
    }
```

再进行clone操作——

```
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

如果buckeys很大，上面的clone方法将很容易导致SOO；因为进行deepCopy时每一次调用都占据了一个栈frame。改进版本——

```
    // Iteratively copy the linked list headed by this Entry
    Entry deepCopy() {
        Entry result = new Entry(key, value, next);
        for (Entry p = result; p.next != null; p = p.next)
            p.next = new Entry(p.next.key, p.next.value, p.next.next);
        return result;
    }
```

### Item 14: Consider implementing `Comparable`

- `compareTo`不是Object对象中声明的方法，是`Comparable`接口中的唯一方法
- 实现了此方法的对象数列，可以直接通过`Arrays.sort(a)`进行排序
- 只有相同的对象之间才能进行对比，否则跑出`ClassCastException`异常
- 接口方法返回int类型，通常选择返回`-1, 0, 1`，对于相同类型对象x, y来说，`x.compareTo(y)`返回-1表示x小于y；0表示相等；1表示x大于y。参考Integer的实现——

```
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }

    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
```

- 同equals方法一样，此接口方法也有传递性，反射性，对称性
- 此方法返回0时并不要求两个对象相等，但推荐做相等的实现。区别在于，基于equals方法时（例如HashSet中），`new BigDecimal("1.0") `, `new BigDecimal("1.0") `被认为时两个对象；如果基于compareTo方法（例如TreeSet），则被认为只添加了一个对象
- Java库中多个类依赖这个接口，包括 `TreeSet`, `TreeMap`, `Collections`以及`Arrays`
- 需要对类中的多个关键属性进行对比时，不建议使用`>, = , <`的符号，而使用静态方法或Comparator。例如——

```
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};

// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder =
        Comparator.comparingInt(o -> o.hashCode());
```

对于`PhoneNumber`对象来说，使用Comparator时需要注意，第一次需要进行明确的类型指定`(PhoneNumber pn) -> pn.areaCode`

```
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
            .thenComparingInt(pn -> pn.prefix)
            .thenComparingInt(pn -> pn.lineNum);
public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

## 3 Classes and Interfaces

### Item 15 Minimize the accessibility of classes and members

访问控制是开始写代码时接触最早，应用最多都。控制限制好处一大堆，但似乎实际使用中用得都很随意？

第一原则：越严格越好。理论上，定义好要暴露到API之后，其它所有到属性和方法都变成private的；除了静态常量（大写、下划线）

对于类似`public static final Thing[] VALUES =  { ... };`都对象，可以处理为——

```
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

# 或者
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}

```

关于Module：(还没实际使用过)

- Java 9中引入了`module`模块（一组package构成一个module；一组class构成一个package），可以通过`module-info.java`声明当前module中哪些包和类可以被访问，其它不在声明之列的package即使是public的或protected的，都无法在module之外被访问到
- 如果把module都jar包放到类class path下而不是module path下；则module中都上面都限制失效。回退到正常到访问限制

Item16: In public classes, use accessor methods, not public fields

直接暴露属性违反了上一条的“封装”原则。所以要定义 访问者(accessor)方法和修改者(mutator)，即通常所说的getter和setter。

例外说明—— 

- 如果类本身是package private的或者是内部类，暴露属性也可以——代码更少不是～ 

- 实际上Java平台组建中的一些类，例如`java.awt`包下的`Point`, `Dimension`直接暴露类field——主要是基于性能考虑

- 如果暴露的field是不可修改的，也问题不大，例如——

```
// Public class with exposed immutable fields - questionable
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;
    public final int hour;
    public final int minute;
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute); this.hour = hour;
        this.minute = minute;
    }
 // Remainder omitted
}
```

### Item 17: Minimize mutability

imutable class：整个生命周期内无法进行任何修改操作的类。Java平台类库中的String、出箱后的基础类、BigInteger和BigDecimal都是

实现imutable类的五个原则：

- 不提供可以修改类状态的方法
- 确保类不会被扩展，定义为final级别的类(历史原因，BigInteger和BigDecimal实现时还没有这个共识？所以不是final级别的，所以在使用时需要进行必要的检查)

```
public static BigInteger safeInstance(BigInteger val) {
       return val.getClass() == BigInteger.class ?
               val : new BigInteger(val.toByteArray());
}

```

- 所有属性为final类型
- 所有属性为私有
- 确保对任何可变组建的访问为独有/排他的

满足上述规则的示例——一个简单的`复数`类 

```
// Immutable complex number class public final class Complex { private final double re; private final double im;
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
            //double 类型的对比使用compare，不用 == 
           && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    } }
```

优点说明：  
- 基础的四则运算方法称为“函数式”方法：调用一个函数进行操作，而不是执行一个“过程”或“命令”——对应的称呼为“过程式”(procedural)/“命令式” (imperative）
- 使用介词plus而不是动词add
- 函数式确保了类的不可变，默认就是线程安全的，不需要在多线程场景下进行加锁操作——因为对象本身就是不可变，这几乎是实现线程安全的最简单方式 
- 不需要提供“防御性复制(defensive copy)”和clone方法
- 不可变类可以为一些高频使用的场景提供静态实例（相当于提供cache），可以降低GC和内存占用

缺点： 
- 需要为每一个值提供独立的对象，创建这些对象是昂贵的，可能在特定场景下出现性能问题

序列化说明——  
如果不可变类中包含类可变属性，并且实现类`Serializable`。则必须显示地提供`readObject`, `readResolve`方法，或使用`ObjectOutputStream.writeUnshared`，`ObjectInputStream.readUnshared`方法

所以，能不可变就不可变；非变不可时，能少变就少变。样例参考`CountDownLatch`类的实现

### Item 18: Favor composition over inheritance

注：这里讨论的继承仅指对类的继承，不包括对接口的继承。

继承可能出现的问题：  

- 子类的表现依赖父类的实现，有可能由于父类的实现导致子类的意图出现偏差。（举例HashSet的子类意图获取从开始到现在一共有多少元素被添加到Set中——但现在的HashSet似乎已经没有了addAll方法？——不影响这种可能的场景出现）而为了解决可能出现的问题所做的“妥协”可能又引入新的问题：不易使用、耗时、易错、还可能影响性能
- 新发布版本中的父类可能“新”引入了方法：a.) 子类重新了现有的方法，调用前都做了前置检查，但父类的新方法可能破坏这种检查；b.) 子类原来的方法正好与父类新引入的方法同名，导致1)返回类型不一致，编译失败；2)出现a中的问题

所以，建议使用组合。新建一个类，将原父类作为一个私有属性引入，然后组合想要的功能。想要调用原父类，在这个新建的类中做转发(forwarding)处理即可，对于的方法称为"转发方法"(forwarding methods)。

新建的类称为包装类(wrapper)，这也是所谓的"装饰模式"(Decorator pattern)，或者通常所谓的“委托” (delegation)。

唯一的确定大概是在回调框架中，对象传递自身的引用到其他对象中，然后等待回调。由于对象不知道自己的wrapper是谁，只传递了自己(不包括wrapper)到调用对象里去。——这就是所谓的“SELF problem”

对性能和内存占用的担心在实际使用中被证明是可以忽略的。只是写wrapper比较无聊，仅此而已。

当且仅当Class B与Class A确实是同一种对象时才使用继承。（is-a）尽管Java平台类库里有很多违反了这一原则。例如Stack不是Vector，不应该扩展Vector；Property列表不是Hash表，所以Properties不应该扩展Hashtable。——一个太晚发现的问题。

继承违反了封装，所以尽管强大，却也容易出错。尤其是在跨包进行继承使用时尤其如此

### Item 19: Design and document for inheritance or else prohibit it

要使用继承，先做好“设计”和“文档”说明。否则就禁止使用继承。

如何做说明？需要说明实现是“如何”(how)工作的，会有哪些可能的影响。参考`java.util.AbstractCollection`类中的`public boolean remove(Object 0)`的说明。

尽管这违反了原则：好的API应该描述当前方法**做**什么而不是**如何做**。——这就是继承违反封装之后的**后果**。LOL

  在文档中启用`@implSpec`进行说明，以生产javadoc。这个tag在Java 8 中引入，Java 9中广泛使用，但默认是被javadoc忽略，需要在编译时通过`-tag apiNote:a:API Note:`，[用法参考](https://docs.oracle.com/javase/1.5.0/docs/tooldocs/windows/javadoc.html#tag)

有时候为了内部使用，设计protected级别的方法。如何考虑让哪些方法称为protected的呢？没有魔法，仔细思考。自己写子类进行测试验证。一旦公开，就称为“永久承诺”。

另外一个要求：构造方法中不能调用可重写的方法。因为此时父类中调用重写的方法，但此时子类构造方法还没执行，但子类重写的方法却被调用，结果无法预期。

可以调用其他的私有方法，final方法，静态方法——因为这些方法无法重写。

如果实现了`Cloneable`或`Serializable`接口，注意`clone`和`readObject`方法中也不能调用重写的方法。

如果实现了`Serializable`接口的类包含`readResolve`或`writeReplace`方法，将这两个方法设置为protected，如果是private的讲被忽略执行。

普通类避免被继承：声明为final类型；或者将构造方法设置为private或默认的package-private级别。

如果需要继承，父类中至少确保自己不调用重写的方法。需要的话：可以将重写方法的实现重写包装到helper方法中。

### Item 20:  Prefer interfaces to abstract classes

接口优于抽象类。尽管两者都可以用来定制允许多重实现的实例

- 显然接口更灵活。任何现有的类都可以很容易扩展实现新的接口，而不需要调整层级；采用抽象类却可能导致层级混乱（代码组织形式）
- 对于混合来说`mixins`，接口是理想的方式。所谓混合，指允许其他可选方法混入现有的原始方法
- 无层级关系类型的类框架可以使用接口实现。例如“歌手”和“写歌手”
- 接口可以做安全，有力的扩展——通过item 18中提到过的包装类`wrapper classs`

可以利用“骨架实现类”(`skeletal implementation classs`)同时结合接口和抽象类的有点：由接口提供默认方法，同时这个类实现基本的非原始方法`non-primitive interface method`。这也被称为 “模板方式”(Template Method Pattern)

通常这种类称为`AbstractInterface`，这里的interface只要实现的接口。例如集合框架中的`AbstractCollection`、`AbstractSet`、`AbstractList`、`AbstractMap`

```
    // Concrete implementation built atop skeletal implementation
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);
        // The diamond operator is only legal here in Java 9 and later
        // If you're using an earlier release, specify <Integer>
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i]; // Autoboxing (Item 6)
            }
            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val; // Auto-unboxing
                return oldVal; // Autoboxing
            }
            @Override public int size() {
                return a.length;
            }
        };
    }
```

设计好哪些属于"原始"(primitive)类型方法；哪些不是，然后就可以组合出所谓的`AbstractInterface`。参考guava包中`com.google.common.collect`包下的`AbstractMapEntry<K,V>`的设计

### Item 21: Design interfaces for posterity

本节强调接口设计时的稳定性。因为在Java 8之后，接口可以增加方法，同时完成默认的实现`default method`。这样，实现此方法的类即使没有实现新增方法，也可以编译通过

显然，默认方法的实现可能导致运行时的问题。举例——

`Collection`接口中的`removeIf`方法提供了默认实现

```
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

Apache的common包下的`org.apache.commons.collections4.collection.SynchronizedCollection`采用wrapper的方式实现了Collection接口，内部提供了一个锁完成同步操作，直到今天依然没有实现新增的`removeIf`接口，依赖默认实现

那么，如果多个线程操作这个类对象并且调用`removeIf`方法，必然会抛出`ConcurrentModificationException`异常

所以，这种依赖default实现的接口方法应该尽力避免——尽管这让子类的实现更容易。

>While it may be possible to correct some interface flaws after aninterface is released, you cannot count on it.

### Item 22: Use interfaces only to define types

这一章好像主要是用来吐槽：“利用接口提供静态变量”的操作？

当“类”实现了某个“接口”时，这里的“接口”应该理解为是一种“类型”(type)，这一“类型”代表了“类”的实例

区别： `java.lang.Class`实现了`java.lang.reflect.Type`接口 

```
package java.lang.reflect;

/**
 * Type is the common superinterface for all types in the Java
 * programming language. These include raw types, parameterized types,
 * array types, type variables and primitive types.
 *
 * @since 1.5
 */
public interface Type {
    /**
     * Returns a string describing this type, including information
     * about any type parameters.
     *
     * @implSpec The default implementation calls {@code toString}.
     *
     * @return a string describing this type
     * @since 1.8
     */
    default String getTypeName() {
        return toString();
    }
}
```
一种错误的使用场景是所谓的“常量接口”(constant interface)，不包含任何方法，只有静态final属性，作为常量提供。

```
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // Mass of the electron (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

想要使用这些常量，就的实现此接口——这容易引起误解和混乱。如果将来不需要这些常量时，依然需要保留对接口的实现以确保兼容性。Java平台库中的`java.io.ObjectStreamConstants`就是这样的错误。

这一场景应当使用枚举类型或公共类(utility class)的方式提供服务，上面的例子应该修改为——

```
// Constant utility class
package com.effectivejava.science;
public class PhysicalConstants {
    private PhysicalConstants() { } // Prevents instantiation
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
注意`_`下划线的使用，Java 7引入，在数值表达中没有实际含义，用来方便区分。以三位一组进行划分。

### Item 23: Prefer class hierarchies to tagged classes

类中包含了一个标签用来区分不同实例的形状。下例中的`final Shape shape`就是所谓的TAG——“我觉得还挺好”——包含很多问题： 样板代码，枚举声明，switch语句……一顿花样操作。

无论是阅读，还是GC都增加了负担。如果TAG不能在构造函数时确定，还不能声明为final级别，更琐碎。总之：冗长、易错、低效(verbose，error-prone，inefficient)

```
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    // Tag field - the shape of this figure
    final Shape shape;
    // These fields are used only if shape is RECTANGLE
    double length;
    double width;
    // This field is used only if shape is CIRCLE
    double radius;
    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

前面吐槽继承，现在是继承显身手的时候了。定义抽象类，抽象出相同的方法或属性；基于TAG的实现抽象为抽象方法，子类做实现，如果子类还有自己的偏好，自由处理。

处理之后，是不是感觉“心理负担”都轻了——没有样板代码(boilerplate)，没有switch，没有枚举声明，所有属性都是final级别，需要定制时，不需要访问父类就可以直接添加子类的特性

```
// Class hierarchy replacement for a tagged class
abstract class Figure {
    abstract double area();
}
class Circle extends Figure {
    final double radius;
    Circle(double radius) { this.radius = radius; }
    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    @Override double area() { return length * width; }
}
```

如果有需要定义Squre，还可以直接使用——

```
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

### Item 24: Favor static member classes over nonstatic

四种嵌套类(nested classed)：静态成员类(static member classes)、非静态成员类(nonstatic member classes)、匿名类(anonymous classes)、局部类(local classes)

注意静态成员类和非静态成员类的区别：

- 任何一个非静态类的实例都隐含包含在起所在类的某个实例中。如果非静态类实例想要单独存在，必须声明为静态类
- 这种联合关系会更占有内存空间并且更耗时（加入外部类对象已经不需要存在了，但由于有非静态类实例的存在，导致无法进行GC）非静态类通常会用来作为适配器，例如——

```
//Typical use of a nonstatic member class
public class MySet<E> extends AbstractSet<E> {
        // Bulk of the class omitted
    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }
    private class MyIterator implements Iterator<E> {
    }
}
```

匿名类的限制：无法实例，声明立即使用；不能做`instanceof`检查；不能实现接口或扩展其他类；

在lambda引入Java之前，常用匿名类；但现在大部分场景都可以使用lambda代替匿名类；

局部类使用的场景更少，相当于局部变量，收到的限制跟局部变量相同。


[示例参考](https://www.cis.upenn.edu/~matuszek/General/JavaSyntax/inner-classes.html) 

静态成员类(static member classes)

```
public class OuterClass {
    int outerVariable = 100;
    static int staticOuterVariable = 200;
 
    static class StaticMemberClass {
        int innerVariable = 20;
         
        int getSum(int parameter) {
            // Cannot access outerVariable here
            return innerVariable + staticOuterVariable + parameter;
        }       
    }
     
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        StaticMemberClass inner = new StaticMemberClass(); 
        System.out.println(inner.getSum(3));
        outer.run();
    }
     
    void run() {
        StaticMemberClass localInner = new StaticMemberClass();
        System.out.println(localInner.getSum(5));
    }
}

```

非静态成员类(nonstatic member classes)
```
// member class 
public class OuterClass {
    int outerVariable = 100;
 
    class MemberClass {
        int innerVariable = 20;
         
        int getSum(int parameter) {
            return innerVariable +  outerVariable + parameter;
        }       
    }
     
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        MemberClass inner = outer.new MemberClass(); 
        System.out.println(inner.getSum(3));
        outer.run();
    }
     
    void run() {
        MemberClass localInner = new MemberClass();
        System.out.println(localInner.getSum(5));
    }
}
```

匿名类(anonymous classes) 匿名类就是在使用时直接使用new关键字创建，作为方法参数或对象直接使用(不赋值给任何变量)。例如`new Thread(){...}.start()`

```
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import javax.swing.*;
 
public class OuterClass extends JFrame {
     
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        JButton button = new JButton("Don't click me!");
        button.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent event) {
                System.out.println("Ouch!");
            }
        });
        outer.add(button);
        outer.pack();
        outer.setVisible(true);
    }
}
```

局部类(local classes)

```
public class OuterClass {
    int outerVariable = 10000;
    static int staticOuterVariable = 2000;
     
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        System.out.println(outer.run());
    }
     
    Object run() {
        int localVariable = 666;
        final int finalLocalVariable = 300;
         
        class LocalClass {
            int innerVariable = 40;
             
            int getSum(int parameter) {
                // Cannot access localVariable here
                return outerVariable + staticOuterVariable + 
                       finalLocalVariable + innerVariable + parameter;
            }       
        }
        LocalClass local = new LocalClass();
        System.out.println(local.getSum(5));
        return local;
    }
}
```

### Item 25: Limit source files to a single top-level class

这条比较好理解。一个源码文件尽量只对应一个类。如果有嵌套类的需求，参考上一条说明。

如果违反这一个原则，有可能导致：因编译的顺序导致出现不同的运行结果。举例——

`Main.java`中的内容如下——

```
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

`Utensil.java`中的内容如下——

```
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pan";
}
class Dessert {
    static final String NAME = "cake";
}
```

这时候没有问题。如何还有另外一个`Dessert.java`的内容如下——

```
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pot";
}
class Dessert {
    static final String NAME = "pie";
}
```

此时如果使用`javac Main.java Dessert.java`进行编译，可能会提示报错（也可能不报错）。因为先编译Main.java时，已经加装了Utensil和Dessert类对象；再编译Dessert.java时，会发现有多重定义。

如果只选`javac Dessert.java Main.java`，可能输出的就是`potpie`结果了。由于不同的编译顺序导致运行结果有差异，这显然不能接受。——这也是这一原则的意义所在了

## 4 Generics

Java 5引入泛型，在此之前，从集合中读取对象时，每次都需要做强制转换。有泛型之后，编译器会自动做转换的动作，如果插入的类型错误，会编译报错。

### Item 26: Don't use raw types

类或接口声明中包含了一个或多个类型参数时，被称为泛型(generic class, generic interface)。`List<E>`读作`List of E`。

每个泛型也定义了“原始类型”(raw type)，不包含任何类型参数——主要为了兼容性。

声明了类型的泛型才会被进行编译检查；原始类型(不属于泛型系统)无法执行编译检查，只有在执行时抛出转换异常(ClassCastException)

当类型无法事先知道时，可以使用`Set<?>`无限通配类型(unbounded wildcard types)，依然是泛型的一种，保留了灵活性，类似`static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }`

两种例外场景(使用原始类型)：  
- 使用字面类(class literal)时，使用原始类型，不使用泛型。例如`List.class`, `String[].class`,`int.class`是合法表达；`List<String>.class`, `List<?>.class`不合法
- 使用`instanceof`检查时，例如下面的例子，注意检查之后使用时做通配处理

```
//Legitimate use of raw type - instanceof operator
if (o instanceof Set) { // Raw type
    Set<?> s = (Set<?>) o; // Wildcard type
}
```

总结一下： `Set<Object>`可以包含任意对象类型的Set；`Set<?>`包含任意特定类型的set；`Set`原始类型。前两种都是泛型，确保安全；后一种不属于泛型系统，不安全


|Term|Example|Item|
|---------|---------|---------|
|Parameterized type|List\<String\>|Item 26|
|Actual type parameter|String|Item 26|
|Generic type|List\<E\>|Items26,29|
|Formal type parameter|E|Item 26|
|Unbounded wildcard type|List<?>|Item 26|
|Raw type|List|Item 26|
|Bounded type parameter|<E extends extends Number>|Item 29|
|Recursive type bound|<T extends extends Comparable\<T\>\>|Item 30|
|Bounded wildcard type|List<? extends extends Number>|Item 31|
|Generic method|static \<E\> List\<E\> asList(E[] a)|Item 30|
|Type token|String.class|Item 33|

![](https://upload-images.jianshu.io/upload_images/3296949-679e354e043ebfad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Item 27: Eliminate unchecked warnings

消除未检查的警告。每一个未检查的告警都是可能导致运行时抛出类型转换的异常。

最简单的例子： `Set<Lark> exaltation = new HashSet();`没有声明类型，会提示未检查的转换，变更为`Set<Lark> exaltation = new HashSet<>();` 即可。添加Java 7中引入的`<>`钻石符号即可，不需要显式什么Lark类型。

- 尽力消除每一处的告警
- 如果无法消除，但可以确保对应的代码是类型安全的，可以使用`@SuppressWarning("uncheck")`注解进行消除。(类型安全时忽略了注解，那么类型不安全的场景也可能被忽略掉)
- 注解需要添加对应的注释
- 注解的范围确保为最小化，能注解到变量声明上的不注解到方法应用上；避免注解到整个类上

`ArrayList`的`toArray(T[] a)`的实现上的注解就有范围过大的问题。
```
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

更合适的方式—— 

```
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // This cast is correct because the array we're creating
        // is of the same type as the one passed in, which is T[].
        @SuppressWarnings("unchecked") T[] result =
                (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

### Item 28: Prefer lists to arrays

数组与泛型之间有两点不同。

第一，数组是协变型的(covariant)，如果`Sub`是`Super`的子类，则数组`Sub[]`是数组`Super[]`的子类型；而泛型是不变型的(invariant)，对应不同的类型`Type1`和`Type2`，List对应的类型之间没有父子关系[JLS 4.10. Subtyping](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.10) 

这意味着，前者只能在运行时报错；后者可以在编译期检查。例如——

```
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException

// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

第二，数组是“具象的”([reified](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.7))，这意味着数组在运行期间会强制检查对象的类型；例如上面的例子例子里，运行时不能将字符类型添加到Long数组中；而泛型是“擦除的”([erasure](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.6))，意味着类型检查发生在编译期，运行时类型被“擦除”——这也是为了兼容泛型在引入Java 5之前的代码。

所以，类似`new List<E>[]`，`new List<String>[]`，`new E[]`的声明都是非法的。否则，下面的代码就变成“合理的”，在String数组里获取出了一个Integer对象

```
// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

两种类型各擅胜场，数组性能更好，但泛型更安全，通用。对比——

使用时，需要每次都对Object进行强制类型转换
```
public class Chooser {
    private final Object[] choiceArray;
    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

使用泛型，注意类似`incompatible types`的报错，

```
public class Chooser<T> {
    private final T[] choiceArray;
    public Chooser(Collection<T> choices) {
        //编译错误，where T is a type-variable:
        choiceArray = choices.toArray();

        //使用下面的方式，需要注意消除warning: [unchecked] unchecked cast
        choiceArray = (T[]) choices.toArray();
    }
}
```

通用正确的方式—— 

```
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

### Item 29: Favor generic types

上一条标题鼓励使用List代替Array，这一条鼓励使用Array。不过两条都是鼓励使用“泛型”，不同的场景使用不同的“泛型”方式。数组方式的“泛型”性能更好。

参考下面的例子——泛型的“候选”，使用`Object`的方式实现`Stack`栈。每次使用时，用户需要手动进行类型转换

```
// Object-based collection - a prime candidate for generics
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
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

第一步是直接使用类型参数E代替Object对象，

```
// Initial attempt to generify Stack - won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
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
   // no changes in isEmpty or ensureCapacity
}
```

但如同上一条说说，数组是“具象的”，不能直接声明类型参数，`elements = new E[DEFAULT_INITIAL_CAPACITY];`这样声明将无法通过编译。

两种解决办法——

第一，在构造函数中进行强制类型转换—— `elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];`，对应编译器的告警，可以使用注解+注释的方式一次解决——


```
// The elements array will contain only E instances from push(E).
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]!
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

第二，将elements声明`Object`类型，保留构造函数的`elements = new Object[DEFAULT_INITIAL_CAPACITY];`，此时`E result = elements[--size];`将无法编译，需要进行强制转换`E result = (E) elements[--size];`，这会导致一个告警。根据注解最小化原则，最终的pop方法变更为——

```
// Appropriate suppression of unchecked warning
public E pop() {
    if (size == 0)
        throw new EmptyStackException();
    // push requires elements to be of type E, so cast is correct
    @SuppressWarnings("unchecked") E result =
            (E) elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

显然第一种方式更具有可读性，更简便——只在声明时做了一次类型转换；实践中也更常用。唯一的问题是造成了没有伤害的“堆污染”(heap pollution)：编译时的类型和运行时类型不一致，运行时数组类型将总是为`Object[]`

Java并不是原生支持List的，所以尽管鼓励使用List代替Array，但在泛型实践中不一定总是可以使用List。例如Java中的`ArrayList`就是在数组上实现的；其他类型例如HashMap，基于性能考虑，也会基于数组实现。

泛型的好处是对于类型参数不做具体限制，所以各种类型都声明，例如`Stack<Object>`，`Stack<int[]>`，`Stack<List<String>>`都是有效的。

如果想限制参数的类型，可以声明为类似`class DelayQueue<E extends Delayed> implements BlockingQueue<E>`，确保类型都是一个已知父类的子类即可。这样就可以声明`DelayQueue<Delayed>`进行使用了

### Item 30: Favor generic methods

有泛型类，也有泛型方法。静态实用类通常为泛型方法，例如`Collections`中的算法方法都是泛型方法。

原始类型——

```
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(S1);
    result.addAll(s2);
    return result;
}
```
编译可以通过，但会提示两个告警，修改为泛型方法——

```
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(S1);
    result.addAll(s2);
    return result;
}
```

注意对类型参数的声明`<E>`，位于方法修饰符`staic`和返回值`Set<E>`之间。如果想更通用，可以使用“有界通配符”(`bounded wildcard types`)(在item 31中具体讨论)

有时你需要创建一个不可变对象，但又要应用给不同的类型。这是可以使用“泛型单例工厂”模式(`generic singleton factory`)(在item 42中讨论)，例如 `Collections.reverseOrder`——

```
@SuppressWarnings("unchecked")
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}
```

例如写一个“恒等函数分发器”(identity function dispenser)，库函数`Function.identity`(输入什么，输出什么)提供了这样的方法，但重写也是有意义的——

```
// Generic singleton factory pattern
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}

//调用示例
public static void main(String[] args) {
    String[] strings = { "jute", "hemp", "nylon" };
    UnaryOperator<String> sameString = identityFunction();
    for (String s : strings)
        System.out.println(sameString.apply(s));
    Number[] numbers = { 1, 2.0, 3L };
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers)
        System.out.println(sameNumber.apply(n));
}
// 依次输出 jute hemp nylon 1 2.0 3
```

类型参数被调用此类型的表达式绑定。这称为“递归类型绑定”(`recursive type bound`) ，例如对比接口——表示对比的对象只能是已经被绑定的类型`T`

```
public interface Comparable<T> {
    int compareTo(T 0)
}
```
如果一组元素都实现了`Comparabel`接口，然后在这一组元素中搜索、计算最大值最小值之类的计算。这要求这些元素直接是可以相互对比的`mutually comparable`，使用递归类型绑定来表达这种相互可比性——

 `public static <E extends Comparable<E>> E max(Collection<E> c)`

这里的`<E extends Comparable<E>>` 读作“任何可以与自己比较的类型E”(any type E that can be compared to itesel)

实现查找最大值的操作——

```
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

### Item 31: Use bounded wildcards to increase API flexibility

使用“有界通配符”(`bounded wildcard`)让API更灵活。听起来很“显然”，实现时稍微有点复杂，记住一个“胸大肌”原则- -|| `PECS`(producer-extends, consumer-spuer)，作为生成者时，使用`extends`关键字；作为消费者时，使用`spuer`关键字。

例如，对于前面举例的Stack类——

```
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

如果增加一个`pushAll`方法，尝试以下实现——

```
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

编译没有问题，但使用时可能报错，例如——
```
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```

此时可以将参数声明为`Iterable<? extends E> src`，表示“E的子类的迭代器”(`Iterable of some subtype of E`)，而不是之前的“E类的迭代器”(`Iterable of E`)，这里参数src针对当前类Stack来说属于“生产者”，用来给Stack类提供“原料”

同样的，如果增加一个`popAll`方法，应该实现为——

```
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

这里参数dst针对当前类Stack来说属于“消费者”，消费类Stack中的“原料”。

其实就是类型转换时的“层级”思路，向类型E进行转换时，至少是类型E的子类；使用类型E装填时，至少是类型E的父类——只有这样才能确保默认的类型转换成功。

对应前面提到的unio方法也需要做类似的修改：`public static <E> Set<E> union(Set<? extends E> s1, Set<? extends> s2)`，注意返回值不需要包含通配符，否则将给使用方增加负担。

使用场景——

```
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

注意，这种方式在Java 8之前会报错，最后一步需要声明为`Set<Number> numbers = Union.<Number>union(integers, doubles);`

另外两点需要讨论的——

第一，对于max函数——

`public static <T extends Comparable<T>> T max(List<T> list)`

现在需要重新声明为——

`public static <T extends Comparable<? super T>> T max(List<? extends T> list)`

参数list作为生产者，所以声明为`? extends T`；
对于类型参数T，这是第一次对类型参数添加通配符。Comparable接口是参数T的消费者（用来对比T并返回int值用于排序），所以需要先声明为`Comparable<? super T>`

这大概是最复杂的声明了- -||

第二，对于类型参数和通配符的二元对立，通常二者只能取其一。假如对于List提供一个swap函数，交换对于位置上的值——

```
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

显然第二种方式更简单（如果类型参数在方法声明中只出现一次，使用通配符代替）。但对于第二种方法的实现却提示无法编译——

```
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}

## 
error: incompatible types: Object cannot be
converted to CAP#1
list.set(i, list.set(j, list.get(i)));
                                ^
where CAP#1 is a fresh type-variable:
  CAP#1 extends Object from capture of ?
```
因为参数list的类型是`List<?>`，除了null值，任何值都无法添加到`List<?>`中，此时需要提供一个帮助函数——

```
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}
// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```
此函数指定参数list的类型为E，所以从list中取出的元素类型依然是E，也就可以重新添加到list中了。

### Item 32: Combine generics and varargs judiciously

尽管“可变参数方法”(varargs methods)和“泛型”(generics)都是在Java 5引入的，但两者之间的交互并不好。

可变参数的引入是为了允许客户端调用方法是传递不定量的参数，但这是一种“抽象泄露”(leaky abstraction)

>a leaky abstraction is an abstraction that exposes to its users details and limitations of its underlying implementation that should ideally be hidden away. 

当可变参数是泛型类型时，将会出现编译告警。类似—— 
```
warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
```

因为泛型属于非具象化类型，编译期的信息比运行期更多。由于出现堆污染而导致类型转换失败，例如——

```
// Mixing generics and varargs can violate type safety!
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // Heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
```

所以，为什么声明一个包含了泛型的可变参数方法是合法的？而显示地创建一个泛型数组反而属于非法？前者为了方便(可以确保类型安全时就可以使用)- -|| 后者会导致类型不安全（因为数组属于具象类型，泛型属于非具象- -||）

库函数中包含很多“泛型可变参数”——  
- `Arrays.asList(T... a)`
- `Collections.addAll(Collection<? super T> c, T... elements)` 
- `EnumSet.of(E first, E... rest)`

Java 7 引入的注解`@SafeVarargs`确保对这些方法的调用不会被编译器告警。

如何确保包含了这些泛型可变参数的方法是类型安全的？ 

- 1. 方法中不会在可变参数数组中存储任何对象（这样做相当于重写了这些参数）
- 2. 数组对象的引用不会被逃逸(确保数组在方法外不可见)

例如以下方法——

```
# 类型不安全，违反第二条原则，暴露了数组对象的引用
static <T> T[] toArray(T... args) {
    return args;
}
```

使用以下方式调用上述方法——

```
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError(); // Can't get here
}
```

编译器创建一个可变参数的数组，然后传递两天类型T的实例给`toArray`方法，此时被分配的是显然是对象数组`Object[]`，无论传递给`pickTwo`方法的是什么对象。`toArray`方法直接返回了此对象数组，然后再返回给原始的调用方。类型转换错误就显而易见了——

```
public static void main(String[] args) {
    String[] attributes = pickTwo("Good", "Fast", "Cheap");
}
```

所以上面的第二点就很明确了，将引用暴露到方法之外导致了类型不安全，引起堆污染，导致类型转换错误。正确的例子——

```
// Safe method with a generic varargs parameter
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

在确保类型安全的情况下，任何包含泛型可变参数的方法都应该使用注解`@SafeVarargs`，以避免调用法的编译告警。

确保方法不能被重写。Java 8中，此注解只能适用于静态方法或final实例的方法；Java 9中，私有方法也可以使用

### Item 33: Consider typesafe heterogeneous containers

类型安全的混合型容器。一般的容器使用普通的泛型，类型参数觉得了容器中只能包含特定类型的对象。

可以通过对容器中的Key做类型参数处理，以达到让容器可以包含不同类型对象的目的。使用`Class`对象作为容器的Key，因为`Class`本身是一种泛型`Class<T>`，`String.class`的类型为`Class<String>`，`Integer.class`的类型为`Class<Integer>`，使用字面类做为方法的参数被称为“类型标记”(type token)

```
// Typesafe heterogeneous container pattern
public class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

// Typesafe heterogeneous container pattern - client
public static void main(String[] args) {
    Favorites f = new Favorites();
    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);
    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);
    System.out.printf("%s %x %s%n", favoriteString,
            favoriteInteger, favoriteClass.getName());
}
```

注意：

- 类中的Map对象的通配符并不是限制map自己，而是限制Key的类型，所以正好可以包含不同类型的Key，从而实现“混合”容器的效果
- Map中的值是Object类型，Map自身并不保证key和value之间的关系(value正好是key的类型，实际上Java的类型系统做不到这一点)，但没有关系，当从map中取数据时，可以重新建立这种关系：使用`Class`类的`cast`方法进行动态类型转换

```
# method in Class 
@SuppressWarnings("unchecked")
public T cast(Object obj) {
    if (obj != null && !isInstance(obj))
        throw new ClassCastException(cannotCastMsg(obj));
    return (T) obj;
}
```

客户端可能“恶意地”添加了“原始类型”的key，可以对put方法进行优化——在添加前就做动态转换

```
// Achieving runtime type safety with a dynamic cast
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(type, type.cast(instance));
}
```

不能添加“非具象化”的类型，例如`List<String>`，因为`List<String>.class`是错误的语法，实际上，`List<String>`和`List<Integer>`共享同一个Class类型——`List.class`

现在方法可以接受任意类型的`Class`，如果想限制为特定的类型，可以使用`bounded type token`：指定了type token可以表示的类型。

Java中的注解大量使用了这种方式。读取注解的方法—— `public <T extends Annotation> T getAnnotation(Class<T> annotationType);` 这里的参数 `annotationType`被限定为必须为注解类型

如果要将`Class<?>`类型的对象传递给“限制类型token”的方法中，使用`Class<? extends Annotation`将导致编译告警。此时可以使用`asSubclass`方法——

```
// Use of asSubclass to safely cast to a bounded type token
static Annotation getAnnotation(AnnotatedElement element,
                                String annotationTypeName) {
    Class<?> annotationType = null; // Unbounded type token
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(
            annotationType.asSubclass(Annotation.class));
}

# Class中的getAnnotation和 asSubclass方法——
@SuppressWarnings("unchecked")
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {
    Objects.requireNonNull(annotationClass);

    return (A) annotationData().annotations.get(annotationClass);
}

@SuppressWarnings("unchecked")
public <U> Class<? extends U> asSubclass(Class<U> clazz) {
    if (clazz.isAssignableFrom(this))
        return (Class<? extends U>) this;
    else
        throw new ClassCastException(this.toString());
}
```

## 5 Enums and Annotations

枚举和注解是Java支持的两种特殊的引用类型

### Item 34: Use enums instead of int constants

引入枚举之前，Java使用的是“整数枚举模式”(`int enum pattern`)或者“整数枚举模式”(`String enum pattern`)，类似下面的方式——

```
// The int enum pattern - severely deficient!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;
public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

整数枚举属于“不变变量”(constant variables)。值变化时，必须重新编译才会生效；不方便调试（只是打印出对应的数字）；无法可靠地获得变量组的长度。整数枚举就更不方便，甚至出现硬编码情况。

枚举类型正式为了解决这类问题——跟其他语言的枚举(只是int的变形)不同，Java的枚举类是完整的Class类型。每一个枚举实例都是一个`public static final`的对象，并且外部无法访问构造函数

```
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

编译类型安全，任何非空值都必然是定义的枚举类型之一，否则将抛出错误。增加枚举常量或调整枚举类中的常量位置不需要重新编译。打印时可以调用`toString`方法。所以即使“冥王星”从“太阳系”被除名，下面的程序也运行良好——

```
// Enum type with data and behavior
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);
    private final double mass; // In kilograms
    private final double radius; // In meters
    private final double surfaceGravity; // In m / s^2

    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;
    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

枚举类中实现抽象方法，被称为“特定常量方法实现”(`constant-specific method implementations`)

```
// Enum type with constant-specific method implementations
public enum Operation {
    PLUS {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y;}},
    DIVIDE{public double apply(double x, double y){return x / y;}};

public abstract double apply(double x, double y);
}
```

还可以有进一步升级版本——

```
// Enum type with constant-specific class bodies and data
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }
    
    @Override 
    public String toString() { return symbol; }
    
    public abstract double apply(double x, double y);

    // Implementing a fromString method on an enum type
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(Collectors.toMap(Object::toString, e -> e));
    
    // Returns Operation for string, if any
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
}
```

枚举类自动包含一个`valueOf(String)`的方法，类似`Operation.valueOf("PLUS")`将返回`PLUS`枚举常量。如果想上面那样重写了`toString`方法，需要考虑增加一个`fromString`方法。

简洁 VS. 安全&灵活 

简洁版本(能写出来也不错了~)，但不易维护，如果有新增的特殊日，需要**记得**处理switch场景——
```
// Enum that switches on its value to share code - questionable
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;

        switch(this) {
            case SATURDAY: case SUNDAY: // Weekend
                overtimePay = basePay / 2;
                break;
            default: // Weekday
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

安全，灵活版本(可以读出优雅的美感)

```
// The strategy enum pattern
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    PayrollDay() { this(PayType.WEEKDAY); } // Default

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // The strategy enum type
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);

        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {

            int basePay = minsWorked * payRate;

            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

枚举类性能表现类似整数常量，只是初始化时需要一些更多的时间和内存。当一组常量的内容在编译期间就已知时，总是使用枚举类

### Item 35: Use instance fields instead of ordinals

枚举类自带方法`ordinal()`，返回枚举常量声明时的“顺序”，从0开始。但不建议使用这个方法，依赖这个值表示枚举常量的位置时，丧失很多灵活性。已经使用中的枚举类再增加、减少新的枚举常量后，原始的“顺序”位置将变化，可能导致使用的混乱。

错误使用方式——
```
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

正确使用方式：使用类属性提供要暴露的API——

```
public enum Ensemble {

    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    
    Ensemble(int size) { this.numberOfMusicians = size; }
    
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

>Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as `java.util.EnumSet` and `java.util.EnumMap`.
>Returns: the ordinal of this enumeration constant

### Item 36: Use EnumSet instead of bit fields

使用EnumSet代替“位字段”(`bit field`)。后者通常称为“整数枚举模式”(`int enum pattern`)，即使用2的不同幂赋值给不同的整数。例如——

```
// Bit field enumeration constants - OBSOLETE!
public class Text {s
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    // Parameter is bitwise OR of zero or more STYLE_ constants
    public void applyStyles(int styles) { ... }
}

# usage
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

性能最佳，还可以利用位运算算法；缺点是不易debug，交互性差？（无法遍历），需要在提供API之前就确定“最大的位”，无法更改大小

使用EnumSet代替。属于Set，提供完整的Set相关API，内部是一个位Vector。如果总大小少于64，EnumSet使用一个Long值表达——性能解决位字段方式。目前惟一的确定是无法创建不可变的EnumSet（如果使用`Collections.unmodifiableSet`包装为不可变Set，会有性能损失）

所以上面的例子可以代替为——

```
// EnumSet - a modern replacement for bit fields
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) { ... }
}
```

调用方式变更为`text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));`

### Item 37: Use EnumMap instead of ordinal indexing

有时会在数组或集合内使用枚举类的ordinal值做为index，对于不同年限的“植物”`Plant`类对象（一年生、两年生、多年生）——

```
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    final String name;
    final LifeCycle lifeCycle;
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    @Override public String toString() {
        return name;
    }
}
```

如果想根据`LifeCycle`列出对于的植物列表，可以这样操作——创建三个Set对应三种周期，遍历所有的植物将不同的植物添加到对应的Set中，最后从打印出三个Set中的植物对象——


```
// Using ordinal() to index into an array - DON'T DO THIS!
Set<Plant>[] plantsByLifeCycle =
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>();
for (Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
// Print the results
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n",
            Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}

```

可以工作、编译不友好（进行了强制转换）、打印不友好（需要标记index代表的含义）。最严重的问题是使用`ordinal`作为数组下标时，需要确保下标正确，避免数组越界问题

Java平台的`EnumMap`使用枚举类型作为key值，性能足够（因为内部使用数组实现）。实现上述相同的功能（没有强制转换，不需要担心数组越界，性能相当。结合了数组的性能+Map的便捷）——

```
// Using an EnumMap to associate data with an enum
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

构建`EnumMap`时，使用类对象(Class Object)作为Map的key类型，`public EnumMap(Class<K> keyType){...}`。


打印语句可以使用`System.out.println(garden.stream().collect(groupingBy(p -> p.lifeCycle)));` (花园garden声明为List对象) 

或`System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));` (花园garden声明为数组对象)

stream方式会使用自己的map实现，不太可能是`EnumMap`类型，可以明确声明使用的map工厂的实现——

```
System.out.println(garden.stream().collect(Collectors
        .groupingBy(p -> p.lifeCycle, 
        () -> new EnumMap<>(Plant.LifeCycle.class),
         Collectors.toSet())));

# 打印区别—— garden中只包含了ANNUAL类型的P1
{ANNUAL=[p1], PERENNIAL=[], BIENNIAL=[]}
{ANNUAL=[p1]}



# groupingBy方法——
public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream) {...}   
      
```

区别在于，直接使用`EnumMap`时，会为每一个Enum类型创建Map，即使garden中没有Enum对应的实例；使用`stream`的方法时，只对有对应enum的实例创建map。

更复杂的场景，使用数组的数组来表示两个Enum实例之间的映射关系。`TRANSITIONS`二维数组表示三种不同物质状态()两两直接装换时的动作。例如："固体"(SOLID)变为“液体”(LIQUID)被称为“融化”（MELT）

```
// Using ordinal() to index array of arrays - DON'T DO THIS!
enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // Rows indexed by from-ordinal, cols by to-ordinal
        private static final Transition[][] TRANSITIONS = {
                { null, MELT, SUBLIME }, // 位置映射：
                { FREEZE, null, BOIL },  // 
                { DEPOSIT, CONDENSE, null }
        };

        // Returns the phase transition from one phase to another
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

构造精巧也就智力负担严重，依赖对二维数组的“维护”。稍有不慎，可能跑场数组越界，空指针异常，甚至有错误，但无报错

使用上面类似的思路变更后——

需要对Map的API使用比较熟练，构造一个map，key值为枚举变量，值是另外一个map。初始化操作比较烧脑(我还没掌握)——

- a cascaded sequence of two collectors，两个collector的级联顺序
- 第一个collector将source phase的状态transitions进行分组（参考上面提到的`Collectors.groupingBy`方法）
- 第二个collector创建一个EnumMap，映射了目的phase到transition（即`Collectors.toMap`方法）
- 第二个collector中的 mergeFunction 函数`((x,y)->y)`没有使用，是为了明确指定Map工厂使用`EnumMap`


```

import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

// Using a nested EnumMap to associate data with enum pairs
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // Initialize the phase transition map
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(Collectors.groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                Collectors.toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}

#  toMap method
public static <T, K, U, M extends Map<K, U>>
    Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction,
                                Supplier<M> mapSupplier) {}
```

对m的初始化操作看不清楚可以使用更直白的方式——

```
// Initialize the phase transition map
private static final Map<Phase, Map<Phase,Transition>> m =
        new EnumMap<Phase, Map<Phase,Transition>>(Phase.class);
static {
    for (Phase p : Phase.values())
        m.put(p,new EnumMap<Phase,Transition>(Phase.class));
    for (Transition trans : Transition.values())
        m.get(trans.from).put(trans.to, trans);
}
```

这时候如果需要增加一种类型，有必要学习一下四种物理态直接转换的动词表达了——

```
// Adding a new phase using the nested EnumMap implementation
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA; // 固体，液体，气体，等离子体
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), // 融化，凝固，
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID), // 沸腾，凝结
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID), // 升华(气化)，沉积
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS); // 离子化，去离子化
        ... // Remainder unchanged
    }
}
```

### Item 38: Emulate extensible enums with interfaces

在所有层面上，枚举类型都比本书第一版提到的“类型安全的枚举模式”(typesafe enum pattern)要好。表面来看，“可扩展性”是唯一的例外。

第一版提到的`typesafe enum pattern`示例——(第一版的item 21)
```
// The typesafe enum pattern
public class Suit {
    private final String name;

    private Suit(String name) { this.name = name; }
    
    public String toString() { return name; }
    
    public static final Suit CLUBS = new Suit("clubs");
    public static final Suit DIAMONDS = new Suit("diamonds");
    public static final Suit HEARTS = new Suit("hearts");
    public static final Suit SPADES = new Suit("spades");

}
```

但的确有场景需要扩展枚举性质的类型：操作码(operation codes，常被称为opcodes)。实现逻辑——

- 为操作码定义接口
- 枚举类型实现此接口
- 以接口的形式提供API

item 34中提到的四则运算的枚举以上述逻辑重新实现——

```
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {

    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

尽管枚举类型`BasicOperation`本身是不支持扩展的，但接口可以。如果需要支持“求幂”(exponentiation)和“求余”(remainder)，可以使用新的枚举类实现Operation接口——

```
// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

使用时甚至可以直接传递整个扩展后的枚举类型。

注意main方法中传递的实参`ExtendedOperation.class`

test方法使用了“有界类型token”`bounded type token`限制了参数`Class<T> opEnumType`的类型，要求形参`<T extends Enum<T> & Operation>` 参数`Class<T> opEnumType`既是枚举类型`Enum<T>`又是接口`Operation`的子类

```
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);

    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {

    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

也可以使用“有界通配类型”`bounded wildcard type`，参数定义为`Collection<? extends Operation>`，此时传递的就是Class对象——

```
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,
                            double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

这种扩展方式唯一的缺点是两个枚举类之间无法继承。Java系统库使用这种模式的类包括`java.nio.file.LinkOption`枚举类实现了接口`CopyOption`和`OpenOption`


### Item 39: Prefer annotations to naming patterns

历史上，使用“命名模式”(naming patterns)来定义要求特定元素，例如在JUnit 4之前，要求测试方法必须以`test`开头。

这种模式的缺陷：  
- 笔误导致的问题不可避免却无法事先检查
- 无法确保对应的要求只在被要求的场景下使用，例如Juni 3支持者对方法名的判断，如果在类名上使用`TestSomeClass`是无效的
- 没有好的方式使用参数化。如果测试中跑场特定异常时判断为通过，Junit 3框架支持起来就比较麻烦

注解可以很好解决以上问题。Junit 4框架采用注解方式代替之前的命名方式。通过一个玩具级别的测试框架说明——

注解声明——

```
// Marker annotation type declaration
import java.lang.annotation.*;

/**
* Indicates that the annotated method is a test method.
* Use only on parameterless static methods.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
注解上的注解被称为“元注解”(`meta-annotations`)，其中`@Retention(RetentionPolicy.RUNTIME)`表示此注解在运行时保留(三种方式`SOURCE`编译忽略, `CLASS`默认级别，编译期识别，但运行时忽略,`RUNTIME`编译+运行期)，即编译识别，虚拟机运行时也识别

`@Target(ElementType.METHOD)`表示只对方法生效，对类，变量声明等无效。注释中强调的只适用于静态方法，无法由编译期默认强制实现，触发定制自己的注解解析器`annotation processor`实现（参考`javax.annotation.processing`）

这种没有参数的注解方式被称为“标记式注解”(marker annotation)，可以使用以下方式执行——

```
// Program to process marker annotations
import java.lang.reflect.*;
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                } catch (Exception exc) {
                    System.out.println("Invalid @Test: " + m);
                }
            }
        }
        System.out.printf("Passed: %d, Failed: %d%n",
                passed, tests - passed);
    }
}
```

支持抛出特定异常的注解声明——

```
// Annotation type with a parameter
import java.lang.annotation.*;
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to succeed.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
Class<? extends Throwable> value();
}
```

value的返回值`Class<? extends Throwable> `表示“扩展了Throwable对象的某种类对象”。使用示例——

```
// Program containing annotations with a parameter
public class Sample2 {
@ExceptionTest(ArithmeticException.class)
public static void m1() { // Test should pass
    int i = 0;
    i = i / i;
}
@ExceptionTest(ArithmeticException.class)
public static void m2() { // Should fail (wrong exception)
    int[] a = new int[0];
    int i = a[1];
}
@ExceptionTest(ArithmeticException.class)
public static void m3() { } // Should fail (no exception)
}

```

此时在判断运行结果时，需要对抛出的异常进行进一步判断，类似——
```
catch (InvocationTargetException wrappedEx) {
    Throwable exc = wrappedEx.getCause();
    Class<? extends Throwable> excType =
    m.getAnnotation(ExceptionTest.class).value();
    if (excType.isInstance(exc)) {
        passed++;
    } else {
        System.out.printf(
        "Test %s failed: expected %s, got %s%n",
        m, excType.getName(), exc);
    }
} 
```

进一步，可以支持抛出多种异常，只需要将声明中的value值定义为数组即可：`Class<? extends Throwable>[] value();`，使用时，将指定的多个异常类定义如下——

```
// Code containing an annotation with an array parameter
@ExceptionTest({ IndexOutOfBoundsException.class,
    NullPointerException.class })
public static void doublyBad() {
    List<String> list = new ArrayList<>();
    // The spec permits this method to throw either
    // IndexOutOfBoundsException or NullPointerException
    list.addAll(5, null);
}
```

上述catch中的代码针对异常的解析判断需要进一步修改为——

```
catch (InvocationTargetException wrappedEx) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Exception>[] excTypes =
    m.getAnnotation(ExceptionTest.class).value();
    for (Class<? extends Exception> excType : excTypes) {
        if (excType.isInstance(exc)) {
            passed++;
            break;
        }
    }
    if (passed == oldPassed)
        System.out.printf("Test %s failed: %s %n", m, exc);
} 
```

Java 8支持注解`@Repeatable`代替上面类似的数组式声明，但使用时也有不方便的地方，类似——

```
// Repeatable annotation type
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Exception> value();
}
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}

# 使用场景
// Code containing a repeated annotation
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }

```

使用场景需要多次声明—— 

```
// Code containing a repeated annotation
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }

```

### Item 40: Consistently use the Override annotation

一句话总结：总是使用`Override`注解，如果你的方法重写了父类的相同方法声明。除非，父类中对应的方法声明为抽象方法。即使如此，添加上`Override`也没什么损失。

否则，下例中的错误我一开始也忽略了——

```
// Can you spot the bug?
public class Bigram {
    private final char first;
    private final char second;
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    public int hashCode() {
        return 31 * first + second;
    }
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

实际上，需要重新的`equals`方法的参数必须为Object对象`equals(Object object)`，上面的例子传递的参数是当前对象，所以equals方法并没有被重写。如果添加了`Override`的声明，编译期会检查出对应的错误。

当重新父类的方法时，许多编译期目前支持自动插入`Override`注解。如果没有添加此注解，还可以进行告警。

此注解支持针对接口或类中的方法。加入接口没有默认的方法，可以忽略使用此注解（此时应该也没有方法可以写这个注解吧？）

例如`Set`接口尽管扩展了`Collection`接口——`public interface Set<E> extends Collection<E> `，但没有添加任何新的接口。这种情况下，应该给所有的方法声明都添加上`Override`注解。


### Item 41: Use marker interfaces to define types

“标记接口”(marker interface)是指接口不包含任何方法，只用来声明某种属性。例如`Serializable`接口，实现此接口表示当前实例可以进行序列化即可以写入到`ObjectOutputStream`

相比于注解，标记接口有两个优势：
- 接口定义了一种类型，可以包含实现类，注解不行。前者可以在编译期检查错误，后者只能到运行时发现
- 接口可以更精确地定位。`ElementType.TYPE`类型的注解可以被任何类或接口使用；标记接口只能由类实现

但由于`ObjectOutputStream.write`的API设计原因——参数定位为Object——导致对`Serializable`接口类的检查只能在运行时才能被发现。

注解优于标记接口的地方在于它属于整个注解设施的一部分。

所以，如果标记的对象不只是类或接口，自然要使用注解方式；如果只针对类和接口，并且你希望提供以实现了标记接口的类作为参数的方法，那么应该使用标记接口。

如果使用了重度使用注解的框架，当然更偏好注解。

总之，是一个平衡的选择。item 22中说“如果不想定义一个类型，不要使用接口”；这里表达的意思正好相反。

## 5 Lambdas and Streams

Java 8 引入了“函数接口”(functional interfaces)，Lambdas和“方法引用”(method reference)。同时提供了Streams相关的API

### Item 42: Prefer lambdas to anonymous classes

只有一个抽象方法的接口在历史上一直作为“函数类型”(function types)使用。对应的实例称为函数对象(function objects)，表示某种方法或行为。自1997年Java 1.1发布以来，创建函数对象的方式都采用“匿名类”(anonymous class)。

例如，将一组字符串按照长度进行排序的实现——
```
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

Java 8中决定对这种场景(只有一个抽象方法的接口)使用lambda表达式进行特殊处理，被称为"lambdas"，上面的例子可以精简为——

```
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

省略的内容由编译期依赖“类型引用”(type inference)根据上下文进行推断。类型引用的规则比较复杂，参考 [JLS, 18](https://docs.oracle.com/javase/specs/jls/se8/html/jls-18.html)。

省略所有lambda参数的类型除非加上类型可以让程序更清晰。如果编译期报错——无法推断出类型，再进行明确声明。

上述例子进一步省略，使用“对比器构造方法”(comparator construction methods)`Collections.sort(words, comparingInt(String::length));`

终极简化方式：`words.sort(comparingInt(String::length));`

对于 item 34中的枚举列子，可以使用lambdas进行如下简化——为每个枚举常量传递一个lambda表达式，构造方法将lambda保存在字段op中，然后在apply方法中调用这个表达式

```
// Enum with function object fields & constant-specific behavior
public enum Operation {
    PLUS ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

`DoubleBinaryOperator`是Java 8中引入的函数接口——

```

/**
 * Represents an operation upon two {@code double}-valued operands and producing a
 * {@code double}-valued result.   This is the primitive type specialization of
 * {@link BinaryOperator} for {@code double}.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #applyAsDouble(double, double)}.
 *
 * @see BinaryOperator
 * @see DoubleUnaryOperator
 * @since 1.8
 */
@FunctionalInterface
public interface DoubleBinaryOperator {
    /**
     * Applies this operator to the given operands.
     *
     * @param left the first operand
     * @param right the second operand
     * @return the operator result
     */
    double applyAsDouble(double left, double right);
}
```

PS: 
- lambdas缺少声明和注释，如果计算无法自解释，或需要较多行数，不要使用lambdas。
- 传递给枚举构造函数的参数是在静态上下文中推断的，所以lamdbas无法访问枚举实例。


### Item 43: Prefer method references to lambdas

通常lambdas都很短，足够简洁。Java提供了比lambdas更简洁的函数对象：“方法应用”(method references)。例如，Map的merge方法——

- 使用lambdas方式：`map.merge(key, 1, (count, incr) -> count + incr);`
- 使用方法应用方式：`map.merge(key, 1, Integer::sum);`因为sum方法提供了lambda表达式相同的功能实现

但lambda可以利用参数描述提供相当于注释说明的作用，代码更易读和维护，即使表达式稍长一些。另外极限情况下，如果方法应用时所在的类名过长的话，lambdas反而代码更短。对比`service.execute(GoshThisClassNameIsHumongous::action);` 和 `service.execute(() -> action());`

需要注意的是，尽管lambda可以替换绝大多数方法应用的场景，但[JLS, 9.9-2](https://docs.oracle.com/javase/specs/jls/se9/html/jls-9.html#jls-9.9)中描述的场景无法做到——

>A generic function type for a functional interface may be implemented by a method reference expression ([§15.13](https://docs.oracle.com/javase/specs/jls/se9/html/jls-15.html#jls-15.13 "15.13. Method Reference Expressions")), but not by a lambda expression ([§15.27](https://docs.oracle.com/javase/specs/jls/se9/html/jls-15.html#jls-15.27 "15.27. Lambda Expressions")) as there is no syntax for generic lambda expressions.

泛型类型的函数接口可以由方法引用实现，但不能被lambda表达式实现，因为没有针对语法支持泛型lambda表达式。

方法引用大部分是静态方法，但也有其他类型，一共五种，详情如下—— 


|Method Ref Type|Example|Lambda Equivalent|
|---------|---------|---------|
|Static|Integer::parseInt|str -> Integer.parseInt(str)|
|Bound|Instant.now()::isAfter|Instant then = Instant.now();t -> then.isAfter(t)|
|Unbound|String::toLowerCase|str -> str.toLowerCase()|
|Class Constructor|TreeMap<K,V>::new|() -> new TreeMap<K,V>|
|Array Constructor|int[]::new|len -> new int[len]|

### Item 44: Favor the use of standard functional interfaces

如果Java库提供了标准的函数接口，优先使用。尽管`java.util.Function`包下的**43**个函数接口不可能都记得住。但记住下面六个基础的接口，其他部分可以推到出来——

- 基础的`Operator`函数接口表示：参数类型和返回类型一致，包括一元操作`UnaryOperator<T>`和二元操作`BinaryOperator<T>`
- `Predicate`函数接口表示：接受一个参数，返回boolean结果
- `Function`表示：参数和返回值类型不同
- `Supplier`表示：没有参数，返回(“提供”)一个结果
- `Consumer`表示：接受参数，不返回结果

列表总结下——

|Interface|Function Signature|Example|
|---------|---------|---------|
|UnaryOperator<T>|T apply(T t)|String::toLowerCase|
|BinaryOperator<T>|T apply(T t1, T t2)|BigInteger::add|
|Predicate<T>|boolean test(T t)|Collection::isEmpty|
|Function<T,R>|R apply(T t)|Arrays::asList|
|Supplier<T>|T get()|Instant::now|
|Consumer<T>|void accept(T t)|System.out::println|

针对原始类型int, long和double，以上六个基础函数接口都可以演化出对应的变种，在方法名前加上前缀即可，例如，`IntPredicate`，接受int值为参数；`LongBinaryOperator`，接受两个long参数，返回long值。除了Function类型的变种，其他类型变种都不需要指定参数类型。Function类型的变种需要指定参数类型，例如`LongFunction<int[]>`接收long类型参数，返回int数组。这样多出来**18**个。

针对原始类型，另外还有**9**个Function接口的变种。因为Function接受的参数和返回值不同——否则就是`UnaryOperator`函数接口了

- 当参数和返回值都是原始类型时，添加`SrcToResult`的前缀即可，例如`LongToIntFunction`(三个基础类型两两转换，共6个)
- 但返回值不是基础类型时，添加`SrcToObj`前缀，例如`DoubleToObjFunction`，包含3个

"双参数"类型的变体一共还有**9**个——

3个“双参数”类型的函数接口版本，即`BiPredicate<T, U>`，`BiFunction<T, U , R>`，`BiConsumer<T, U>`

3个返回原始类型的`BiFunction`变体，即`ToIntBiFunction<T, U>`， `ToLongBiFunction<T,U>`，`ToDoubleBiFunction<T,U>`

3个Consumer的“双参数”版本，参数一个为对象，一个为原始类型，即`ObjDoubleConsumer<T>`，`ObjIntConsumer<T>`，`ObjLongConsumer<T>`

以上一共42个函数接口，最后一个是`BooleanSupplier`属于`Supplier`的变体返回boolen类型，这是唯一显示声明boolean类型的标准函数接口。当然`Predicate`类型的总是返回boolean类型

大部分函数接口都是用来处理原始类型的，不要试图使用这些函数处理基本包装类型，尽管可以生效，但违反了item 61的原则“原始基础类型优于基础包装类型”`prefer primitive types to boxed primitives`

那么什么情况下需要写自定义的函数接口呢？如果上面提供的无法满足时，当然要自定义；但有些时候即使上面的可用，也需要自定义一个新的。

例如`Comparator<T>`。本质上它与`ToIntBiFunction<T,T>`是等价的。并且引入前者的时候，后者已经存在。

- 如果广泛使用并且名称描述了更准确的含义
- 有强约束条件(strong contract)
- 默认方法可以被广泛应用

满足以上条件时就可以创建自定义的函数接口。同时使用注解`@FunctionalInterface`进行声明。

最后需要强调一点：不要提供可以在相同位置调用不同函数接口的重载方法。例如`ExecutorService`的`submit`方法，可以同时支持`Callable<T>`或`Runnable`——不是一个好设计。


### Item 45: Use streams judiciously

谨慎使用流处理。流处理API在Java 8中引入以更好地进行批处理（串行或并行地）。这些API提供了两个关键的抽象：

- 流，`stream`表示有序的一组数据元素（可以是无限的，也可以是有限的）
- 流操作`stream pipeline`表示对这些元素的多重计算

流可以来这集合，数组，正则表达式等。流中的元素可以是引用类型也可以是基础类型（支持int， long， double）

流操作包含留个或多个中间操作（intermediate operations）和一个最终操作（terminal operation）。每个中间操作都对元素做某种变形，类似映射到某个函数；或进行符合某种条件过滤中间操作总是将流装换到另外一个流中，后一个流的元素可能跟原始流中的相同（进行过滤操作），也可能不同（进行重新映射）。最终操作获得最后的结果，利润存到集合中或返回某个元素，或进行打印。

流处理是“懒式”操作：直到最终操作调用前不会进行处理，同时与最终结果无关的元素都不会参与计算。这也是流可以是无效流的原因。没有最终操作的流处理是空指令(no-op)，相当于没有执行。

同时流处理也是“流畅”的。允许多个操作合并到一个表达式中进行链式操作。通常流处理是串行的，出发调用了并行操作的方法。

实际使用时要谨慎。正确使用可以使程序更短更清晰；使用不当讲让程序更难懂更难维护。没有强制的规则，以实际场景为准。

例如下来程序，从文件中读取单词，并将所有的单词按照“同字异序”（anagrams）规则添加到map中。例如“staple”和“petals”就是同字异序字，因为都由“aelpst”(按照字母顺序排列，成为"alphagram"）组成。

```
// Prints all large anagram groups in a dictionary iteratively
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next(); 
                groups.computeIfAbsent(alphabetize(word),
                    (unused) -> new TreeSet<>()).add(word);
            } 
        }
        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    } 
}
```

其中的`computeIfAbsent`方法是Java 8中引入的流处理方法：在map中寻找当前原始对应的key，如果已经有，返回当前key对应的value值；如果没找到，用当前元素作为参数执行给定的函数以计算出新的key值，再返回key值对应的value。

如果使用下面的流处理方式完成相同的功能——(不要担心如果你觉得很难懂，需要流处理专家才能理解- -)

```
// Overuse of streams - don't do this!
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        } 
    }
}
```

正常版本应该类似这样——需要对参数进行有意义的命名；同时提取有意义的函数名

```
// Tasteful use of streams enhances clarity and conciseness
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize) .forEach(g -> System.out.println(g.size() + ": " + g));
        } 
    }
    // alphabetize method is the same as in original version
}
```

其中的`alphabetize`本身也可以进行流式操作，但由于Java缺少针对char类型的原始char流。如果执行`"Hello world!".chars().forEach(System.out::print);`打印出来的将是一组数字，需要进行强制转换`"Hello world!".chars().forEach(x -> System.out.print((char) x));`。通常不应该使用流式操作处理char类型值。

尽管理论上所有的循环操作都可以转换为流式操作，但不要着急那么重构。`code block`和`stream`各擅胜场。

- 在块操作中，你可以读取或修改任意一个局部变量；但在lambda里，只能获取到最后一个变量的信息并且不能操作任何局部变量
- 块操作中，可以在方法中任意位置结束，中断，继续或抛出任意想检查的异常。lambda中无法实现

下列场景更适合流处理——

- 对顺序的元素做统一的转换
- 过滤一组元素
- 对一组元素应用单一的操作（例如添加，计算最小值，连接操作等）
- 对元素进行分组
- 搜索满足某种标准的元素

另外一个流处理的不足之处是：stream无法在不同的pipeline阶段获得中间状态的对应元素。一旦你将某个值映射到字典，原始的值就不在了。

写程序打印最小的二十个“梅森素数”(Mersenne primes)：如果`2^p-1`格式的正整数，其中p是素数，同时`2^p-1`也是素数，那么这个数就被成为“梅森素数”。


```
static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}

public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

- 首先，获取到所有的素数（无限）流。
- 假设已经静态导入了TWO`import static org.bouncycastle.math.ec.ECConstants.TWO;`
- 其中的魔数50用来控制“概率素性检测”（Probabilistic Primality Test）
- 限制获取20个原始
- 最后进行打印

假设我们需要获取每个梅森素数的幂数值`P`，这个值只在最初的stream中存在，所以在最终的stream中是无法获取到的。尽管这个例子中可以很幸运地在最终元素中获取到`.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));`

假设要构造一组牌，假设牌（Card）是封装了大小（Rank）和花色（Suit）的一组实例对象。大小和花色是枚举类型。数学上，这被称为“笛卡尔积”（Cartesian product），通常的做法——

```
// Iterative Cartesian product computation
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```

采用流处理方式的做法——

```
// Stream-based Cartesian product computation
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit ->
                    Stream.of(Rank.values())
                    .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```

根据实际情况选择你所要采用的方式。

### Item 46: Prefer side-effect-free functions in streams

Steams不只是一组API，更是基于函数式编程的一种范式。

构造你的流处理器时最重要的是：对每一阶段的计算处理转换都尽可能地成为下一阶段的“纯函数”（pure function）。

纯函数的意思是：结果只取决于输入，不依赖任何可更改的状态，不进行任何状态更新。

**没有副作用**

已这个标准来看，下面的代码就违反了这一范式——只是采用了流式表达的迭代方式

```
// Uses the streams API but not the paradigm--Don't do this!
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    }); 
}
```

实际上，`forEach`方法是最终操作方法中能力最低的一个，显示进行迭代，无法并行。通常只应该用来报告流计算的结果（打印，或者添加到集合中），不应该进行计算动作。

正确的写法为——

```
// Proper use of streams to initialize a frequency table
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words
            .collect(groupingBy(String::toLowerCase, counting()));
}
```

为了使用流计算，必须要掌握`Collector`，尽管对应的API有点吓人——包含了39个方法，有些方法甚至包含5个参数。

好消息是不需要探究内部的复杂性，就可以推断出大部分API的作用。我们可以将Collector看做封装了“消减策略”（reduction strategy）的不透明对象。

消减的意思用集合器完成是将stream中的对象集合到一个单一的对象，这个对象称为“Collection”集合。

有三种集合方式：`toList()`，`toSet()`，`toCollection(collectionFactory`。

例如收集频率表中的前十集合——

```
// Pipeline to get a top-ten list of words from a frequency table
List<String> topTen = freq.keySet().stream() 
        .sorted(comparing(freq::get).reversed()) 
        .limit(10)
        .collect(toList());
```

大部分集合器的目的是最终映射为一个map。其中的每个元素包含一个key和一个value。最简单的map收集器接受两个参数`toMap(keyMapper, valueMapper)`，例如我们在item 34中提到的——

```
//Using a toMap collector to make a map from string to enum
private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                Collectors.toMap(Object::toString, e -> e));
```
这要求每个元素映射的key值都是唯一的，如果有多个元素映射到同一个key，将抛出`IllegalStateException`异常。较复杂的`toMap`方法，或者`groupingBy`方法可以处理这种异常，提供一个`merge function`合并函数。任何映射到相同key的元素都这些曾合并函数，例如覆盖（只取最后一个）

更多介绍参考`java.util.stream.Collectors`。最重要的方法包括：`toList, toSet, toMap, groupingBy, joining.`





