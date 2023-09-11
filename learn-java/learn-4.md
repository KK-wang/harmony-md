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

此时我们可以引入 `wait` 和 `notify`。对于以下代码：

```java
public synchronized String getTask() {
    while (queue.isEmpty()) {
        this.wait();
    }
    return queue.remove();
}
```

当线程执行到 `this.wait()` 时，线程就会进入到等待状态。这里的关键是 `wait()`方法必须在当前获取的锁对象上调用，这里获取的是`this`锁，因此调用`this.wait()`。

> `wait()`方法不会返回，直到将来某个时刻，线程从等待状态被其他线程唤醒后，`wait()`方法才会返回，然后，继续执行下一条语句。

这里有个小细节，当线程由于 wait 方法在 `getTask()` 内部等待时，其他线程按道理来说依旧无法拿到 this 锁，从而无法执行 `addTask()` 方法。

这个问题的关键就在于`wait()`方法的执行机制非常复杂。首先，它不是一个普通的Java方法，而是定义在`Object`类的一个`native`方法，也就是由JVM的C代码实现的。其次，必须在`synchronized`块中才能调用`wait()`方法，**因为`wait()`方法调用时，会释放线程获得的锁，`wait()`方法返回后，线程又会重新试图获得锁**。

```java
// 持有 this 锁。
this.wait(); // 释放 this 锁。
// 重新获取 this 锁。
```

因此当一个线程在`this.wait()`等待时，它就会释放`this`锁，从而使得其他线程能够在`addTask()`方法获得`this`锁。

> 之前曾经提到，当一个线程处于等待状态，且有其他线程调用其的 `interrupt()` 方法时，就会抛出 InterruptedException 异常。现已知：
>
> * wait。
> * sleep。
> * join。
>
> 三者都会让线程进入等待状态。

如何让等待的线程被重新唤醒，然后从 `wait()` 方法返回？答案是**在相同的锁对象上**调用 `notify()` 方法。

`notify()` 方法会唤醒一个正在 `this` 锁中等待的线程（即在 `getTask()` 中位于 `this.wait()` 的线程），**从而使得等待线程从 `this.wait()` 方法中返回**。

如果有多个线程处于 `wait` 状态，那么 `notify` 会唤醒几个线程呢？答案是一个，具体是哪个线程，这取决于操作系统，有一定的随机性。如果我们想要一次性唤醒所有的 wait 线程，那么可以使用方法 `notifyAll()`。

> 通常来说，notifyAll 要更安全些，因为有些时候，如果我们的代码逻辑考虑不周，用`notify()`会导致只唤醒了一个线程，而其他线程可能永远等待下去醒不过来了。

不过需要注意的是，**`wait()`方法返回时需要重新获得`this`锁**。假设当前有3个线程被唤醒，唤醒后，首先要等待执行`addTask()`的线程结束此方法后，才能释放`this`锁，随后，这3个线程中只能有一个获取到`this`锁，剩下两个将继续等待。

另外，注意到在 getTask 方法中，判断队列是否为空使用的是 while 而不是 if，假设我们将其更换为 if：

```java
public synchronized String getTask() throws InterruptedException {
    if (queue.isEmpty()) {
        this.wait();
    }
    return queue.remove();
}
```

这种写法实际上是错误的，因为线程被唤醒时，需要再次获取`this`锁。多个线程被唤醒后，只有一个线程能获取`this`锁，此刻，该线程执行`queue.remove()`可以获取到队列的元素，然而，剩下的线程如果获取`this`锁后执行`queue.remove()`，此刻队列可能已经没有任何元素了，**所以，要始终在`while`循环中`wait()`，并且每次被唤醒后拿到`this`锁就必须再次判断**：

```java
while (queue.isEmpty()) {
    this.wait();
}
```

### 1.4.使用 ReentrantLock（灵活的 synchronized）

从 Java5 开始，引入了一个高级的处理并发的 `java.util.concurrent` 包，他提供了大量更高级的并发功能，能够大大简化多线程程序的编写。`synchronized` 关键词用作锁时有两个缺陷：

* 锁很重。
* 获取锁时必须一直等待，没有额外的尝试机制。

以下是 ReentrantLock 的使用示例及对比：

