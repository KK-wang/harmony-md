# learn-4

## 1.多线程

### 1.1.同步方法

如果一个类被设计为允许多线程正确访问，我们就说这个类就是“线程安全”的（thread-safe）。Java标准库的`java.lang.StringBuffer`就是线程安全的。

还有一些不变类，例如`String`，`Integer`，`LocalDate`，它们的所有成员变量都是`final`，多线程同时访问时只能读不能写，这些不变类也是线程安全的。

> 这里的只能读不能写指的是引用变量指向的变量只能读不能写，但是引用变量本身是可以被读写的。

最后，类似`Math`这些只提供静态方法，没有成员变量的类，也是线程安全的。

> 如果一个变量是只能读不能写，那么这个变量就是线程安全的。

除了上述几种少数情况，大部分类，例如`ArrayList`，都是非线程安全的类，我们不能在多线程中修改它们。但是，如果所有线程都只读取，不写入，那么`ArrayList`是可以安全地在线程间共享的。

>没有特殊说明时，一个类默认是非线程安全的。

一个典型的线程安全类如下：

```java
public class Counter {
    private int count = 0;
    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
    public void dec(int n) {
        synchronized(this) {
            count -= n;
        }
    }
    public int get() {
        return count;
    }
}
```

不难发现，`synchronized`锁住的对象是`this`，即当前实例，这又使得创建多个`Counter`实例的时候，它们之间互不影响，可以并发执行。而各个 Counter 实例在多线程模式下又能正确地执行 add 和 dec。

而当我们锁住的是 this 实例时，实际上可以用 `synchronized` 修饰方法，如下所示：

```java
public synchronized void add(int n) {
    count += n;
}
```

用`synchronized`修饰的方法就是同步方法，它表示整个方法都必须用`this`实例加锁。

如果对一个静态方法添加 `synchronized` 修饰符，**锁住的是该类的`Class`实例**。如下所示：

```java
public synchronized static void test(int n) {
    ...
}

// <==>

public class Counter {
    public static void test(int n) {
        synchronized(Counter.class) {
            ...
        }
    }
}
```

判断方法是否需要同步修饰，需要考虑方法在多线程情景下是否依旧能够正确运行，例如对于以下 get 方法：

```java
public class Counter {
    private int first;
    private int last;

    public Pair get() {
        Pair p = new Pair();
        p.first = first;
        p.last = last;
        return p;
    }
    ...
}
```

由于在给变量 p 赋值的过程中可能会出现 first 和 last 值改变的情况，因此需要对 get 方法添加 synchronized 关键词。

### 1.2.死锁

#### 1.2.1.可重入锁

Java 的线程锁是可重入的锁。什么是可重入的锁？例子如下：

```java
public class Counter {
    private int count = 0;

    public synchronized void add(int n) {
        if (n < 0) {
            dec(-n);
        } else {
            count += n;
        }
    }

    public synchronized void dec(int n) {
        count += n;
    }
}
```

观察`synchronized`修饰的`add()`方法，一旦线程执行到`add()`方法内部，说明它已经获取了当前实例的`this`锁。如果传入的`n < 0`，将在`add()`方法内部调用`dec()`方法。由于`dec()`方法也需要获取`this`锁，现在问题来了：对同一个线程，能否在获取到锁以后继续获取同一个锁？

答案是肯定的。JVM 允许同一个线程重复获取同一个锁，**这种能被同一个线程反复获取的锁，就叫做可重入锁**。由于 Java 的线程锁是可重入锁，所以，获取锁的时候不但需要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出`synchronized`块，记录-1，减到0的时候，才会真正释放锁。

#### 1.2.2.死锁

在获取多个锁的时候，不同线程获取多个不同对象的锁可能导致死锁。对于上述代码，线程1和线程2如果分别执行`add()`和`dec()`方法时：

- 线程1：进入`add()`，获得`lockA`。
- 线程2：进入`dec()`，获得`lockB`。

随后：

- 线程1：准备获得`lockB`，失败，等待中。
- 线程2：准备获得`lockA`，失败，等待中。

此时，两个线程各自持有不同的锁，然后各自试图获取对方手里的锁，造成了双方无限等待下去，这就是死锁。

> 需要注意的是，没有任何机制能解除死锁，只能强制结束JVM进程。

为了尽可能地避免死锁现象，应当**保证线程获取锁的顺序是一致的**。例如严格保证先获取`lockA`，再获取`lockB`的顺序。

### 1.3.使用 wait 和 notify

在 Java 程序中，synchronized 解决了多线程竞争的问题，但是 **synchronized** 并没有解决多线程协调问题。多线程协调运行的原则是：

1. 当条件不满足时，线程进入等待状态（非 while 循环的忙等）。
2. 当条件满足时，线程被唤醒，继续执行任务。

此时我们可以引入 `wait` 和 `notify`。







