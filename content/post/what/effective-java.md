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

## 6 Lambdas and Streams

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


### Item 47: Prefer Collection to Stream as a return type

需要返回最终元素时，根据需要返回不同的集合样式即可。包括`Collections, Set, List, Iterable, Array`。

- 只需要执行for-each循环，不包含集合方法例如`contains(object)`时，返回`Iterable`
- 如果是基础类型并且对性能有要求，使用数组

引入Stream之后，这种情况需要谨慎了——

```
// Won't compile, due to limitations on Java's type inference
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // Process the processs
}
```

上面的代码实际上无法编译通过，需要进行强制转换——

```
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcesses()::iterator) {
    // Process the processs
}
```
更好的方式是提供一个适配器函数，通常成对出现，相互转换——

```
// Adapter from Stream<E> to Iterable<E>
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

// Adapter from Iterable<E> to Stream<E>
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}

```
然后使用——

```
for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // Process the process
}
```
提供的API服务对这两种情况都需要考虑。Collection接口既是Iterable的子类又包含stream方法，所以这种类型可以同时满足上面两种情况。如何返回的数据不大，可以使用`ArrayList`或`HashSet`直接返回。

如果返回的数据过大，但可以简明地进行表达，可以返回特殊的集合。例如返回一个指定Set的幂Set：如果Set包含n个元素，对应的幂Set包含`2^n`个子set。`{a,b,c}`的幂set对应：`{{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}`

但数据不是很大时，可以利用`AbstractList`实现自定义的集合——（只需要实现`size`和`contains`方法即可）

```
// Returns the power set of an input set as custom collection
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }
            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }
            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```
因为Collection包含了一个`size`方法，最大值为`Integer.MAX_VALUE`，`2^31-1`。更通用的实现是返回`Stream`类型

算法思路——

- 包含第一个元素的list称为前缀list，例如 `{a,b,c}`的前缀list包含：` (a), (a, b), (a, b, c)`
- 包含最后一个元素的list称为后缀list，`{a,b,c}`的后缀list包含：`(a, b, c), (b, c), (c)`
- list的所有子list实际是：前缀list的所有后缀list；或者说后缀list的所有前缀list。

```
// Returns a stream of all the sublists of its input list
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }
    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }
    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

相当于——
```
for (int start = 0; start < src.size(); start++)
    for (int end = start + 1; end <= src.size(); end++)
        System.out.println(src.subList(start, end));
```

实现到一个stream中——
```
// Returns a stream of all the sublists of its input list
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
        .mapToObj(start ->
            // start + (int)Math.signum(start) 可以产生空list？
            IntStream.rangeClosed(start + 1, list.size())
                .mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
}
```

上面两个都不包含空list，需要手动添加上。`start + (int)Math.signum(start)` 代替`start +1`可以产生空list？还不太明白


### Item 48:  Use caution when making streams parallel

一句话总结：非必要，不要对streams做并行处理。

因为只需要对streams添加`.parallel()`就可以触发并行处理，但并行处理依赖的限制条件比较苛刻，条件不当时，这个调用不但不能提速，还可能导致灾难性结果(CPU飙升，进入死循环假死状态)。

通常如果stream来自`Steam.iterate`或者中间操作包含了`limit`，并行操作无法提速性能。

来自`ArrayList`, `HashMap`,`HashSet`, `ConcurrentHashMap`实例，`int range`，`long range`的Stream可以并行处理以提高性能。因为这些元素可以很容易地分割为任意大小(通过`spliterator`方法)。

另外一个原因是这些元素有很好的“局部引用性”(locality of reference)——元素顺序存储在内存中。否则并行线程需要处于等待状态，等待元素加载到内存中

如果最终操作的时长占总时长的比例较高，并行处理的效果也比较有限。`reductions`类型的最终操作——例如`min, max, count, sum`——比较适合并行操作。

如果自定义Stream，Iterable或者Collection类型对象，提高并行计算需要重写`spliterator`方法并进行严格测试，确保并行可以提供性能。

PS：维护数百万行代码并且重度使用streams的同事(who?)发现只有数个地方适合并行处理stream对象- -||

当然，满足条件时，并行处理将获取接近线性的处理速度。在机器学习和大数据处理领域需要这种速度

PS2： 一个光明的结尾~ 下列中可以使用并行处理，有效提高处理速度。处理`10^8`的数字从31秒提高到9.2秒

```
// Prime-counting stream pipeline - benefits from parallelization
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
            //.parallel() 
            .mapToObj(BigInteger::valueOf)
            .filter(i -> i.isProbablePrime(50))
            .count();
}
```

## 7 Methods

如何处理参数和返回值；如何设计方法签名；如何给方法注释——同时适用于构造函数和方法。

### Item 49: Check parameters for validity

大部分方法或构造函数都对参数有一定的限制，应该在一开始就对参数限制做检查，避免因参数不合格导致的后续异常，异常导致的异常将使得后续排除更复杂。——这也违反了“错误原子性”（failure atomicity）

对应public或protected的方法，使用JavaDoc注解`@throws`添加抛出的异常描述。通用的异常可以在class级别进行注释，方法上可以添加更具体一些的异常描述。例如——

```
/**
* Returns a BigInteger whose value is (this mod m). This method
* differs from the remainder method in that it always returns a
* non-negative BigInteger.
*
* @param m the modulus, which must be positive
* @return this mod m
* @throws ArithmeticException if m is less than or equal to 0
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("Modulus <= 0: " + m);
    ... // Do the computation
}
```

对于空指针可以使用Java 7中引入的`Objects.requireNonNull()`方法进行检查，不需要再进行手动检查。Java 9中引入了范围检查的方法，但没有这个方法方便，暂时不用也罢。

对于私有方法，可以使用`assert`进行参数检查。检查后续的表达为真，如果失败，抛出`AssertionError`。如果启动时不启用检查(使用 -ea 或 -enableassertions 参数启动java应用)，这种方式没有任何额外的开销。

这一规则的例外是：如果检查成本过于昂贵或不实际时，不做入参检查。例如对List进行排序操作时。

如果计算时发出异常，抛出的异常于注释不符合时，应当做异常转换(exception translation)。

武断的限制参数并不是好事，方法的参数应当越通用越好。

### Item 50: Make defensive copies when needed

相比与C或C++，Java算是一门安全语言，没有缓存溢出，数组溢出，野指针之类对问题。尽管如此，也需要做一下防御措施，防止调用方对类对象不可更改性质对破坏。

```
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start the beginning of the period
     * @param end   the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException if start or end is null
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
    // Remainder omitted
}
```

看起来上述类是final级别对，不能修改。但实际上很容易被破坏——

```
 // Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); end.setYear(78); // Modifies internals of p!
```

在Java 8里，可以使用`Instant`或`LocalDateTime`或`ZoneDateTime`等不可变类代替可变类对象Date。

另外一种保护方法是采用“防御性拷贝”(defensive copy)。对每一个构造函数都可变对参数象都先做一次复制——

```
// Repaired constructor - makes defensive copies of parameters
   public Period(Date start, Date end) {
       this.start = new Date(start.getTime());
       this.end   = new Date(end.getTime());
       if (this.start.compareTo(this.end) > 0)
         throw new IllegalArgumentException(
             this.start + " after " + this.end);
}
```

注意先做拷贝再做检查都顺序，并且检查是针对拷贝判断。——这种做法是防止`TOCTOU`(Time-ofcheck/time-of-use)攻击：参数检查和参数复制时间差之间进行攻击（在检查后，复制前进行修改）

另外也不实用Date都clone方法进行复制，因为传递进来都参数有可能是Date的子类，clone方法依然不安全。

上面的防御拷贝力度实际还不够，依然可以被修改——

```
// Second attack on the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); p.end().setYear(78); // Modifies internals of p!
```

需要对访问方法也提供防御复制——

```
// Repaired accessors - make defensive copies of internal fields
   public Date start() {
       return new Date(start.getTime());
}
   public Date end() {
       return new Date(end.getTime());
}
```

这样，Period对象变成真正不可更改类。实际上，针对访问方法是可以使用clone方法对，因为我们确定这里对对象是Date本身，而不是子类。

防御拷贝不知针对可变类，任何当你需要对客户端提供对引用保存到内部结构体时，都需要考虑这一引用是否会被修改，修改是否可被接受。

总结：最好是使用不可变类；使用可变类需要考虑防御拷贝；如果只是包内使用或防御拷贝代价太大，不做防御拷贝时，做好对应的注释说明。

### Item 51: Design method signatures carefully

方法签名的小建议：

- 方法名应当遵守命名规范（item 68中细谈），包内规范统一，可参考Java库的方法名命名思路，如此大规模下一致性依然足够
- 不要提供太多的方法

- 避免参数列表过长，尤其应当避免类型相同的参数过多

三种缩短参数列表的技巧——

第一，将方法重新差分为多个不同方法（同时尽量避免造成方法过多）

第二，参数列表对象化。创建帮助类以进行参数传递，

第三，结合上述两种，使用Builder方式构造方法调用，尤其当有些参数是可选项的时候


- 接口优于具体类。例如，使用Map，而不是HashMap。前者还可以传递不同的Map，比如TreeMap，ConcurrentMap等。

- 使用两元素等枚举类代替boolean参数。意义更明确并且更容易扩展。

### Item 52: Use overloading judiciously

谨慎使用方法重载。

```
// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

期望分别打印出来`Set, List, Unknown Collection`，实际上三次打印出来等都是`Unknown Collection`。

重载方法纠结调用哪个是在编译期决定的，三次循环在编译期都识别为`Collection<?>`，尽管在运行时三次循环都对象分别不同，但不影响都调用了相同都重载方法。

这有点反直觉。因为重载方法都选择是静态的（编译期）；重写方法的选择是动态的（运行时）。下面是运行时对重写方法的调用——（跟你期待的一样，重写方法的调用采用“最具体”原则，子类的重写方法优先于父类）

```

class Wine {
    String name() { return "wine"; }
}
class SparklingWine extends Wine {
    @Override String name() { return "sparkling wine"; }
}
class Champagne extends SparklingWine {
    @Override String name() { return "champagne"; }
}
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = com.sun.tools.javac.util.List.of(
                new Wine(), new SparklingWine(), new Champagne());
        for (Wine wine : wineList)
            System.out.println(wine.name());
    }

}
```

上面的`classify`方法想打印期待的结果，应当修改为——

```
public static String classify(Collection<?> c) {
       return c instanceof Set  ? "Set" :
              c instanceof List ? "List" : "Unknown Collection";
}
```

重写是一种规范；重载却是一种例外。重载会导致用户的期待混乱，应当避免使用重载。

纠结重载到哪一步会引起混乱还没有定论。但保守的使用原则是：不用重载相同参数个数的方法；如果方法使用变长参数，永远不要重载。

可以理由不同的命名规则代替重载。例如`ObjectOutputStream`类中针对基本类型的write方法，分别命名为`writeInt(int)`，`writeBoolean(boolean)`，`writeLong(long)`。

对于构造函数来说，无法重命名，可以使用静态工厂模式代替构造方法模式。

相同参数个数的重载方法中，如果参数类型“极端不同”(`radically different`)时，问题也不大(因为无法在两种不同类型直接强制转换)。这种情况下，重载的调用会根据运行时的参数类型进行判断（选择对应的重载方法），而不会受限于编译期的类型判断。例如`ArrayList`包含一个参数为int的构造函数，一个类型为Collection的构造函数。

在Java 5之前，所有的基本类型都是“极端不同”的，但由于自动装箱的引入，可能会引起问题，例如——

```
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```
期望的打印结果：`[-3, -2, -1] [-3, -2, -1]`  
实际的打印结果：`[-3, -2, -1] [-2, 0, 2]`

因为`Set`包含重载方法`remove(E)`，int被自动装箱为Integer，与期望相同；

`list.remove(i)`调用时选择了`remove(int i)`的重载方法（而不是期望的`remove(E)`方法），移除对应index下的值，三次操作之后，原始的`[-3, -2, -1, 0, 1, 2]`就变成了`[-2, 0, 2]`，如果希望得到期望的值，调用需要修改为`list.remove((Integer) i);`或`list.remove(Integer.valueOf(i));`

范型和自动装箱的引入破坏了`List`类的接口。也提醒要跟谨慎地使用重载方法。

Java 8中引入的Lambda表达式和方法引用使得方法重载的调用机制更加负载。例如——

```
new Thread(System.out::println).start();

ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

第一个调用可以编译通过，第二种调用无法通过编译。原因是`submit`方法有一个重载方法`submit(Callable<T> c)`，而`Thread`的构造函数没有。

尽管所有的`println`方法重载都是void类型，不可能是`Callable<T>`类型。但重载算法不是这样工作的。

实际上，如果`println`方法如果没有重载，上面的`submit`方法调用执行将是合法的。正是方法引用的重载(println)和调用方法(submit)的组合导致了重载算法没有按照期待的结果执行。

技术上将，`System.out::println`属于一种“不精确方法引用”`inexact method reference`[JSL, 15.13.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13.1)

“如果参数表达式中包含了隐式的lambda表达式或不精确的方法引用，则适用性测试时可以忽略，因为直到类型被选定之前无法确定其含义“——针对编译器开发者的这段表示如果暂时无法理解也不用担心。

只需要记住：如果方法在相同参数位置接受不同的函数接口时，不用重载此方法。

如果必须要违反这一原则时，确保不同的重载方法的执行逻辑一致。例如String类中的`contentEquals`方法——

```
// Ensuring that 2 methods have identical behavior by forwarding
   public boolean contentEquals(StringBuffer sb) {
       return contentEquals((CharSequence) sb);
}
```

### Item 53: Use varargs judiciously

变长方法(Varargs methods)又被称为“变量变长方法”(variable arity methods)，参考[JLS, 8.4.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.1)。指方法接受零个或多个相同类型的参数。

处理时，首先会创建一个长度与变量个数相等的数组，然后将变量赋值给数组(变长方法每次都有这个开销)。简单示例——

```
// Simple use of varargs
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum; 
}
```

有时候方法通常需要的是一个或多个变量，需要先检查变量个数，类似——

```
// The WRONG way to use varargs to pass one or more arguments!
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments"); int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min; 
}
```

上述写法的问题：如果没有传入任何参数，运行时才会报错；另外就是……丑，需要显示验证参数，并且无法使用for-each循环，除非将min初始化为`Integer.MAX_VALUE`（也很丑陋）

正确的写法——

```
// The right way to use varargs to pass one or more arguments
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min; 
}
```

另外一种场景：加入95%的情况下参数个数都小于三个的话，考虑重载此方法（避免了数组创建、赋值的开销以提高性能），例如—— 

```
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

实际上，Java库中的`EnumSet`的`of(E)`方法就采用了这种小技巧


### Item 54: Return empty collections or arrays,not nulls

返回空集合，空数组，而不是返回null值。否则调用方始终需要处理null值的场景。

有争议认为null值避免了分配空集合或空数组的性能开销。第一，不建议性能优化需要到这个级别，除非测量验证需要如此；第二，可以返回空数组或结合并且不进行内存分配。(how？)

返回不可变的空集合对象，例如`Collections.emptyList`, `Collections.emptySet`, `Collections.emptyMap`。

对于数组来说也类似，例如——(传递一个零长度的数组)

```
//The right way to return a possibly empty array
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```
如果需要考虑性能，可以优化为不可变数组对象——

```
// Optimization - avoids allocating empty arrays
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

每一次调用都是返回相同都数组。不要进行提前分配，研究证明下面都方式反而会影响性能——

```
// Don’t do this - preallocating the array harms performance!
   return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

### Item 55: Return optionals judiciously

Java 8之前，当某些情况下无法返回一个值时，有两种处理方式：一，抛出异常（由于要保留堆栈信息，代价昂贵）；二，返回null值（调用方需要对其做特殊处理）。两种方式都不完美

Java 8引入了`Optional<T>`类做第三种处理：要么是空值；要么是非空都引用。一个Optional实例属于不可变的集合，可包含绝大多数元素（不包括集合性质的元素）

对于item 30中的例子——

```
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) { if (c.isEmpty())
    throw new IllegalArgumentException("Empty collection");
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    return result;
}
```

使用Optional的方式之后处理为——

```
// Returns maximum value in collection as an Optional<E>
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    return Optional.of(result); 
}
```

使用工厂模式返回合适的Optiona类型即可。注意其中的`Optional.of(value)`中的value不能为null，否则将抛出空指针异常；`Optional.ofNullable(value)`可以接受空值的value（但永远不要返回包含类null值但Optional对象，这完全违背类这个类但目的）

需要流操作但最终动作返回optional类型，使用stream方式重写max方法——

```
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>>
            Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

Optional类提供类不同但API方便调用方进行返回值处理：

- 提供默认返回值的方式 `String lastWordInLexicon = max(words).orElse("No words...");`

- 抛出异常的方式 `Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);` 
注意传入的异常是一个异常工厂而不是实际的异常，这样可以避免创建异常实例的开销，除非实际抛出异常时，才会创建异常实例

- 如果可以确保总会返回值，可以直接进行获取 `Element lastNobleGas = max(Elements.NOBLE_GASES).get();`

- 当获取默认值开销巨大时，可以提供一个`Supplier<T>`用来获取——

```
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

不适合包装在Optional中的类型：Collections，maps，streams，arrays，optionals。很明显，返回`Optional<List<t>>`不如直接返回`List<T>`。

确定方法的返回值可能出现无法获取的情况时才考虑使用Optional；

Optional对象必须要进行创建，初始化，读取值等操作，对性能有严格要求的场景需要进行性能评估；

禁止使用包含装箱后基础类型的Optional对象，这相当于进行了两次“装箱”操作。Java库提供了对应的`OptionalInt`，`OptionalLong`，`OptionalDouble`进行操作

永远不要使用Optional对象做为Map的key，提高了理解负担。类似的不要这集合或数组中使用optional对象做为key、value或元素

是否严格在对象的属性中包含Optional类型？通常不是一种好方式，但有时也是正确的选择：对象的有些字段并不是“必须”的时候，当然可以选择Optional

### Item 56: Write doc comments for all exposed API elements

可用的API必须有注释。通常API文档需要手动维护，与源码的同步是一件苦差事。Java语言提供了Javadoc工具，可以直接从源码获取到文档：使用特殊到“文档注释”(documentation comments)

文档注释规范不是Java语言到正式部分，但已经是事实上的API标准。Java 4(2002年2月)发布的[“如何写注释”](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)文档已经过去十八年，但依然价值巨大。

- Java 5 `{@literal}`, `{@code}`
- Java 8 `{@implSpec}`
- Java 9中引入`{@index}`

以上新引入的javadoc注释不包括在上述文档中（原文档一直未更新）

- 每个暴露的类，接口，构造函数，方法，字段声明都应该包含对应的注释（缺少时，javadoc重复对应的声明做为java文档）
- 文档注释应当简洁描述方法和调用者直接的契约(contract): 包含前提条件，事后场景，副作用等。对于继承的方法例外，后者关注what而不是how的问题

一个完整注释的例子——

```
/**
* Returns the element at the specified position in this list. *
* <p>This method is <i>not</i> guaranteed to run in constant
* time. In some implementations it may run in time proportional * to the element position.
*
* @param index index of element to return; must be
* non-negative and less than the size of this list
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*  ({@code index < 0 || index >= this.size()})
*
*/
E get(int index);