```java
// synchronized
public class Counter {
    private int count;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
}

// ReentrantLock
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```

因为`synchronized`是Java语言层面提供的语法，所以我们不需要考虑异常，而**`ReentrantLock`是Java代码实现的锁，我们就必须先获取锁，然后在`finally`中正确释放锁**。

> `ReentrantLock`是可重入锁，它和`synchronized`一样，一个线程可以多次获取同一个锁。

和`synchronized`不同的是，`ReentrantLock`可以尝试获取锁：

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

上述代码在尝试获取锁的时候，最多等待 1 秒。如果 1 秒后仍未获取到锁，那么 `tryLock()` 返回 `false`，程序就可以做一些额外处理，而不是无限等待下去。

基于这个方法，因此可以说使用 `ReentrantLock` 要比直接使用 `synchronized` 更安全，线程在 `tryLock()` 失败的时候不会导致死锁。

### 1.5.使用 Condition（灵活的 wait 和 notify）

前面提到，使用 `ReentrantLock` 比直接使用 `synchronized` 更安全，可以替代 `synchronized` 进行线程同步。但是，`synchronized`可以配合`wait`和`notify`实现线程在条件不满足时等待，条件满足时唤醒，用`ReentrantLock`我们怎么编写`wait`和`notify`的功能呢？

答案是使用 `Condition`。

以 `TaskQueue` 为例，将前面用 `synchronized` 实现的功能通过 `ReentrantLock` 和 `Condition` 来实现：

```java
private final Lock lock = new ReentrantLock();
private final Condition condition = lock.newCondition();
private Queue<String> queue = new LinkedList<>();

public void addTask(String s) {
    lock.lock();
    try {
        queue.add(s);
        condition.signalAll();
    } finally {
        lock.unlock();
    }
}

public String getTask() {
    lock.lock();
    try {
        while (queue.isEmpty()) {
            condition.await();
        }
        return queue.remove();
    } finally {
        lock.unlock();
    }
}
```

可见，使用 `Condition` 时，引用的 `Condition` 对象必须从 `Lock` 实例的 `newCondition()` 返回，这样才能获得一个绑定了 `Lock` 实例的 `Condition` 实例。

>Java 中，定义成员变量并初始化和成员变量在构造方法中初始化有什么区别？
>
>声明时为成员变量赋值，那么你一创建对象，这个赋值就进行，而且先于构造器执行，~~**而且你每次创建这个类的对象，都是同一个值**~~，不过，**和构造方法初始化一样的是，每个对象的成员变量的值是不同的**。

`Condition`提供的`await()`、`signal()`、`signalAll()`原理和`synchronized`锁对象的`wait()`、`notify()`、`notifyAll()`是一致的，并且其行为也是一样的：

- `await()`会释放当前锁，进入等待状态；
- `signal()`会唤醒某个等待线程；
- `signalAll()`会唤醒所有等待线程；
- 唤醒线程从`await()`返回后需要重新获得锁。

此外，和 `tryLock()` 类似，`await()` 可以在等待指定时间后，如果还没有被其它线程通过 `signal()` 或者 `signalAll()` 唤醒，可以自己醒来：

```java
if (condition.await(1, TimeUnit.SECOND)) {
    // 被其他线程唤醒
} else {
    // 指定时间内没有被其他线程唤醒
}
```

不难发现，相比于 synchronized 和 wait，ReentrantLock 和 Condition 有更加灵活的线程同步机制，因为两者都有超时机制，如果太久无法持有锁，那么可以做别的事情。

> 注意，`Condition`对象必须从`Lock`对象获取。

### 1.6.使用 ReadWriteLock（读写锁）

