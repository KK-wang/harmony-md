# learn-1

## 1.命令行参数

Java 程序的入口是 `main` 方法，而`main`方法可以接受一个命令行参数，它是一个`String[]`数组。

这个命令行参数由 JVM 接收用户输入并传给`main`方法。

我们可以利用接收到的命令行参数，根据不同的参数执行不同的代码。例如，实现一个`-version`参数，打印程序版本号：

```java
public class Main {
    public static void main(String[] args) {
        for (String arg : args) {
            if ("-version".equals(arg)) {
                System.out.println("v 1.0");
                break;
            }
        }
    }
}
```

从上面的例子我们不难引申出一个结论，所有基于 Java 开发的工具在基于命令行使用它时，都会调用 main 函数，而在 main 函数中开发者就能够使用各种命令解析工具来提供强大的命令行功能。但是在 main 函数看来，这些命令只是字符串而已。 

## 2.包

如果我们自己写了一个 `Array` 类，恰好JDK也自带了一个`Arrays`类，如何解决类名冲突？在Java中，我们使用`package`来解决名字冲突。

>在Java虚拟机执行的时候，JVM只看完整类名，因此，只要包名不同，类就不同。

**Java定义了一种名字空间，称之为包：`package`。一个类总是属于某个包，类名（比如`Person`）只是一个简写，真正的完整类名是`包名.类名`。**例如，小明的`Person`类存放在包`ming`下面，因此，完整类名是`ming.Person`。

我们使用 package 来声明包，一般来讲，包需要和文件路径保持一致。

> 要注意的是，虽然文件目录有父子关系，但是包并没有父子关系。java.util 和 java.util.zip 是不同的包，两者没有任何继承关系。

位于同一个包的类，可以访问包作用域的字段和方法。不用`public`、`protected`、`private`修饰的字段和方法就是包作用域。

此外，在一个 class 中，我们总会引用其他的`class`。如果这个其他的 class 和我们目前的这个 class 是同一个 package 声明，那么就不需要 import 这个其他的 class，如果不是同一个 package 声明，那么就需要引用。

在代码中，当编译器遇到一个`class`名称时：

* 如果是完整类名，就直接根据完整类名查找这个`class`。

* 如果是简单类名，按下面的顺序依次查找：

  - 查找当前`package`是否存在这个`class`。

  - 查找`import`的包是否包含这个`class`。

  - 查找`java.lang`包是否包含这个`class`。

    > 注意，自动导入的是java.lang包，但类似java.lang.reflect这些包仍需要手动导入。java 的包并没有父子关系。

为了避免名字冲突，我们需要确定唯一的包名。**推荐的做法是使用倒置的域名作为包前缀来确保唯一性**。例如：

- org.apache。
- org.apache.commons.log。
- com.liaoxuefeng.sample。

子包就可以根据功能自行命名。

## 3.内部类

有一种类，它被定义在另一个类的内部，所以称为内部类（Nested Class）。

**Inner Class**

如果一个类定义在另一个类的内部，这个类就是Inner Class：

```java
class Outer {
    class Inner {
        // 定义了一个Inner Class
    }
}
```

上述定义的`Outer`是一个普通类，而`Inner`是一个Inner Class，它与普通类有个最大的不同，就是Inner Class的实例不能单独存在，必须依附于一个Outer Class的实例。

```java
Outer.Inner inner = outer.new Inner();
```

这是因为Inner Class除了有一个`this`指向它自己，还隐含地持有一个Outer Class实例，可以用`Outer.this`访问这个实例。所以，实例化一个Inner Class不能脱离Outer实例。

Inner Class和普通Class相比，除了能引用Outer实例外，还有一个额外的“特权”，就是可以修改Outer Class的`private`字段，因为Inner Class的作用域在Outer Class内部，所以能访问Outer Class的`private`字段和方法。

观察Java编译器编译后的`.class`文件可以发现，`Outer`类被编译为`Outer.class`，而`Inner`类被编译为`Outer$Inner.class`。

**Anonymous Class**

还有一种定义Inner Class的方法，它不需要在Outer Class中明确地定义这个Class，而是在方法内部，通过匿名类（Anonymous Class）来定义。示例代码如下：

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, " + Outer.this.name);
    }
}
// new Runnable() 之后的花括号就代表**匿名类**。
```

观察`asyncHello()`方法，我们在方法内部实例化了一个`Runnable`。`Runnable`本身是接口，接口是不能实例化的，所以这里实际上是定义了一个实现了`Runnable`接口的匿名类，并且通过`new`实例化该匿名类，然后转型为`Runnable`。

匿名类和Inner Class一样，可以访问Outer Class的`private`字段和方法。之所以我们要定义匿名类，是因为在这里我们通常不关心类名，比直接定义Inner Class可以少写很多代码。

**Static Nested Class**

最后一种内部类和Inner Class类似，但是使用`static`修饰，称为静态内部类（Static Nested Class）。用`static`修饰的内部类和Inner Class有很大的不同，它不再依附于`Outer`的实例，而是一个完全独立的类，因此无法引用`Outer.this`，但它可以访问`Outer`的`private`静态字段和静态方法。

## 4.classpath 与 jar 包

参考文章：https://www.liaoxuefeng.com/wiki/1252599548343744/1260466914339296。

在 jar 包中，还可以包含一个特殊的`/META-INF/MANIFEST.MF`文件，`MANIFEST.MF`是纯文本，可以指定`Main-Class`和其它信息。JVM会自动读取这个`MANIFEST.MF`文件，如果存在`Main-Class`，我们就不必在命令行指定启动的类名，而是用更方便的命令：

```bash
java -jar hello.jar # 运行 java 程序。
```

在大型项目中，不可能手动编写`MANIFEST.MF`文件，再手动创建zip包。Java社区提供了大量的开源构建工具，例如 Maven，可以非常方便地创建jar包。