```

说明：  
- 支持HTML标记，例如`<p>`和`<i>`
- `{@code}`标记的作用是：内容使用代码字体进行渲染；对应内容不需要进行HTML解析，例如小于号`<`。还可以将对应的内容再包装到`<pre>`标记中，可以保留换行记录

对于应用于继承的方法的注释示例——

```
/**
* Returns true if this collection is empty.
*
* @implSpec
* This implementation returns {@code this.size() == 0}. 
*
* @return true if this collection is empty
*/
public boolean isEmpty() { ... }

```

说明：  

- 需要使用`@implSpec`注释说明：自用`self-use`场景。描述方法和子方法的契约。
- Java 9中依然会忽略`@implSpec`标记，需要显示启用`-tag "implSpec:a:Implementation Requirements:"`

- 对于HTML特殊符号`<, > &`的注释可以使用`{@literal}`注释，它与`{@code}`类似，只是不采用代码字体进行渲染。
- 使用时需要确保源码效果和文档效果都有可读性。例如`* A geometric series converges if {@literal |r| < 1}.` 注释中实际只是对小于号进行声明，但将整个表达式都做声明保留了源码中的可读性

- 第一句做为方法的概述(summary description)。避免任何不同的两个方法有相同的概述。注意不要让概述中的字符本身截断了句子（默认句号+空格做为概述的结束）。例如——

```
/**
* A college degree, such as B.S., {@literal M.S.} or Ph.D. */
public class Degree { ... }
```

- Java 9中生成的javadoc，右上角的搜索框中输入文字是，会自动生成下拉列表显示当前页面的index；源码注释中的index可以显示的在搜索中成为关键字匹配

```
* This method complies with the {@index IEEE 754} standard.

```

- 范型、枚举类、注解。需要对所有类型进行文档注释

范型示例——
```
/**
* An object that maps keys to values. A map cannot contain * duplicate keys; each key can map to at most one value.
*
* (Remainder omitted)
*
* @param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> { ... }
```

枚举示例——

```
/**
* An instrument section of a symphony orchestra.
*/
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,
    /** Brass instruments, such as french horn and trumpet. */
    BRASS,
    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,
    /** Stringed instruments, such as violin and cello. */
    STRING; 
}
```

注解示例——

```
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to pass.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
    * The exception that the annotated test method must throw
    * in order to pass. (The test is permitted to throw any
    * subtype of the type described by this class object.)
    */
    Class<? extends Throwable> value();
}
```

- 包级别的注释可以直接写在`package-info.java`中，如果采用模块系统，模块级别的注释写在`module-info.java`中

- 如果某个API没有提供注释，Javadoc会自动搜索合适的注释。搜索算法中[JavaDoc指南](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javadoc.html)中有介绍
- 生成的JavaDoc页面可以利用[W3C验证器](https://validator.w3.org/)验证其有效性

## 8 General Programming

本章讨论Java语言的具体细节。包括本地变量；控制结构；类库；数据类型；以及两个语言之外的东西：反射和原生方法；还包括优化和命名规范。

### Item 57: Minimize the scope of local variables

与旧式的C语言不同，Java允许在任何地方进行变量声明。

- 最有效的方式是：在第一次使用前进行变量声明（方便控制变量周期）
- 几乎所有的变量声明都应该进行初始化。例外是当变量初始化可能抛出异常时，在try-catch语句中进行初始化。
- 循环中的变量声明。使用for循环方式优先while循环。前者可以自动限制变量的生命周期，后者扩大了变量周期，随意性可能导致问题。对比——

```
// Preferred idiom for iterating over a collection or array
for (Element e : c) {
    ... // Do Something with e
}

 // Idiom for iterating when you need the iterator
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e and i
}

# 出现复制错误，for-loop方式下不会出现
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
    doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator(); while (i.hasNext()) { // BUG!
       doSomethingElse(i2.next());
   }

```

使用for-loop循环的另外一个好处是代码更简洁，同时可以确保初始化判断只执行一次，例如——

```
for (int i = 0, n = expensiveComputation(); i < n; i++) {
       ... // Do something with i;
}
```

- 最后一条——短小，聚焦

### Item 58: Prefer for-each loops to traditional for loops

使用for-each类型的循环代替传统的循环方式。

```
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // Do something with e
}

// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
    ... // Do something with a[i]
}
```

传统的循环中有很多琐碎代码，迭代器和index都不是必须的。for-each循环被官方称为“增强的循环”(enhanced for statement)，没有性能损失。

```
// The preferred idiom for iterating over collections and arrays
for (Element e : elements) {
    ... // Do something with e
}
```

在嵌套循环中，增强循环类型更有优势，下面都传统循环方式包含了bug——

```
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
    NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next()));
```
内部循环对`i.next()`的调用超过了期待。很可能抛出`NoSuchElementException`的异常。应当在外层循环保留循环的值，再传递到内部循环中——

```
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}  
```

使用for-each循环方式，代码更简洁且不会出现上面的问题——

```
// Preferred idiom for nested iteration on collections and arrays
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

增强循环的限制——

- 有损过滤(Destructive filtering)。如果需要移除集合中的特定元素，需要使用传统的index循环方式
- 变换(Transforming)。需要对集合中的元素进行替换时需要使用index循环方式
- 并行迭代(Parallel iteration)。使用传统循环方式

事实上，任何实现了`Iterable`接口的对象都可以使用for-each增强循环方式。尽管实现自己都`iterator`有点麻烦。

```
public interface Iterable<E> {
    // Returns an iterator over the elements in this iterable
    Iterator<E> iterator();
}
```

### Item 59: Know and use the libraries

```
// Common but deeply flawed!
static Random rnd = new Random();
static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```
上例中有三个隐藏问题：
- 如果n是2的低次幂，很快随机数就会重复
- 如果n不是2的幂次方，有些随机数返回的概率更大，尤其当n的值较大时（下面的例子说明）
- 极少数情况下，random方法将出现失败情况，返回值超出特定范围（因为使用`Math.abs`方法返回正数，如果`netxInt()`返回了`Integer.MIN_VALUE`，`Math.abs`也返回`Integer.MIN_VALUE`，此时余数方法`%`将返回一个负数。假设n不是2的幂次方，此时当前方法将必然失败）并且很难复现


```
public static void main(String[] args) {
    int n = 2 * (Integer.MAX_VALUE / 3);
    int low = 0;
    for (int i = 0; i < 1000000; i++)
        if (random(n) < n/2)
            low++;
    System.out.println(low);
}

```

要写出一个好的随机数生成方法，需要了解不少以下知识才行——

- 伪随机数生成器(pseudorandom number generators)
- 数论(number theory)
- 整数二进制补码的数学原理(two's complement arithmetic)

幸运的时你也可以不需要了解，直接使用库方法`Random.nextInt(int)`即可。

使用库方法的五大好处——

- 第一，利用专家知识和经验

上述库函数有高级算法工程师花费很多时间设计、实现、测试验证并经过专家评审以确保正确，即使出现问题也会很快发布修复版本。

现在大部分场景下的随机数相关使用`ThreadLocalRandom`类；并行计算场景下使用`SplittableRandom`

- 第二，不需要浪费时间实现只跟自身场景相关的算法，更应该关注在业务点上

- 第三，随着时间推移，库方法性能会进一步提高（因为它们在工业级别大量使用，维护的组织有充分动力进行优化。实际上，Java平台库这些年来的性能有显著提示——

- 第四，功能会越来越丰富
- 第五，让代码更主流。更容易阅读，维护，重用。

但很多程序员不去用，为什么？因为不知道。每次大版本发布都会有大量功能加入。

需要掌握的基础库——

- `java.lang`
- `java.util`
- `java.io`
- 集合框架`the collections framework` 
- 流处理`the streams library`
- `java.util.concurrent`

基础库无法满足要求时，优先考虑优质第三方库，例如`google guava`

最后才考虑重复造轮子。（当然，进行研究学习时，鼓励造轮子）

### Item 60: Avoid float and double if exact answers are required

float和double是为工程科学计算而设计的，计算时使用"二进制浮点计算"(binary floating-point arithmetic)，获取的是近似值。

需要获取精确值时，不要使用。尤其是涉及到货币计算时使用这两种类型尤其有问题。参考下例——

```
System.out.println(1.03 - 0.42);
#  0.6100000000000001

System.out.println(1.00 - 9 * 0.10);
# 0.09999999999999998
```

计算：一美元买10美分、20美分、30美分……的东西(每个比前一个贵10美分)，一共可以购买多少个？

采用double计算时——

```
// Broken - uses floating point for monetary calculation!
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + " items bought.");
    System.out.println("Change: $" + funds);
}
```

提示只能购买3个，还剩`0.3999999999999999`。显然是错误的结果。涉及到货币计算时，使用`BigDecimal`，`int`或者`long`。例如使用BigDecimal的方式（使用String方式构造函数）——

```
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");
    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00"); 
    for (BigDecimal price = TEN_CENTS;
            funds.compareTo(price) >= 0;
            price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + " items bought.");
    System.out.println("Money left over: $" + funds);
}
```

这种方式有两个问题：一是写起来复杂，二是计算速度慢。使用int的方式——

```
 public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + " items bought.");
    System.out.println("Cash left over: " + funds + " cents");
}
```

- 精度小于8位，使用int
- 精度不超过18位，使用long
- 精度超过18位，使用BigDecimal

### Item 61: Prefer primitive types to boxed primitives

Java包含两种类型系统：原始类型，例如`int, double, boolean`；引用类型，例如`String, List`。

每一种原始类型都有对应的引用类型，被称为“装箱原始类型”(boxed primitive)，例如`int, double, boolean`对应的引用类型分别是`Integer, Double, Boolean`。

原始类型和装箱的原始类型主要有三点不同——

- 原始类型只有值；装箱的原始类型还包含与其代表的值不同的“身份”(identitiy)，这表示两个装箱原始类型实例的值可以相同，但有不同的“身份”
- 原始类型只有函数值(functional value)，装箱的原始类型另外还有一个非函数值`null`
- 原始类型比装箱的原始类型更有效率（时间空间效率）

演示一些错误示例——

```
// Broken comparator - can you spot the flaw?
Comparator<Integer> naturalOrder =
    (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
调用`naturalOrder.compare(new Integer(42), new Integer(42))`将得到1而不是0。装箱的原始类型使用`=`操作符几乎总会出错（等号进行“身份/同一性”对比）

使用变量进行自动拆箱——

```
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, j = jBoxed; // Auto-unboxing
    return i < j ? -1 : (i == j ? 0 : 1);
};
```

第二个——

```
public class Unbelievable {
    static Integer i;
    public static void main(String[] args) {
        if (i == 42)
            System.out.println("Unbelievable");
    } 
}
```

将抛出空指针异常。因为装箱的原始类型默认值是null，类似于引用类型的默认出生值。此例中声明为原始类型int则不会抛出异常

第三个——

```
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```

由于sum被声明为装箱的原始类型，每一次循环都会进行自动拆箱装箱操作，导致性能极大降低。

那么什么情况下使用装箱的原始类型(而不是原始类型)呢？

- 做为集合的元素、key或value时
- 在参数化类型和参数化方法中只能使用装箱的原始类型，例如必须使用`ThreadLocal<Integer>`，不能使用`ThreadLocal<int>`
- 反射调用时必须使用装箱的原始类型

### Item 62: Avoid strings where other types are more appropriate

String被设计用来表示“文本”。如果对象时间表示的含义不是“文本”，强烈建议不要使用String类型。

当数据来自文件、网络、键盘输入时，形式都是文本。通常人们将此数据一直保持为String类型。但如果数据实际但类型是数值、是否、枚举含义时，应当将其做对应的转换。

- 值类型：int, float, BigInteger
- 枚举类型
- 聚合类型(aggregate type)：当对象包含多个组件时，不要使用String类型对其进行表示，例如——

```
// Inappropriate use of string as aggregate type
String compoundKey = className + "#" + i.next();
```

- 素质(capability)，或称为“不可更改的key”(unforgeable key)

有时string被用来进行功能访问，例如对本地线程提供变量设施(variable facility)

```
// Broken - inappropriate use of string as capability!
public class ThreadLocal {
    private ThreadLocal() { } // Noninstantiable
    // Sets the current thread's value for the named variable.
    public static void set(String key, Object value);
    // Returns the current thread's value for the named variable.
    public static Object get(String key);
}
```

由于String使用共享的全局命名空间，上例中如果多个线程无意中使用了相同的String做为key，将导致意外。

修复方式——

```
public class ThreadLocal {
    private ThreadLocal() { }  // Noninstantiable
    public static class Key {  // (Capability)
        Key() { }
    }
    // Generates a unique, unforgeable key
    public static Key getKey() {
        return new Key();
    }
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```
这种情况下，也不需要再使用静态方法，不需要使用key表示本地变量，变量本身就属于当前线程——

```
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

上面情况的问题是使用时还需要对获取到的对象进行转换，进一步的优化——

```
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

这其实就是`java.lang.ThreadLocal`提供的API的简易版本。

### Item 63: Beware the performance of string concatenation

字符串连接操作(`+`)可以方便地将数个字符串连接为一个。如果字符串不多，且大小固定，可以使用这种操作。

n个字符串使用`+`号连接时消耗的时间是n的平方，这是因为string属于不可变对象，当两个字符串连接时，两个内容都需要做拷贝，——所以性能很差

```
// Inappropriate use of string concatenation - Performs poorly!
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++)
        result += lineForItem(i);  // String concatenation
    return result;
}
```

当数值很大时，上面的操作性能非常糟糕。应当使用`StringBuilder`代替String，并且最后事先声明其长度，避免自动扩展导致的性能损失——

```
public String statement() {
StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH); 
for (int i = 0; i < numItems(); i++)
    b.append(lineForItem(i)); return b.toString();
}
```

第一种方法是item的平方级别；第二张是线性级别。

总之，除非不考虑性能，或者字符串不多，否则不要使用字符串`+`操作，使用`StringBuilder`做为替代。

（即使Java 6以来对字符串`+`操作做了大量改进，性能相比`StringBuilder`依然有巨大差距）

### Item 64: Refer to objects by their interfaces

当对应的接口存在时，在参数、返回值、变量声明、字段定义等场景下，都应该优先使用接口类型。例如：

使用——

```
// Good - uses interface as type
Set<Son> sonSet = new LinkedHashSet<>();
```

而不是——

```
// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

当使用接口时，做变更时只需要对对应的声明做不同的实现即可，例如上例中使用`HashSet<>()`代替`LinkedHashSet`。其它对应的代码都不需要做变更。

需要注意的时，如果对应代码使用了某个实现类特有的方法时，做对应的替换时需要确保替换的类实现也有对应的功能实现。

使用接口方式，还可以确保客户端调用的兼容问题。即使将声明和对应的代码都做了对应的修改，但客户端调用时如果依赖修改前的实现，将会引起编译问题。

有时候，需要使用类声明代替接口声明——

- 没有对应的接口类。例如值类型类`String`或`BigInteger`。值类型类很少包含多种不同实现，方法也大多数final类型，很少有对应的接口类。（这种情况下，可以直接使用类声明）
- 使用的框架基于class，没有合适的接口类。`java.io`下的类例如`OutputStream`都属于这种类型。这种情况下，尽量使用对应的抽象类
- 实现接口的类提供了额外的接口中未定义的方法。例如`PriorityQueue`包含一个comarator方法在接口`Queue`中未定义，当程序需要依赖此额外方法时，声明时使用类，而不是接口方式

### Item 65: Prefer interfaces to reflection

反射机制`java.lang.reflect`提供动态访问任意类的能力。任意一个Class对象，可以利用反射访问构造函数`Constructor`，方法`Method`，属性字段`Field`。

即使在包含反射机制的类编译时不存在的类，也可以通过反射进行访问。

不过，反射有以下问题——