前面讲的 ReentrantLock 和 synchronized 保证了只有一个线程可以执行临界区的代码：

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        lock.lock();
        try {
            counts[index] += 1;
        } finally {
            lock.unlock();
        }
    }

    public int[] get() {
        lock.lock();
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            lock.unlock();
        }
    }
}
```

但是有些时候，这种保护有点过头。因为我们发现，任何时刻，只允许一个线程修改，也就是调用`inc()`方法是必须获取锁，但是，`get()`方法只读取数据，不修改数据，它实际上允许多个线程同时调用。

> 也就是说，一个线程在读的时候，其他线程也是可以读的，**这有助于增大读并发**。

实际上我们想要的是：允许多个线程同时读，但只要有一个线程在写，其它线程就必须等待：

|                        | 一个线程在读 | 一个线程在写 |
| :--------------------: | :----------: | :----------: |
| **是否允许其他线程读** |     允许     |    不允许    |
| **是否允许其他线程写** |    不允许    |    不允许    |

使用 `ReadWriteLock` 可以解决这个问题，它保证：

* 只允许一个线程写入（其它线程既不能写入也不能读取）。
* 没有写入时，多个线程允许同时读（提高性能）。

用 `ReadWriteLock` 实现这个功能十分容易。我们需要创造一个 `ReadWriteLock` 实例，然后分别获取读锁和写锁：

```java
public class Counter {
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();
    private final Lock wlock = rwlock.writeLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        wlock.lock(); // 加写锁
        try {
            counts[index] += 1;
        } finally {
            wlock.unlock(); // 释放写锁
        }
    }

    public int[] get() {
        rlock.lock(); // 加读锁
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            rlock.unlock(); // 释放读锁
        }
    }
}
```

把读写操作分别用读锁和写锁来加锁，**在读取时，多个线程可以同时获得读锁，这样就大大提高了并发读的执行效率**。

> ReadWriteLock 是可以用来代替 ReentrantLock 的。

使用 `ReadWriteLock` 时，适用条件是同一个数据，有大量线程读取，但仅有少数线程修改。例如，一个论坛的帖子，回复可以看做写入操作，它是不频繁的，但是，浏览可以看做读取操作，是非常频繁的，这种情况就可以使用`ReadWriteLock`。

### 1.7.使用 StampedLock（乐观锁）

前面介绍的`ReadWriteLock`可以解决多线程同时读，但只有一个线程能写的问题。如果我们深入分析 `ReadWriteLock`，会发现它有一个潜在的问题：**如果有线程正在读，写线程需要等待读线程释放锁后才能获取写锁，即读的过程中不允许写，这是一种悲观的读锁**。

要进一步提升并发执行效率，Java 8 引入了新的读写锁：`StampedLock`。`StampedLock`和`ReadWriteLock`相比，改进之处在于：**读的过程中也允许获取写锁后写入**！这样一来，我们读的数据就可能不一致，所以，**需要一点额外的代码来判断读的过程中是否有写入**，这种读锁是一种乐观锁。

乐观锁的意思就是**乐观地估计读的过程中大概率不会有写入**，因此被称为乐观锁。反过来，悲观锁则是读的过程中拒绝有写入，也就是写入必须等待。显然乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。

一个使用 StampedLock，即乐观锁的例子如下：

```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100, 200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            stamp = stampedLock.readLock(); 
            // 获取一个悲观读锁。
            // 这里的悲观锁不能保证数据一定是之前的 x y，但是可以保证数据是同一版本的 x y。
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
        /* 不必关心释放悲观锁到 return 之间是否还会出现数据修改，因为对读加锁，本质上是为了保证数据的一致性，即此次读到的数据是同一个版本的 */
    }
}
```

看得出来，写锁的使用方法和 ReadWriteLock 的使用方法一致。

注意到首先我们通过 `tryOptimisticRead()` 获取一个乐观读锁，并返回版本号。接着进行读取，读取完成后，我们通过 `validate()` 去验证版本号，如果在读取过程中没有写入，版本号不变，验证成功，我们就可以放心地继续后续操作。**如果在读取过程中有写入，版本号会发生变化，验证将失败。在失败的时候，我们再通过获取悲观读锁再次读取。由于写入的概率不高，程序在绝大部分情况下可以通过乐观读锁获取数据，极少数情况下使用悲观读锁获取数据**。

可见，`StampedLock` 把读锁细分为了乐观锁和悲观锁，能进一步提升并发效率。但这也是有代价的：

* 代码更加复杂。

* `StampedLock` 是不可重入锁，不能在一个线程中反复获取同一个锁。

  > 举一个例子，如果当前线程已经获取了写锁，那么再次重复获取的话（不论写锁还是读锁）就会**死锁**。

此外，还提供了更复杂的将悲观读锁**升级**为写锁的功能（因为不是可重入的锁，所以这里是升级），它主要使用在 if-then-update 的场景：即先读，如果读的数据满足条件，就返回，如果读的数据不满足条件，再尝试写。

### 1.8.使用 Semaphore（信号量与 pv 操作）

前面我们讲了各种锁的实现，本质上锁的目的就是保护一种受限资源，保证同一时刻只有**一个**线程能访问（ReentrantLock），或者只有**一个**线程能写入（ReadWriteLock）。还有一种受限资源，它需要保证同一时刻最多有 **N 个**线程能访问，比如同一时刻最多创建 100 个数据库连接，最多允许 10 个用户下载等。

**这种限制数量的锁，如果用 Lock 数据来实现就太麻烦了，可以使用 `Semaphore`**，例如，最多允许 3 个线程访问：

```java
public class AccessLimitControl {
    // 任意时刻仅允许最多3个线程获取许可:
    final Semaphore semaphore = new Semaphore(3);
    // 相当于操作系统中的信号量，其具有 PV 操作。

