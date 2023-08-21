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

Java 的线程锁是可重入的锁。