- 无法享受编译期检查的好处。如果通过反射访问不存在的方法，只能在运行时抛出异常
- 反射代码通常笨拙而又冗长(clumsy and verbose)
- 反射方法比正常调用低效很多。具体多少很难评估，受多种因素限制，但直观感受：仅调用一个不包含参数的方法就比正常调用慢11倍左右

但有些代码分析工具和依赖注入框架的代码需要用到反射。看一个例子——“接受第一个命令行参数命名的`Set<String>`类型，打印剩余的参数列表”

```
// Reflective instantiation with interface access
public static void main(String[] args) {
    // Translate the class name into a Class object
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) // Unchecked cast!
                Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("Class not found.");
    }
    // Get the constructor
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("No parameterless constructor");
    }
    // Instantiate the set
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("Constructor not accessible");
    } catch (InstantiationException e) {
        fatalError("Class not instantiable.");
    } catch (InvocationTargetException e) {
        fatalError("Constructor threw " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Class doesn't implement Set");
    }
    // Exercise the set
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}
private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}

```

上述例子演示了使用反射的两个缺点——

- 第一，运行期可能抛出六种不同的异常，如果不使用反射，这六种异常在编译期就会被检查出来
- 第二，需要二十五行代码完成实例的创建，正常使用时只需要一行代码 

Java 7引入了`ReflectiveOperationException`这一各种反射异常的超类/父类。可以替换上述异常。

一些复杂的系统编程任务中需要利用到反射机制。如果代码必须应对在编译期未知的类，应当使用反射。

(总之能不用就不用吧- -)

### Item 66: Use native methods judiciously

谨慎使用本地方法。

Java本地调用(JNI: Java Native Interface)允许程序调用C或C++边写的本地方法。历史上看，本地调用主要有三大场景：

- 访问平台特定的设施，例如注册表？(registry)
- 访问本地代码库，例如提供历史数据的历史库
- 本地调用以提高性能

随着Java平台的成熟，已经可用提供之前由宿主平台提供的功能。例如Java 9中引入的进程API，可用直接访问操作系统的进程。如果没有对应的类库，依然可以使用本地调用访问本地库

目前很少建议使用本地调用以提高性能。在早期(Java 3之前)是必要的，但JVM已经比早期快很多，对大多数任务，可以使用Java获得相当的性能。例如，Java 1.1版本发布时，`java.math`中的BigInteger依赖于C语言编写的“多倍精度运算”(multiprecision arithmetic)；当Java 3发布时，算法使用Java重写，获得了比最初原生语言实现更好的性能

这个故事有个忧伤的结尾(sad coda)，BigInteger后续有些变化，针对大数的快速乘法运算，依赖GNU多精度算法库GMP(Multiple Precision arithmetic library)，获取高性能的多倍精度运算时，依然需要依赖本地方法调用

本地调用的缺点——

- 本地方法不够“安全”，参考item 50。本地调用不再对内存损坏免疫
- 本地方法依赖特定平台相关，移植性不好，debug困难
- 使用不当时性能损失更大，因为GC甚至无法跟踪原生的内存使用
- 胶水代码冗长，不易阅读
- 原生代码中的小bug将导致整个应用崩溃

(again, 总之能不用就不用吧- -)


### Item 67: Optimize judiciously 

谨慎优化。人人都应该知道这有三条关于优化的格言——

>More computing sins are committed in the name of efficiency(without neccessarily achieving it) than any other single reason--including blind stupidity. —— William A. Wulf    
>以优化(没有最终实现)为名而导致的计算问题比其他任何一种原因(包括盲目地愚蠢)都要多。—— William A. Wulf

>We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. —— Donald E. Knuth    
>97%的情况下，我们应该忘记小的效率提高：过早的优化是万恶之源 —— Donald E. Knuth

>在优化方面，我们遵守两条原则——  
> 规则一： 不用做(优化)  
> 规则二：（仅针对专家）直到你有了清晰完美无法再优化的解决方案之前，不要做(优化)。  
> —— M. A. Jackson

- 努力写好的程序而不是快的程序。如果好的程序不够快，架构本身会运行你进行优化而不影响系统本身
- 避免设计原因导致的性能限制。不同组件间交互或与外部世界交互的部分一旦确定将很难改变。例如API，底层协议，格式化数据等
- 优化前后要对性能做对比。有可能优化的部分并没有提供总体性能，常识来说，系统会在10%的代码上耗费90%的时间

性能分析工具(Profiler)可以帮助定位应当在哪里进行优化。Java的性能优化比其他语言更难一些，因为“抽象差距”(abstruction gap)更大：程序员写的代码和实际执行的代码（因为虚拟机的存在?）另外，不同平台上的不同实现也可能导致相同的代码有不同的表现，优化需要对不同的平台分别进行测量评估

当然，算法本身导致的性能问题需要先做对应的优化。

### Item 68: Adhere to generally accepted naming conventions

