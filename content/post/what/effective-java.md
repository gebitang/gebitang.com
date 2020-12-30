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