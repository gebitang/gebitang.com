+++
title = "Java Thread"
description = "2nd Edition and 3rd Edition"
tags = [
    "java"
]
date = "2021-04-25"
topics = [
    "java"
]
toc=true
+++

[第二版，1999年](https://2lib.org/book/490277/32b999)、[第三版，2004年](https://2lib.org/book/11045353/7ad5ea)，可对照看。

一个类中的多个`synchronized`的方法：针对当前对象，同一时间多个线程只能执行其中的一个`synchronized`的方法。因为锁是针对对象的，并且每个对象只有一个锁token。

静态方法的`synchronized`同步锁被称为`class lock`(只是一种概念，实际上加锁的是一个静态对象？)：静态方法中传入实例对象引用，依然可以继续调用对应实例的同步方法。因为这是两把不同的锁。


第一版本BusyFlag——
```
public class BusyFlag {

    protected Thread busyflag = null;
    protected int busycount = 0;

    public void getBusyFlag() {
        while (!tryGetBusyFlag()) {
            try {
                Thread.sleep(100);
            } catch (Exception e) {}
        }
    }

    public synchronized boolean tryGetBusyFlag() {
        if (busyflag == null) {
            busyflag = Thread.currentThread();
            busycount = 1;
            return true;
        }
        if (busyflag == Thread.currentThread()) {
            busycount++;
            return true;
        }
        return false;
    }
    public synchronized void freeBusyFlag () {
        if (getBusyFlagOwner() == Thread.currentThread()) {
            busycount--;
            if (busycount == 0)
                busyflag = null;
        }
    }
    public synchronized Thread getBusyFlagOwner() {
        return busyflag;
    }
}
```

- `synchronized`关键字可以赋予整个方法，也可以赋予代码块——使用哪种方式取决于个人选择，只要避免死锁问题即可
- `freeBusyFlag()`方法和`getBusyFlagOwner()`方法都添加了`synchronized`关键字；并且前者方法体中还调用了后者。
>这种情况并不会造成死锁(后者不需要等待前者释放锁之后再执行)。实际上，如果当前线程已经获取了锁，是不需要在等待锁被释放，然后再去获取一次。所以从`freeBusyFlag()`方法中调用`getBusyFlagOwner()`方法没有问题。
>引申含义是，如果开始执行同步方法时没有进行获取锁的操作的话，执行完同步方法也就不存在释放锁的动作。

上述版本中使用`sleep`方法进行等待，造成了CPU的浪费。使用`wait()`和`notify()`机制优化如下——

```
public class BusyFlag {

    protected Thread busyflag = null;
    protected int busycount = 0;


    public synchronized void  getBusyFlag() {
        while (!tryGetBusyFlag()) {
            try {
                wait();
            } catch (Exception e) {}
        }
    }

    public synchronized boolean tryGetBusyFlag() {
        if (busyflag == null) {
            busyflag = Thread.currentThread();
            busycount = 1;
            return true;
        }
        if (busyflag == Thread.currentThread()) {
            busycount++;
            return true;
        }
        return false;
    }

    public synchronized void freeBusyFlag() {
        if (getBusyFlagOwner() == Thread.currentThread()) {
            busycount--;
            if (busycount == 0) {
                busyflag = null;
                notify();
            }
        }
    }
    public synchronized Thread getBusyFlagOwner() {
        return busyflag;
    }
}

```

- `getBusyFlagOwner()`方法也添加了同步操作，因为`wait()`方法必须在同步方法中调用
- `wait()`和`notify()`方法都必须要在同步方法中调用(或使用相同锁对象的同步代码块)。前者表示“需要等待某个条件发生”；后者表示“等待的条件已经发生”
- `wait()`在同步方法中调用，似乎永远占据了对象锁，导致其他同步方法无法执行。实际上，在执行`wait()`前，将释放锁，然后进入wait状态；条件满足后(`notify()`调用后，)`wait`方法返回前，重新获取到对象锁。
- 实际上，Java并没有提供获取锁、释放锁的API，`wait()`方法是本地方法，与同步操作紧密结合。这样从外部看，似乎是`wait()`方法一直占据着锁
- 如果`notify()`方法调用时，没有`wait()`方法调用，立即返回即可，不会有副作用


在内部实现上，`wait()`和`notify()`通常都是通过控制变量来实现的。所以这要求对这两个方法的调用需要加锁，确保内部对变量的操作是原子的。

另外需要注意的是`getBusyFlag()`方法需要在循环中进行检查，如果采用如下方式，`wait()`结束后直接进行获取，依然可能无法得到flag——

```
public synchronized void  getBusyFlag() {
    if (!tryGetBusyFlag()) {
        try {
            wait();
            tryGetBusyFlag()
        } catch (Exception e) {}
    }
}
```

1. 第一个线程获取到`busyflag`
2. 第二个线程调用`tryGetBusyFlag()`：返回false
3. 第二个线程执行`wait()`方法：释放同步锁
4. 第一个线程进入`freeBusyFlag()`方法：获取到同步锁
5. 第一个线程调用`notify()`方法
6. 第三个线程尝试调用`getBusyFlag()`方法，等待获取到锁对象
7. 第一个线程退出`freeBusyFlag()`方法，释放了锁对象
8. 第三个线程获取到锁对象，进入`getBusyFlag()`方法。此时busyflag被释放，这第三线程获取到busgflag并退出`getBusyFlag()`方法，同时释放锁
9. 第二个线程收到了来自第一个线程的`notify`的通知，从`wait()`方法返回并获取到同步锁
10. 第二个线程再次调用`tryGetBusyFlag()`方法，busgflag被占有，再次进入`wait`状态——如果采用上面的实现，这一步实际上无法获取到busyflag



利用锁对象的同步块例子——

```
public class ExampleBlockLock {
    private StringBuffer sb = new StringBuffer();
    public void getLock() {
        doSomething(sb);
        synchronized (sb) {
            try {
                sb.wait();
            } catch (Exception e) {}
        }
    }
    public void freeLock() {
        doSomethingElse(sb);
        synchronized (sb) {
            sb.notify();
        }
    }
}
```

---

假设有多个线程都处于`wait()`状态。

`notify()`是无法知道**应该**通知哪一个线程。所以可以调用`notifyAll()`，所有的等待线程都会收到通知。

但并不是所有的等待线程都并行执行。因为从wait()状态返回后，还是要重新获取到对象锁才能继续执行，然后每个线程可以根据自己的情况判断：是继续执行；还是需要再次进入等待状态。

这样有个好处是：

- 系统释放的资源可能无法满足某一个wait的线程；但可能满足另外一个wait的线程
- 甚至：释放的总资源，可能满足多个wait的线程（这需要程序员设计机制来进行资源分配，例如优先让需要资源较小的线程收到通知后继续执行）

如果需要“精确”控制哪一个线程接收到通知，可以设计一个类似如下的类——

```
public class TargetNotify {
    private Object Targets[] = null;
    public TargetNotify (int numberOfTargets) {
        Targets = new Object[numberOfTargets];
        for (int i = 0; i < numberOfTargets; i++) {
            Targets[i] = new Object();
        }
    }
    public void wait (int targetNumber) {
        synchronized (Targets[targetNumber]) {
            try {
                Targets[targetNumber].wait();
            } catch (Exception e) {}
        }
    }
    public void notify (int targetNumber) {
        synchronized (Targets[targetNumber]) {
            Targets[targetNumber].notify();
        }
    }
}
```