[命名规范](https://docs.oracle.com/javase/specs/jls/se8/html/jls-6.html)通常分为两类：有关印刷的、有关语法上的

印刷相关的包括：包、类、接口、方法、属性、类型变量——通常千万不要违背对应的命名规范

**包**：使用句号并且以组件进行分割。通常使用小写字母，很少包含数字。对外使用的包使用域名的反序形式，例如`edu.cmu`, `com.google`, `org.eff`。  标准库除外，例如以`java`，`javax`开头的。

接下来的包名称应当包含一或两个描述包信息的名字，尽量不超过8个字符，可以使用缩略。例如 `utl`, `awt`

**类或接口**包括枚举类和注解：应当包含一个或多个单词。例如`List`，`FutureTask`。避免使用缩写除非是一些通用缩写，例如`min`和`max`。至于是`HTTPURL`还是`HttpUrl`尚无定论

**方法或属性**：跟类或接口的规范相同，至少第一个字母应当小写。例如`remove`，`ensureCapacity`


|Identifier Type |Examples|
|---------|---------|
|Package or module| org.junit.jupiter.api, com.google.common.collect|
|Class or Interface |Stream, FutureTask, LinkedHashMap, HttpClient|
|Method or Field |remove, groupingBy, getCrc|
|Constant Field |MIN_VALUE, NEGATIVE_INFINITY|
|Local Variable |i, denom, houseNum|
|Type Parameter |T, E, K, V, X, R, U, V, T1, T2|


语法相关的就比较灵活了。多学习，使用通用常识。一般的例子——

`Thread`, `PriorityQueue`, `ChessPiece`——一两个单词的名词实例；  
`Collectors`, `Collections`通用方法；  
`BindingAnnotation`, `Inject`, `ImplementedBy`, `Singleton`注解方法名；  
`append`, `drawImage`表动作的方法名；`isDigit`, `isProbablePrime`, `isEmpy`, `isEnabled`, `hasSiblings`, 表状态的方法名；  
`toString`, `toArray`的变换方法名；`asList`转换类型方法；`intValue`, `getInstance`, `newInstance`等。

## 9 Exceptions

使用得当，异常可以提高程序可读性、可靠性、可维护性；使用不当，效果相反。

### Item 69: Use exceptions only for exceptional conditions

走“正道”，不要“奇技淫巧”。

```
// Horrible abuse of exceptions. Don't ever do this!
try {
    int i = 0;
    while(true)
        range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
```

上面这种就属于“奇技淫巧”，利用异常来中止循环。基于错误的原因(既然虚拟机会检查数组访问的边界，一般循环中的边界检查就不需要了)做出的提升性能的处理。

这种原因有三点错误——

- 第一，异常是为异常场景设计的，Java虚拟机的实现没有动机为此做性能优化
- 第二，在try-catch语句块中的代码会抑制某些虚拟机原本会执行的优化
- 第三，传统的循环中对数组的检查并不是不必要的，需要虚拟机实现会对此做优化

实际上，上面的代码性能反而更差。不但如此，还可能导致其他的异常情况。如果循环体中有其他的数组访问并且出现了越界情况，那么catch将忽略此异常，中止循环…… 

“正道”是异常就用在异常场景下，不要用于流程控制，即使可能提高性能。

```
for (Moutain m: range) 
    m.climb();
```

API设计时也应当如此，有“状态依赖”(state-dependent)的方法应该做对应的判断，而不是利用“状态依赖”做流程控制，例如——

```
// Do not use this hideous code for iteration over a collection!
try {
    Iterator<Foo> i = collection.iterator();
    while(true) {
        Foo foo = i.next();
        ...
    }
} catch (NoSuchElementException e) {
}
```

“正道”方式——

```
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    ...
}
```

### Item 70:  Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

这句也算来自官方[文章](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)——

> If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception.
> 如果客户端预期可以从异常中恢复，使用可检查的异常；如果不能，则使用非检查异常。

Java提供了三类异常类型(Throwable)：检查异常(checked exceptions)、运行时异常(runtime exceptions)、错误(errors)

“检查异常”(checked exception)和“非检查异常”(unchecked exception)是从编译检查来说的。编译期会检查的异常被称为“检查异常”，例如，IO异常`IOException`；编译期不检查的称为“非检查异常”，例如，运行时异常、空指针异常、数组越界异常等

检查异常要求调用方处理此类异常信息(catch或进行忽略)，属于方法API的一部分（特定场景下，此方法会出现跑场异常的场景）。

非检查异常表示程序运行时违反了某些“前置条件”(precondition violation)。例如，数组越界异常`ArrayIndexOutOfBoundsException`表示违反了数组访问的前置条件。

尽管JAVA规范没有要求，但通用的规范是错误(errors)都保留给Java虚拟机使用。表示程序不应当继续执行了（继续执行更大可能带来更大的错误）。所以通常我们实现的非检查异常都应该是运行时异常的子类。

尽管可以定义不属于`Exception`，`RuntimeException`，或`Error`的子类类别的异常(Throwable)，但永远不推荐这样使用。

### Item 71: Avoid unnecessary use of checked exceptions

避免不必要地使用检查的异常。

许多程序员不喜欢使用检查的异常，但如果使用得当，可以增强程序的可读性。如果使用不当/过度使用检查的异常，则会起到反效果。

使用检查的异常时，调用方必须处理此异常：或者在catch语句块中处理；或者继续抛出。哪一种方式都增加了调用方的负担。（Java 8中，此负担更重，因为在流处理中不能包含检查的异常）

如果合理使用对应的API时无法避免异常情况，并且API使用者遇到异常场景时可以采取一些有用的行动——满足这两点时，检查异常的负担是值得的。否则，应当使用非检查的异常。

消除检查异常的最简单方式是返回期待的返回类型的Optional对象，抛出异常时，返回空类型Optional对象即可。这种做法的缺点是此时无法提供额外的信息给调用方。

另外一种方式是将抛出异常的方法拆成两个方法：第一个检查是否抛出了异常，另外一个返回正常情况下的返回值。例如，将下面的例子——

```
// Invocation with checked exception
try {
       obj.action(args);
   } catch (TheCheckedException e) {
       ... // Handle exceptional condition
}
```

拆为以下格式——

```
// Invocation with state-testing method and unchecked exception
   if (obj.actionPermitted(args)) {
       obj.action(args);
   } else {
       ... // Handle exceptional condition
}
```

这种做法在多线程操作时需要注意，对象的状态在`actionPermitted`和`action`之间可能发生变化。

### Item 72: Favor the use of standard exceptions

优先使用标准库的异常。专家程序员和无经验程序员之间的一个显著区别就是前者总是努力让自己的代码有更高的重用性。在抛出异常方面也是如此。

Java库提供了可以覆盖绝大部分场景的异常。使用标准库异常至少有三点好处——

- 第一，让程序更易学习和使用，因为遵守了大家熟悉的规范
- 第二，程序更具可读性，因为没有琐碎的不熟悉的异常
- 第三，更少的异常意味着更小的内存开销和加载耗时

常用异常包括——

`IllegalArgumentException`：参数不合法异常。例如参数要求正数的场景下传递了一个复数参数

`IllegalStateException`：状态无效异常。例如调用方试图在对象初始化之前使用。可以说，每个错误的方法调用都可以最终归结为无效参数或状态。但有更具体的异常时，可以使用更具体异常。例如`NullPointerException`或`IndexOutOfBoundsException`

`ConcurrentModificationException`：设计为单线程场景使用的对象被多个线程同时访问。


`UnsupportedOperationException`：很少被使用。某个对象不支持特点的方法时抛出此异常。例如仅支持追加的List被调用了remove方法。

具体场景下应该使用哪种异常是棘手的问题，因为上面的异常并不是相互排斥的。另外你也可以自定义异常如果上述异常不满足你的场景。

### Item 73： Throw exceptions appropriate to the abstraction

让一个方法抛出与自己所执行的任务不相干的异常会令调用方担心。当方法向上传递底层抛出的异常时会出现这种场景。实际上，这污染了上层API。如果上层API在后续的发布时对应的实现做了变化，将破坏现有的客户端调用。

为避免这种情况，上层方法应当catch下层的异常，让后抛出上层抽象可解释的异常。——这也是通常所说的“异常转换”(exception translation)

```
// Exception Translation
   try {
       ... // Use lower-level abstraction to do our bidding
   } catch (LowerLevelException e) {
       throw new HigherLevelException(...);
}
```

上面的是伪代码表达，来看`AbstractSequentialList`类中get方法中的具体应用—— 

```
/**
* Returns the element at the specified position in this list. * @throws IndexOutOfBoundsException if the index is out of range * ({@code index < 0 || index >= size()}).
*/
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
}
```

异常转换的一种特殊方法被称为“异常链接”。当底层异常有利于对上层的异常做调试分析时，底层的异常被传递给上层异常(通过`Throwable`对象的`getCause`方法提供访问）。例如——

```
// Exception Chaining
   try {
       ... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) { throw new HigherLevelException(cause);
}
```

上层异常的构造函数将异常原因传递给“链式感知”(chaining-aware)的父类构造函数，最终传递给`Throwable`类的对应构造函数。

```
// Exception with chaining-aware constructor
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

如果没有类似的构造方法，还可以通过`Throwable`类的`initCause`方法进行设置。

尽管异常转换优于无脑传播(向上抛出)，但也不能过度使用。最好是能确保底层调用成功，避免抛出异常；次一步的处理是上层方法静默处理底层的异常，对上层方法的调用方隔绝底层的异常，这种情况下可以记录底层的异常信息。


### Item 74： Document all exceptions thrown by each method

对方法抛出异常的描述属于文档注释的重要部分。应当花时间仔细记录每个方法抛出的所有异常场景。

对每一个检查的异常都要使用`@throws`关键分别注释。不要将多个异常场景合并在一起记录，更不要仅仅标记抛出`Exception`，或更糟糕的`Throwable`，这将引起调用方的混乱。（main方法属于例外，因为只有虚拟机调用）

尽管Java语言不要求程序员声明方法可能抛出的非检查异常，更明智的做法是对这种异常也应当注释。因为每个方法实际上都有一些使用的前置条件，注释非检查异常也是声明这些前置条件的最佳方式。

接口方法的非检查异常注释尤其重要，这相当于接口的“通用条件”(general contract)。

对应非检查的异常注释时，不要使用`throws`关键字。这样调用方可以区分哪些是检查的异常场景，哪些是非检查异常场景。

理想情况是每一种异常都进行注释，但现实中可能无法做到。尤其是当前类进行修改的时候。假设某个类调用了另外一个类的方法，前者已经注释了所有的异常，但后者发生了变更增加了新的检查异常。

当某个类的许多方法都抛出某个异常时，可以将异常说明文档添加到类的注释上。


### Item 75: Include failure-capture information in detail messages

当程序由于未捕获的异常导致失败时，系统会自动打印异常的堆栈信息，包含了对应异常的字符串表示(string representation)，调用了异常的`toString`方法。通常包含异常对应的类名和详细信息。

通常程序员或运维工程师关注这些详细信息，用来分析异常发生的原因。所以详细信息应当包含尽可能多的内容。通常应当包含导致异常的所有参数和属性。例如`IndexOutOfBoundsExceiption`应当包含起始位、结束位和当前位，因为任何一个都可以导致这个异常。

另外需要考虑的是安全角度。异常的详细信息中不应当包含密码，私钥之类的信息。

不需要过多描述自身，堆栈信息通常会包括调用过程，通过对应的源码或文档可以找到对应的位置。

异常的详细信息是面向程序员、运维人员的，区别于面向用户的提示信息，后者通常是本地化之后的内容。

需要的异常信息通常在构造函数中就获取到，而不是直接要求一个字符串内容，例如越界异常的一个构造函数如下——

```
/**
* Constructs an IndexOutOfBoundsException.
*
* @param lowerBound the lowest legal index value
* @param upperBound the highest legal index value plus one
* @param index the actual index value
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound,
                                int index) {
    // Generate a detail message that captures the failure
    super(String.format("Lower bound: %d, Upper bound: %d, Index: %d",
                        lowerBound, upperBound, index));
    
    // Save failure information for programmatic access
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```

对详细信息的要求更适合检查的异常，程序员很少会动态访问非检查异常。但作为一个普遍原则，也应该提供(详细信息)。


### Item 76: Strive for failure atomicity

尽量保持错误的原子性——含义是即使发生了错误，也不应当影响对象的已有状态(依然可以正常访问、使用)——尤其是对于检查性的错误异常。

有以下几种方式达到这个效果——

最简单的是设计不可变对象。如果对象是不可变的，自然实现了异常原子性。

对于可变对象，最普遍的做法是进行操作之前先进行参数检查。例如`Stack.pop`的操作——

```
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

如果不做检查，当size为零时依然进行pop操作，将使得size字段变成负数。类似的方式还有：将计算动作放到对象操作之前。

第三种方式是将操作动作施加于对象的副本，成功后再带他原对象。

另外一个很少使用的方式是添加“恢复代码”(recovery code)。通常针对基于磁盘的持久化对象数据。

异常的原子性不是总能实现，例如多线程操作同一个对象时，将可能使得对象处于不一致的状态中；有时为了实现异常原子性将提供复杂性，代价过大。

### Item 77: Don't ignore exceptions

“不要忽略异常”。尽管这个建议看起来显而易见，但却被经常违反，值得再强调一遍。

抛出异常就是为了告诉你对应的方法发生了一些事情。空catch块代码忽略异常如同忽略现实中的火警。任何时候看到空catch块，头脑中都应该响起警灯。

```
// Empty catch block ignores exception - Highly suspect!
try {
    ...
} catch (SomeException e) {
}
```

有些情况下可以忽略异常。例如关闭一个`FileInputStream`，一是你没有更改文件的状态，不需要进行恢复操作；二是已经获取了文件内容，不需要中止进行中的操作。这种情况下，异常可以忽略，最好添加上log信息，如果频繁出现，方便调研。

另外，如果忽略异常，还应当添加合理的介绍说明忽略的合理性。例如——

```
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
    // Use default: minimal coloring is desirable, not required
}
```

“不要忽略异常”。这一建议对检查异常和非检查异常都适用。

## 10 Concurrency

### Item 78： Synchronize access to shared mutable data

`synchronized`关键字用来确保每次只有一个线程执行某个方法或代码块。许多程序员将同步化看做一种“互相排斥”(mutual exclusion)的方法，防止某个线程中不稳定状态的对象被另外一个线程看到。正确使用同步可以确保没有线程可以观测到处于不一致状态的对象。

但这只是一方面。没有同步化，一个线程的改版对另外一个线程来说可能是不可见的。同步不只是确保对象的不一致状态不可见，也确保了进入同步方法或代码块的线程可以看到前一次修改的状态。

Java语言的[内存模型规范](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)确保读写一个非long或double类型的变量是原子操作。换句话说，即使多个线程同时修改这种变量且没有进行同步化，其他线程读取这个变量时依然可以获取到变量值。

你可能听过这种说法：为了提高性能，读写原子性数据时可以省略同步化操作。这是很危险的建议。语言规范确保线程读取变量时不会获取到随意值，但不能确保一个线程写入的值可以被另外一个线程看到。对于“互相排斥”(mutual exclusion)和“可靠通信”(reliable communication)来说，同步化都是必须的。上述内存模型规范中描述了一个线程何时以及如何看到另外一个线程的改变。

Thread类中的stop方法已经废弃，因为其实现不安全，可能导致数据损坏。推荐做法是通过boolean变量控制线程执行流程，因为boolean类型值的读写是原子的，不使用同步化的方式这样处理——

```
// Broken! - How long would you expect this program to run?
public class StopThread {

    private static boolean stopRequested;
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
                    int i = 0;
                    while (!stopRequested)
                        i++;
                });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
期待应用执行一秒，主线程修改变量值之后，后台线程也将结束。然而，这个程序可能永不停止。原因是没有同步化，后台线程无法确保何时可以看到主线程对boolean变量的修改。没有同步化操作，虚拟机可能将项目的代码——

```
while (!stopRequested) 
    i++;
```

转换为——

```
if(!stopRequested) 
    while(true) 
        i++;
```
这种优化被称为"提升"(hoisting)，OpenJDK的虚拟机服务就是这样实现的。这导致了“活性失败”(liveness failure: fail to make progress)。修改的方式之一就是采用同步访问方式——

```
// Properly synchronized cooperative thread termination
public class StopThread {
    private static boolean stopRequested;
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

注意：只有写方法(requestStop)和读方法(stopRequested)都添加了同步关键字，同步化才有保障。只对写方法添加同步是不够的。偶尔只对其中一种方法添加同步可能在某些机器上也能正确执行。

即使没有同步化操作，上述例子中对boolean值的方法操作也是原子性的。对方法的同步化只是确保“可靠通信”的，不涉及到“互相排斥”。尽管循环方法中每次迭代检查的花销很小，但这种情况下可以换一种方式以获取更好的性能：将变量声明为`volatile`类型的，此修饰符无法保证“互相排斥”，但可以确保任意线程读取其修身的字段时都会获取到最近写入的值。

```
// Cooperative thread termination with a volatile field
public class StopThread {
    private static volatile boolean stopRequested;
    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

使用`volatile`关键字时需要小心，下面的方法并不能确保每次获取到的返回值都是不同的序列数字——

```
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

尽管只涉及到对`volatile`类型的字段操作，但由于加操作(`++`)不是原子性的，它包含两个操作：读取字段值；然后再写入一个新值(等于原值加一)。这两个操作之间如果有不同的线程调用这个方法，将导致"安全错误"(safety failure: computes the wrong results)

可以对声明添加`sychronized`关键字以确保多线程之间的调用不会相互干扰，每次调用都可以看到前一次的结果。这时可以移除`volitile`关键字。

或者使用`java.util.concurrent.atomic`包下的`AtomicLong`类。提供了无锁且线程安全的方式。

```
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

这种操作是可接受的：一个线程修改对象数据之后在分享给其他线程，只对分享操作添加同步化。其他线程只多钱对象，不做修改的情况下，是可以不添加同步化操作的。这种对象被称为`事实不可变`(effectively immutable)对象；在不同线程中传递这种对象被称为"安全发布"(safe publication)。可以有多种安全发布方式：存在类初始化时的静态变量中；字段声明为volatile类型；字段声明为final类型；或访问时添加了同步锁；或放到同步集合中(concurrent collection)

### Item 79: Avoid excessive synchronization

上一条讨论不要“锁得不充分”，这一条讨论相反的方面：不要“锁得太过分”。否则可能降低性能，导致死锁甚至不可知的程序行为。

为避免活性失败和安全失败，永远不要在加锁的代码块中将控制权限交给客户端(调用方)，换句话说，不要对设计目的是被重写的方法加锁。从当前类的角度来看，这种方法属于“异形”(alien)，当前类对它们没有控制权。

下面的列子可以演示这种场景——

```
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override 
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }
    @Override 
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // Calls notifyElementAdded
        return result;
    }
}
```

观察者模式：观察者调用`addObserver`订阅通知，调用`removeObserver`取消订阅。两个方法都会传递下面的回调函数——

```
@FunctionalInterface 
public interface SetObserver<E> {
    // Invoked when an element is added to the observable set
    void added(ObservableSet<E> set, E element);
}
```

这个接口在结构上与`BiConsumer<ObservableSet<E>, E>`是相同的，我们使用自定义的名称只是为了语义更明确。简单调用时看起来没有问题——会正常打印出0~99

```
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());
    set.addObserver((s, e) -> System.out.println(e));
    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

如果我们传递一个更复杂一些的`Observer`：打印传递的整数并且如果其值为23时，再删除自己。

```
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23)
            s.removeObserver(this);
    }
});
```

这里使用匿名类而不是lambda函数是因为函数对象需要将自己传递给`s.removeObserver`方法，lambda函数无法访问自身。

我们期望打印到23然后程序停止，但实际上抛出了`ConcurrentModificationException`异常。因为当调用add方法时，`notifyElementAdded`方法正在对观察队列进行迭代，此时又需要移除观察队列中的元素。尽管`notifyElementAdded`方法加了锁，但无法阻止自身的迭代线程对观察set执行回调函数并对其进行修改

现在换一种方式解决取消订阅的逻辑，不在当前线程中直接调用`removeObserver`，而使用executor服务——

```
// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec =
                    Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();
            } catch (ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});
```

此时不会出现异常，但造成了死锁场景。执行器的新建线程中调用`s.removeObserver`方法试图锁住`observers`但无法成功，因为主线程已经获取到锁；但同时主线程又在等待执行器的线程通知它移除observer。

解决方式——(将“异形”移除锁范围即可)

```
// Alien method moved outside of synchronized block - open calls
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

实际上，语言库中的同步集合中的`CopyOnWriterArrayList`是更好的解决方案，几乎是为这种场景定制的。内部实现了ArrayList的一个变体，每次操作都会重新复制一份新的数组。因为内部数组永远不会被修改，所以不需要加锁——

```
// Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers =
        new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

作为一个原则：在加锁的代码块中应当做尽量少的操作。“获取锁，检查状态，需要的话进行状态转变，释放锁”——仅此而已。

上面从正确性上看待加锁问题，现在从性能角度看一下：现在是多核时代，加锁将丧失了并行计算的优势；同时限制了VM的优化。

如果写一个可变类，有两种选择：忽略所有的同步操作，交个调用方客户端处理；或者内部进行同步操作，确保类是线程安全的。Java早期，许多类的实现违反了这一原则，例如`StringBuffer`实例几乎总是被单线程使用，却采用了内部同步的机制，所有后续出现了`StringBuilder`，相当于无锁的`StringBuffer`。

总结——
>In summary, to avoid deadlock and data corruption, never call an alien method from within a ynchronized region. More generally, keep the amount of work that you do from within synchronized  egions to a minimum. When you are designing a mutable class, think about whether it should do its own synchronization. In the multicore era, it is more important than ever not to oversynchronize.  ynchronize your class internally only if there is a good reason to do so, and document your decision clearly (Item 82)

### Item 80: Prefer executors, tasks, and streams to threads

在本书第一版本中包含一个`工作队列`(work queue)的简单实现；当第二版出版时，`java.util.concurrent`包已经加入Java，其中包含了一个基于接口的灵活的任务`执行框架`(Executor Framework)。

- 创建一个工作队列只需要一行代码:`ExecutorService exec = Executors.newSingleThreadExecutor();`，
- 添加执行任务`exec.execute(runnable);`
- 优雅关闭执行器`exec.shutdown();`

实际上还支持很多其他操作。框架提供了不同种类的执行器配置，满足大部分执行场景。如果还不满足，还可以直接使用`ThreadPoolExecutor`类自定义。

选择哪种执行服务需要一些技巧。对应小型轻量服务来说，`Executors.newCachedThreadPool`通常是一个好选择，因为不需要做任何配置就可以很好完成工作。但对于高负荷生产服务就不是好选择。因为缓存线程池接受到任务时不会加入队列，而是立即创建一个线程去执行，如果负荷过高，所有CPU资源将很快被创建线程所耗尽。这种情况下，`Executors.newFiexedThreadPool`是更好的选择，如何需要更精细的控制，可以直接使用`ThreadPoolExecutor`

一个线程服务同时包含了：“需要执行的单元”和“如何执行的机制”。在Java的执行框架中，执行单元和执行机制是分离的。执行单元的抽象被称为“任务”(task)，包含两种：`Runnable`和`Callable`，后者与前者的不同在于其可以返回值并且可以抛出异常。执行单元和执行机制分离后，可以根据场景选择对应的执行机制，然后有执行服务框架完成调度执行即可

关于执行框架的更深一步解释，可参考<<Java Concurrency in Practice>>

### Item 81: Prefer concurrency utilities to wait and notify

对于`wait`和`notify`的使用建议，本书的第一版中做了单独的一个条目，这些建议依然有效并且列于本条目结尾。但这些建议不像曾经那么重要了。因为自Java 5以来，Java提供了高层抽象后的并发库`java.util.concurrent`，这个包括三个部分的内容：执行框架、并发集合、同步器。第一部分上个条目已经结束，本条目关注剩余两个方面。

并发库实现了标准集合接口例如`List`、`Queue`和`Map`，内部实现管理同步请求，提供了高并发支持。在并发集合中显然无法排除并发操作，所以并发集合接口提供了状态依赖的控制操作(state-dependent modify operations)。

例如：Map的`putIfAbsent(key, value)`方法：如果key不存在则添加新key并绑定value，方法本身返回null值；如果key已经存在，则返回key对应的当前值。`String.intern`方法就可以基于此实现——

```
// Concurrent canonicalizing map atop ConcurrentMap - not optimal
private static final ConcurrentMap<String, String> map =
    new ConcurrentHashMap<>();

public static String intern(String s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}
```

`ConcurrentHashMap`甚至对取数据做了优化，例如`get`方法——(下面的intern方法比`String.intern`方法快很多)

```
// Concurrent canonicalizing map atop ConcurrentMap - faster!
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null)
            result = s;
    }
    return result;
}
```

一些并发结合对象扩展了阻塞操作(blocking operation)，一直等待直到任务成功执行完毕。例如`BlockingQueue`扩展了`Queue`的同时增加了几个方法：`take`从队列头取出元素，如果队列为空则一直等待。实际上，执行器的许多实现都基于BlockingQueue实现

同步器`Synchronizer`确保线程依次等待，用来协调他们之间的活动。最常用的同步器是`CountDownLatch`和`Semaphore`，不常用的包括`CyclicBarrier`和`Exchanger`；最强大的则是`Phaser`

“倒数锁”(`Countdown latch`)允许一个或多个线程等待一个或多个线程完成事务处理。唯一的构造函数只需要一个int值表示`countDown`方法要被调用多少次之后，所以等待线程才被允许继续执行。例如，假设创建一个框架计算某个action的并发执行耗时：

- 包括一个执行器执行动作
- concurrency水平表示有多大的并发量
- runnable对象

所有的工作线程做好准备等待计时器线程“发令”执行；最后一个线程执行完任务后，计时线程停止计时。使用`wait`和`notify`实现这个场景将十分复杂，但基于`CountDownLatch`之上却十分简洁——

```
// Simple framework for timing concurrent execution
public static long time(Executor executor, int concurrency,
                        Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);
    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {
            ready.countDown(); // Tell timer we're ready
            try {
                start.await(); // Wait till peers are ready
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown(); // Tell timer we're done
            }
        });
    }
    ready.await(); // Wait for all workers to be ready
    long startNanos = System.nanoTime();
    start.countDown(); // And they're off!
    done.await(); // Wait for all workers to finish
    return System.nanoTime() - startNanos;
}
```

上述方法需要注意的点：

- 需要确保可以创建足够的执行线程，否则这个方法将永远不会停止。线程不足导致的死锁被称为：“线程饥饿死锁”(`thread starvation deadlock`)
- 如果某个线程被抛出了中断异常`InterruptedException`，调用了`Thread.currentThread().interrupt()`方法重新声明中断，正常返回run方法结果
- 计时使用`System.nanoTime`而不是`System.currentTimeMillis`。前者更精确并且不受系统时间的影响
- 只有runnable对象需要消耗一些时间时，最终的计时才有意义。实际上，精确的“微基准”是很难获取到的。最后使用专业的框架例如`JMH`([Java Microbenchmark Harness (JMH)](https://github.com/openjdk/jmh))

上述例子只是一个演示，实际上其中的三个`countdownlatch`对象可以使用一个`CyclicBarrier`或`Phaser`对象代替。代码将更简洁但会更难理解一些。

标准的wait用法：永远只在循环中调用wait方法。同时确保`wait`方法在`notify`或`notifyall`方法调用之前就已经被调。否则无法保证线程会被重新唤醒

```
// The standard idiom for using the wait method
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(); // (Releases lock, and reacquires on wakeup)
        ... // Perform action appropriate to condition
}
```

有了并发框架之后，最后不要再使用原生的`wait`和`notify`方法控制并发流程。

### Item 82: Document thread safety

一个类的方法在并发情况下的行为属于类与其客户端的重要协议。如果没有说明这种情况，调用方不得不自己假设，将可能导致过度同步或同步不足的问题。

为了确保并发场景下的使用，类必须明确说明其支持的线程安全级别。通常有以下几个级别(尽管不够充分，但满足大部分场景)——

- “不可变”(`Immutable`)：类实例相当于常量，不需要外部同步操作。例如`String`、`Long`、`BigInteger`
- “无条件线程安全”(`Unconditional thread-safe`)：类实例是可变的，但内部有充分的同步限制，确保外部并发使用时不需要做同步化操作。例如`AtomicLong`、`ConcurrentHashMap`
- “有条件的线程安全”(`Conditional thread-safe`)：与上一个情况类似，但需要尾部同步化操作以确保并发使用。例如`Collections.synchroinzed`包装返回的集合对象
- “线程不安全”(`Not thread-safe`)：类实例是可变的，并发场景下，调用方必须对每个方法的调用都添加同步锁。例如`ArrayList`、`HashMap`
- “线程恶意”(`Thread-hostile`)：在并发场景下使用不安全，即使外部调用时使用了同步操作

通常关于同步的注释都是类级别的，但如果方法需要额外的说明时，需要对方法进行单独注释。类似`Collections.synchronizedMap`方法的做法——

```
Returns a synchronized (thread-safe) map backed by the specified map. In order to guarantee serial access, it is critical that all access to the backing map is accomplished through the returned map.
It is imperative that the user manually synchronize on the returned map when iterating over any of its collection views:
        Map m = Collections.synchronizedMap(new HashMap());
            ...
        Set s = m.keySet();  // Needn't be in synchronized block
            ...
        synchronized (m) {  // Synchronizing on m, not s!
            Iterator i = s.iterator(); // Must be in synchronized block
            while (i.hasNext())
                foo(i.next());
        }
       
Failure to follow this advice may result in non-deterministic behavior.
The returned map will be serializable if the specified map is serializable.

Params:
    m – the map to be "wrapped" in a synchronized map.
Type parameters:
    <K> – the class of the map keys
    <V> – the class of the map values
Returns:
    a synchronized view of the specified map.

public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
    return new SynchronizedMap<>(m);
}
```

为防止“拒绝服务”(`dnial-of-service`)攻击，使用私有的锁对象(锁对象应该总是final级别的)——

```
// Private lock object idiom - thwarts denial-of-service attack
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        ...
    }
}
```

### Item 83：Use lazy initializaition judiciously

大部分情况下，普通的初始化优于延迟初始化。

谨慎使用“延迟初始化”(`lazy initialization`)操作。直到字段被需要时才进行初始化，如果不需要，字段永远不被初始化。这一技术适用于静态字段也适用于实例字段。通常这是一种优化策略，但使用不当将对类有破坏性。

最后的处理是“能不用就不用(延迟初始化这把双刃剑)”：通过提高延迟初始化字段的成本降低了初始化类的成本。

如果类的某个字段只在类的一部分场景下使用，并且初始化此字段的成本很高，这种场景下值得应用“延迟初始化”策略。

在并发场景下，延迟初始化需要一些技巧。本条目讨论的技巧都是线程安全的方式——

代替一般初始化`private final FieldType field = computeFieldValue();`的最简单操作：打破初始化流程，使用`synchronized`关键字——

```
// Lazy initialization of instance field - synchronized accessor
private FieldType field;
private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

上面的操作针对静态变量也是一样。静态字段的初始化可以使用“holder class”模式，确保静态变量只在被调用时才会被初始化。这一模式的优点在于不需要使用synchronized关键字，只是一次字段访问操作，没有增加额外的成本(通常虚拟机只有在初始化类是进行字段同步，之后再访问时不需要进行任何的同步操作)——

```
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
private static FieldType getField() { return FieldHolder.field; }
```

如果是针对类实例的字段进行延迟初始化，使用“double-check”方式。这种方式避免了初始化之后的加锁操作。之所以要检查两次，是因为如果字段没有初始化，一旦没有加锁，第二次检查时会加锁（有点绕，看代码。注意字段需要声明为`volatile`类型）——

```
// Double-check idiom for lazy initialization of instance fields
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) { // First check (no locking)
        synchronized(this) {
            if (field == null) // Second check (with locking)
                field = result = computeFieldValue();
        }
    }
    return result;
}
```

上面的代码有点费解，尤其是局部变量的使用。其作用是确保实例变量`field`初始化之后，只被读取一次。有利于提高性能。对应静态变量实例的延迟初始化不需要使用这种方式，使用包装类的是更好的选择。

如果可以忍受变量的多次初始化操作，“双检查”模式可以变换为“单检查”模式(变量依然被声明为`volatile`类型)——

```
// Single-check idiom - can cause repeated initialization!
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) { 
        field = result = computeFieldValue();
    }
    return result;
}
```

上述所以方法对原始类型和对象类型字段都有效。当适用于原始类型时，检查对比值为0而不是null。如果不关注是否每个线程都重新计算字段的值并且字段类型是除了long或double之外的原始类型，则`volatile`关键字可以省略——这一策略被称为"竞赛单检查"(`racy single-check`)模式。以增加初始化的代码加速字段的访问。


### Item 84：Don't depend on the thread scheduler

当有多个线程需要运行时，线程调度程序决定由哪个线程先执行以及执行多久。操作系统会尝试尽可能地公平调度，但调度策略区别很大。任何依赖于线程调度器来确保正确性以及性能的程序将丧失可移植性。

编写健壮、有效、可移植程序的最好方式是确保平均的执行线程不大于内核数。这使得线程调度器的可选择性更少。这种情况下，即使调度策略不同，程序的行为区别也不大。

确保运行中的线程尽可能少的主要方法是：让每个执行中的线程做有用的动作，然后等待(执行下一个有用的动作)。如果没有执行有效动作，线程不应该运行。在Java的执行框架中，这意味着限制线程池的大小，同时保持任务足够短小(不能太短，否则频繁分发任务也会影响性能)

线程不应该处于"忙等待"(`busy-wait`)状态——频繁检查共享对象。这将增加内核的负载并使得线程调度跟脆弱。下面是一个极端“忙等待”例子——

```
// Awful CountDownLatch implementation - busy-waits incessantly!
public class SlowCountDownLatch {

    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                    return;
            }
        }
    }

    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```

如果是因为程序线程无法获取到CPU时间导致无法正常工作，不要试图使用`Thread.yield`方法来解决。即使有效，也让程序丧失了可移植性。在一种JVM环境下yield有效时，在另外一种JVM下可能就无效，甚至让程序更糟糕。

基于此的推论是调整线程的优先级。调整一些线程的优先级来调试程序的响应是有道理的，但这样程序也丧失了可移植性。


## 11 Serialization

这一章关注对象的序列化问题(object serialization)。Java框架将对象编码为字节流(byte streams)过程称为“序列化”(`serializing`)，从字节流中重新构建出对象的过程称为“反序列化”(`deserializing`)。对象一旦被序列化后，就可以从一个虚拟机发送给另一个虚拟机或保存在硬盘以待后用。

本章关注序列化时需要注意的危险问题以及如何最小化这些问题的影响。

### Item 85： Prefer alternatives to Java serialization

最好使用其他方式代替Java的序列化。

当序列化在1997年被加入Java时，就被认为有一些冒险。这种方法在研究性语言上使用过，例如`[Modula-3](http://www.modula3.org/)`，但从来没有在生产级别的语言上使用。

尽管轻松分发对象的允诺很有吸引力，但代价是：不可见构造函数；模糊了API和实现直接的界限；对正确性、安全、性能和维护都有潜在的问题。倡导者相信优点大于缺点，但历史却走向了另一个方向。

本书上一版本中对应安全问题的描述如大家所害怕的一样都一一应验。上世纪还属于讨论阶段的脆弱性在接下来的十年内变成了严重的利用漏洞，包括2016年11月著名的针对“旧金山交通局”的勒索病毒攻击，导致系统瘫痪两天。

序列化最基本的问题是：攻击面过于宽广导致无法防御。通过调用ObjectInputStream的readObject方法，只要类实现了`Serializable`接口，类路径下的任何类型的对象几乎都可以实例化。这导致几乎所有的类型都面临被攻击的危险。包括Java平台基础库、第三方库例如Apache Commons Collections。即使你遵守了写序列化类的所有最佳实践，你的应用依然很脆弱。

攻击者和安全研究员研究Java类库和广泛使用的第三方库中的序列化类型，寻找反序列化时调用的可进行危险操作的方法。这些方法被称为“小玩意”(`gadgets`)。多个小玩意可以组成一个链条，时不时就可以发现一个链条，强大到可以允许攻击者对底层硬件执行任意的原生代码。只需要有机会对一段精心构建的字节流执行反序列化操作即可。针对旧金山交通局的工具就是这样发生的。这种攻击不是个案，而且会越来越多。

甚至不需要任何小玩意，你可以利用反序列化发起“拒绝服务”攻击。一小段字节流的反序列化可能消耗很长的时间。这种字节流被称为“反序列化炸弹”(`deserialization bomb`)。下面一段演示只需要使用hash和字符串的序列化炸弹——

```
// Deserialization bomb - deserializing this stream takes forever
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();

    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo"); // Make t1 unequal to t2
        s1.add(t1); s1.add(t2);
        s2.add(t1); s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // Method omitted for brevity
}
```

这组对象包含201个HashSet实例，每个实例包含少于3个的对象引用，整个字节流只有5744个字节。但直到太阳燃烧殆尽，反序列化这些字节依然没有完成。问题的关键是反序列一个HashSet需要计算元素的哈希值，上例中有100层深度的hash值需要计算，以为着`hashCode`方法要被计算2的100次方…… 

最好的方式就是：能不用就不用(序列化)。在不同平台传递对象在本书被称为“跨平台结构化数据表示”(`cross-platform structured-data representation`)，现在有更简单方便的方式：基于文本的JSON格式和基于二进制的Protocal Buffer

如果时历史遗留系统并且必须使用序列化。确保永远不要对不信任的数据进行反序列化操作。“信任的数据”可以利用“白名单”方式声明，或使用“黑名单”进行过滤。前者优于后者，因为后者只能保护已知的著名的危险类


### Item 86: Implement Serializable with great caution

序列化一个类只需要简单声明让其实现`Serializable`接口即可。看起来如此简单以至于被误认为实现序列化对程序员来说很容易。事实却复杂得多，即时代价也许可以忽略，但长期代价却很昂贵。

主要代价之一是：一旦实现了`Serializable`序列化接口，发布后的类也就丧失了灵活性。序列化成为暴露的API的一部分。如果采用了序列化的默认实现，当前类的私有字段也成为了API的一部分。采用默认序列化实现后，如果后续修改了类的内部表现(`internal representation`)，将导致序列化失败。

所以实现序列化接口后，应当仔细设计序列化实现——尽管这将增加一开始时的开发成本，但值得这么做。

实现了序列化接口的类的演化的限制之一是“流统一标识符”(`stream unique identifiers`)，更通用的名称是“序列号”(`serial version UID`)。每个序列化类都有一个独特的标识符。如果没有声明一个静态final类型的long变量`serialVersionUID`，系统将在运行时通过SHA-1算法生成这个参数值。类名、实现的接口、大部分成员变量以及编译器生成的合成对象都将影响这个变量的值。如果其中任何一个项目有变化——例如增加一个方法——最终的UID都将发生变化。将抛出`InvalidClassException`异常

第二个代价是实现序列化后提高了出现安全漏洞和bug的可能性。通常类初始化通过构造函数，但序列化提高了“语言之外的机制”(`extralinguistic mechanism`)完成类初始化动作。

第三个代价是增加了测试成本。新版本发布后，需要验证高低版本之间的序列化转换是否成功。随着版本的增加，序列化的测试成本是指数级的。

### Item 87: Consider using a custom serialized form

默认的序列化实现可以高效地表示当前对象的实体表现（~直接从英文翻译过来已经理解不能了~）时，可以采用默认实现。

换句话表达上面的含义：序列化可以很好地表达对象中包含的数据以及所有从当前对象可触达的所有对象。（看后面一个例子就容易理解了）

如果对象的实体表现(`physical representation`)与自身的逻辑内容(`logical content`)一致时，使用默认的序列化实现是可行的。例如下面的类——

```
// Good candidate for default serialized form
public class Name implements Serializable {
    /**
     * Last name. Must be non-null.
     * @serial
     */
    private final String lastName;
    /**
     * First name. Must be non-null.
     * @serial
     */
    private final String firstName;

    /**
     * Middle name, or null if there is none.
     * @serial
     */
    private final String middleName;

    // Remainder omitted
}
```

逻辑上讲，一个“名字”包含了last name、first name, 和middle name。上述类的字段正是这个逻辑内容的映射。

即使默认序列化是合适的，也应该提供`readObject`方法以确保安全性和不可变性。针对上述类，需要确保lastName和firstName不为空。

另外注意的时，尽管三个字段都是私有属性，依然添加了注释，因为序列化实现使得这些属性变成了公共API，所以需要添加注释。并且适用了`@serial`标签，告诉`Javadoc`将这些注释添加到特定的页面——序列化表格页。

再看一个反例——

```
// Awful candidate for default serialized form
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }

    // Remainder omitted
}
```

逻辑上讲，这个类表示了一连串的字符串。实体上来看，适用双链表表示这一连串的字符串。如果使用默认的序列化操作，将双向映射链表中的每一个entry以及entry之间的所有链接。

这种情况下，默认序列化操作有以下四个劣势——

- 永久性将内部的表示暴露给了外部的API。`StringList.Entry`称为了公共API的一部分。后续表示即使变更，双向链表也无法去除。
- 占据过量空间。默认序列化保持了所有的entry以及连接信息，后者只是实现的方式，不值得包含在序列化信息之中。这导致序列化占据过多空间，存盘还是网络传输时速度很慢。
- 过于耗时。序列化逻辑不知道对象的拓扑结构，必须遍历所有的节点。实际上，上面的例子里只需要跟踪`next`引用即可
- 可能导致堆溢出。默认的序列化操作执行递归遍历，即使数量不大，也可能引起堆溢出。根据不同的机器，1000~1800个字符串的序列化就可能出现堆溢出的情景。

合理的序列化方式只需要记录字符串的个数以及字符串本身。这些构成了StringList类的逻辑数据。

修改序列化实现的改进版本——

```