    public String access() throws Exception {
        // 如果超过了许可数量,其他线程将在此等待:
        semaphore.acquire();
        try {
            // TODO:
            return UUID.randomUUID().toString();
        } finally {
            semaphore.release();
        }
    }
}
```

使用`Semaphore`先调用`acquire()`获取，然后通过`try ... finally`保证在`finally`中释放。调用`acquire()`可能会进入等待，直到满足条件为止。也可以使用`tryAcquire()`指定等待时间：

```java
if (semaphore.tryAcquire(3, TimeUnit.SECONDS)) {
    // 指定等待时间3秒内获取到许可:
    try {
        // TODO:
    } finally {
        semaphore.release();
    }
}
// 如果等待时间3秒内没有获取到许可，则往下执行。
```

`Semaphore` 本质上就是一个信号计数器，用于限制同一时间的最大访问数量。

### 1.9.使用 Concurrent 集合（线程安全的 List、Map 等集合类）

我们在前面已经通过`ReentrantLock`和`Condition`实现了一个`BlockingQueue`：

```java
public class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```

`BlockingQueue` 的意思就是说，当一个线程调用这个`TaskQueue`的`getTask()`方法时，该方法内部可能会让线程变成等待状态，直到队列条件满足不为空，线程被唤醒后，`getTask()`方法才会返回。

因为 `BlockingQueue` 非常有用，所以我们不必自己编写，可以直接使用 Java 标准库的 `java.util.concurrent` 包提供的线程安全的集合：`ArrayBlockingQueue`。除了 `BlockingQueue` 外，针对 `List`、`Map`、`Set`、`Deque` 等，`java.util.concurrent`包也提供了对应的并发集合类。我们归纳一下：

| interface | non-thread-safe         | thread-safe                              |
| --------- | ----------------------- | ---------------------------------------- |
| List      | ArrayList               | CopyOnWriteArrayList                     |
| Map       | HashMap                 | ConcurrentHashMap                        |
| Set       | HashSet / TreeSet       | CopyOnWriteArraySet                      |
| Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
| Deque     | ArrayDeque / LinkedList | LinkedBlockingDeque                      |

使用这些并发集合与使用非线程安全的集合类完全相同。因为所有的同步和加锁的逻辑都在集合内部实现，因此对于外部调用者来说，只需要正常按接口引用，其他代码和原来的非线程安全代码完全一样。

>使用 `java.util.concurrent` 包提供的线程安全的并发集合可以大大简化多线程编程，多线程同时读写并发集合是安全的。
>
>尽量使用Java标准库提供的并发集合，避免自己编写同步代码。

### 1.10.使用 Atomic

Java的`java.util.concurrent`包除了提供底层锁、并发集合外，还提供了一组原子操作的封装类，它们位于`java.util.concurrent.atomic`包。以 `AtomicInteger` 为例，它提供的主要操作有：

* 









### 1.11.使用线程池





### 1.12.使用 Future













## 2.k8s deployment、Jenkins 和 gitlab 做 CICD

本地代码 => git push => gitlab => jenkins 流水线构建 => k8s deployment 自动更新 pod。



