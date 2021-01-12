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

## 1 Creating and Destroying Objects

### item 8: Avoid finalizers and cleaners 

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

### item 9: Prefer try-with-resources to try-finally

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

### item 10: Obey the general contract when overriding `equals`

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

### item 10: Obey the general contract when overriding `equals`

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


### item 11: Always override `hashCode` when you override `equals`

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

### item 12: Always override `toString`

这条大概日常用得最多了，“一致，有效”

- 默认的实现是`类名@十六进制`hashCode的十六进制表示
- 这个实现不“强制”，只是从实际使用角度更“方便”
- 最好返回类中所有的“有效”信息
- 返回格式“不一定”需要固定：不固定可能造成测试问题（输出顺序），但保持灵活性；固定可以提高确定性，可读性，但丧失灵活性，提供给第三方使用时，对方可能已写了对应的parser，如果后续有变更……
- 无论哪一种情况，最好做文档说明
- 无论是否采取固定方式，都需要提供访问“有效”信息的方法

对于值类型，最好重写此方法；静态工具类、枚举类不需要；再次推荐AutoValue

### item 13: Override `clone` judiciously

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

### item 14: Consider implementing `Comparable`

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

### item 15 Minimize the accessibility of classes and members

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

### item 20:  Prefer interfaces to abstract classes

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

### item 21: Design interfaces for posterity

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

### item 22: Use interfaces only to define types

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