// StringList with a reasonable custom serialized form
public final class StringList implements Serializable {

    private transient int size = 0;
    private transient Entry head = null;

    // No longer Serializable!
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // Appends the specified string to the list
    public final void add(String s) { /*...*/ }

    /**
     * Serialize this {@code StringList} instance.
     *
     * @serialData The size of the list (the number of strings
     * it contains) is emitted ({@code int}), followed by all of
     * its elements (each a {@code String}), in the proper
     * sequence.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        // Write out all elements in the proper order.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        
        s.defaultReadObject();
        int numElements = s.readInt();
        // Read in all elements and insert them in list
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    // Remainder omitted
}
```

即使所有字段都声明为`transient`，`writeObject`方法还是第一步先调用了`defaultWriteObject`方法；`readObject`方法第一件事也是调用`defaultReadObject`方法。虽然这种情况下，这两个方法调用可以省略。——这算是一种最佳实践，后续添加了非`transient`属性的字段时，依然可以兼容。

私有方法添加了注释，其中`@serialData`标签告诉`JavaDoc`将这些内容放置在特定的序列化操作页面。

修改之后，占据的空间更少，并且不再出现堆溢出现象。

尽管StringList类的默认序列化操作有各种问题，但至少是可以工作的“正确实现”。有些类的序列化操作如果依赖默认实现将无法正常工作。例如哈希表对象，其实体表示是：一组哈希桶，每个桶包含了键-值对实例。每个桶是其包含的实例的key值的哈希函数。这意味着通常来说，每次实现的位置不一定都是相同的。实际上，每次运行时对应的位置都可能是不同的。这种情况下，使用默认的序列化操作将导致数据损坏。

无论是否使用自定义的序列化操作，调用`defaultWriteObject`方法时，所以非`transient`属性的字段都会被执行序列化操作。因此，所以可以声明为`transient`的字段都应当声明为如此，包含延伸字段(值可以在运行时计算出来的字段)。

默认序列化实现中，所以标记为`transient`属性的字段在反序列化时都会被初始化为默认值：`null, 0, false`。如果这些值不合适，则需要提供自定义的反序列化方法`readObject`，类似上面的例子。

另外需要注意的两点：

- 类中包含了线程安全的同步化操作时，序列化方法也需要添加同步锁。否则，将导致资源序列死锁`resource-ordering deadlock`
- 最好提供自定义的UID值。`private static final long serialVersionUID = randomLongValue;`不需要确保唯一。但不同版本间需要保持一致。不同的值意味着打破了兼容性


### Item 88: Write readObject method defensively 

Item 50中提到下面的例子，使用防御式复制方法以确保对象的不可变性。

```
// Immutable class that uses defensive copying
public final class Period {
    private final Date start;
    private final Date end;

    /**
    * @param start the beginning of the period
    * @param end the end of the period; must not precede start
    * @throws IllegalArgumentException if start is after end
    * @throws NullPointerException if start or end is null
    */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                            start + " after " + end);
    }

    public Date start () { return new Date(start.getTime()); }
    public Date end () { return new Date(end.getTime()); }
    
    public String toString() { return start + " - " + end; }
    
    // Remainder omitted
}
```

假设你决定让这个类实现序列化接口。由于实体展示与逻辑数据一致，使用默认的序列化方法是有道理的。所以只需要让当前类实现`Serializable`接口即可。如果这样做，这个类将不再能确保其关键的不可更改性。

原因是`readObject`方法是另外一个公共的构造方法，应当跟上述已有方法一样需要仔细处理。（默认的序列化方法显然没有做这个动作）

简单来说，`readObject`是一个构造方法，使用字节流作为唯一的参数(`byte stream`)。通常，这些字节流来自对构造实例对象的序列化。但如果字节流来自手动构造并且违反了类可变性，将创造出“不可能对象”(`impossible object`)——即不符合预期的对象。

使用默认的序列化方法时，下面的代码将创建出结束日期在开始日期之前的Period对象。（因为Java缺少字节类型的字面值，所以需要进行强制转换——~这段解释也看不懂其实~）

```
public class BogusPeriod {
    // Byte stream couldn't have come from a real Period instance!
    private static final byte[] serializedForm = {
            (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
            0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
            0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
            0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
            0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
            0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
            0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
            0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
            0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
            (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
            0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
            0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
            0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
            0x00, 0x78
    };
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    // Returns the object with the specified serialized form
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(
                    new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

如果对如何构造字节流感兴趣可以参考[Java Object Serialization Specification](https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serialTOC.html)，打印出`Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984`违反了Period类的定义。

做简单的防御处理——

```
// readObject method with validity checking - insufficient!
private void readObject(ObjectInputStream s)
    throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Check that our invariants are satisfied
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```

上述防御还不充分，可以构造出符合规范的字节流，然后在字节流中再修改私有字段的属性以达到越过检查的目的（如何构造这些字节流超出本书的范围）(也超出我目前的理解范围)。

加强版防御方法——

```
// readObject method with defensive copying and validity checking
private void readObject(ObjectInputStream s)
    throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Defensively copy our mutable components
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // Check that our invariants are satisfied
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +" after "+ end);
}
```

总结一下写反序列化方法的原则——

- 类中的对象引用字段必须是私有时，对每一个这样的对象都要做防御拷贝
- 对任何不可变量都要做检查，如果失败则抛出`InvalidObjectException`异常
- 如果反序列化后必须对整体对象图做验证，使用`ObjectInputValidation`接口（？超纲内容- -）
- 不要在类中调用任何重写的方法

### Item 89: For instance control, prefer enum types to readResolve

Item 3中提到的单例模式如果实现了`Serializable`接口之后将失去“单例”效果(无论采用默认序列化方法，还是使用定制化的`readObject`方法，这一方法都将返回一个新创建的，不同于初始化时的类实例对象)。

```
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

`readResole`功能运行你使用一个实例代替由`readObject`方法返回的类实例，参考官方说法[Serialization, 3.7](https://docs.oracle.com/javase/8/docs/platform/serialization/spec/input.html)——

>For Serializable and Externalizable classes, the `readResolve` method allows a class to replace/resolve the object read from the stream before it is returned to the caller. By implementing the readResolve method, a class can directly control the types and instances of its own instances being deserialized.

利用这一特性，反序列化字节流后可以创建出的对象将返回有`readResolve`返回的对象。大部分场景下，这一对象没有任何引用，可以被GC立即回收。

```
// readResolve for instance control - you can do better!
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
```

这要求序列化`Elvis`类时不能包含任何数据，所有字段必须声明为`transient`。否则可以被攻击者利用：在`readResolve`方法调用之前就构造出对反序列化对象的引用。

攻击实现比较复杂，但原理很简单：如果单例对象包含非`transient`属性的字段，此字段内容将在`readResole`方法被调用之前执行反序列化操作，这就允许构造出“对反序列化对象的引用”。

下面的内容未进行实操，来自原文——

实现了`Serializable`接口的单例对象
```
// Broken singleton - has nontransient object reference field!
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    private String[] favoriteSongs =
            { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
    private Object readResolve() {
        return INSTANCE;
    }
}
```

构造的“小偷”对象——
```
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;
    
    private Object readResolve() {
        // Save a reference to the "unresolved" Elvis instance
        impersonator = payload;
        // Return object of correct type for favoriteSongs field
        return new String[] { "A Fool Such as I" };
    }
    private static final long serialVersionUID = 0;
}
```

攻击演示——

```
public class ElvisImpersonator {
    // Byte stream couldn't have come from a real Elvis instance!
    private static final byte[] serializedForm = {
            (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
            0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
            (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
            0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
            0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
            0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
            0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
            0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
            0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
            0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
            0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
            0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
            0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
    };
    
    public static void main(String[] args) {

        // Initializes ElvisStealer.impersonator and returns
        // the real Elvis (which is Elvis.INSTANCE)
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();
    }
}
```

最终打印出——
```
[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
```

解决这一问题可以将`favoriteSongs`字段声明为`transient`类型；但更好的解决方案是将`Elvis`类声明为枚举类。声明为枚举类之后，Java将保证除了声明的常量之外，不再有任何实例被创建。除非攻击者可以调用类似`AccessibleObject.setAccessible`方法——但这说明攻击者已经获取到了执行任意本地代码的权限。

总结一下，尽管`readResole`方法没有被废弃，但使用单例方式最后使用枚举类的形式。如果即需要序列化又需要单例模式，必须提供`readResolve`方法，并且确保所以字段是`transient`类型或是基础类型。

### Item 90: Consider serialization proxies instead of serialized instances

使用序列化代理，而不是直接序列化实例

item85中讨论的问题——实现序列化接口后更可能出现bug和安全问题——可以利用“序列化代理模式”(`serialization proxy pattern`)来降低风险。

在要进行序列化的类中设计一个私有的静态嵌套类，作为其所在类的序列化代理。只包含一个构造函数，参数为其所在类的对象实例。两个类都需要实现序列化接口——

```
// Serialization proxy for Period class
private static class SerializationProxy implements Serializable {
    
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID = 234098243823485285L; // Any number will do (Item 87)
}
```

下一步，在序列代理所在的类(被称为`enclosing class`)中添加`writeReplace`方法(任何包含序列代理的类中都可以原样添加此方法)——

```
// writeReplace method for the serialization proxy pattern
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

包含了这个方法之后，所在类进行序列化时，序列化系统将产生(`emit`)`SerializationProxy`实例对象而不是所在类的实例。换句话说，在序列化之前，`writeReplace`方法将所在类的实例转化为了序列代理类。

这样序列化实际序列化的是代理类。为防御攻击，需要另外在所在类中添加如下的`readObject`方法——

```
// readObject method for the serialization proxy pattern
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("Proxy required");
}
```

最后，在序列代理类中提供`readResolve`方法，返回一个逻辑上与所在类等价的实例。这样确保让序列化系统将序列代理类重新转化为所在类对象实例。此方法中只需要调用公共的API即可——如果正常运行时调用一样(实际却是发生在反序列化过程中)

```
// readResolve method for Period.SerializationProxy
private Object readResolve() {
    return new Period(start, end); // Uses public constructor
}
```

这让之前提到的防御拷贝，字节流攻击防御都不需要，而且所有字段都可以重新声明为final类型，让所在类重新变成不可变类。

另外一个好处是，序列化代理可以实现在反序列化实例时生成不同的原始序列化类。（这在实际应用中很有用）

例如item 36中提到的`EnumSet`类，没有构造函数，只有静态工厂。从客户端角度看，返回的都是`EnumSet`实例，但在实际的实现时，会根据枚举数量的不同返回不同的类：如果少于64个元素，返回`RegularEnumSet`对象；反之，返回的是`JumboEnumSet`。

假设：序列化一个包含了60个元素的枚举类；然后再添加5个枚举常量；然后反序列化。序列化时，操作的是`RegularEnumSet`，但反序列化后，最后是返回`JumboEnumSet`实例。

使用序列化代理可以轻松完成这一场景——

```
// EnumSet's serialization proxy
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
    // The element type of this enum set.
    private final Class<E> elementType;

    // The elements contained in this enum set.
    private final Enum<?>[] elements;
    
    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]);
    }

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
            result.add((E)e);
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}
```

序列化代理有两个限制：

- 第一，不兼容用户扩展的类
- 第二，不兼容对象graph包含循环的类。这种类中调用readResolve方法时将抛出`ClassCastException`（因为此时还没有对应的对象，只有序列化代理)

另外，序列化代理有一些性能损耗。

总之，如果类不允许用户扩展，需要进行序列化操作时尽量使用序列化代理模式。这是最简单的序列化方式。
