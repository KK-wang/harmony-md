# learn-2

## 1.反射

参考文章：https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512。

反射就是Reflection，Java的反射是指程序在**运行期**可以拿到一个对象的所有信息。反射是为了解决在运行期，对某个实例一无所知的情况下，如何调用其方法。

### 1.1.Class 类

Java 中的 `class` 是由 JVM 在执行过程中动态加载的，JVM在第一次读取到一种`class`类型时，将其加载进内存。

每加载一种`class`，JVM就为其创建一个`Class`类型的实例，并关联起来。注意：这里的`Class`类型是一个名叫`Class`的`class`。

以`String`类为例，当JVM加载`String`类时，它首先读取`String.class`文件到内存，然后，为`String`类创建一个`Class`实例并关联起来。

这个`Class`实例是JVM内部创建的，如果我们查看JDK源码，可以发现`Class`类的构造方法是`private`，只有JVM能创建`Class`实例，我们自己的Java程序是无法创建`Class`实例的。

由于JVM为每个加载的`class`创建了对应的`Class`实例，并在实例中保存了该`class`的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的`class`的所有信息。

这种通过`Class`实例获取`class`信息的方法称为**反射（Reflection）**。有三种方式获取一个 class 的 Class 实例：

* 直接通过一个`class`的静态变量`class`获取。
* 如果我们有一个实例变量，可以通过该实例变量提供的`getClass()`方法获取。
* 如果知道一个`class`的完整类名，可以通过静态方法`Class.forName()`获取。

因为`Class`实例在 JVM 中是唯一的，所以，上述方法获取的`Class`实例是同一个实例。可以用`==`比较两个`Class`实例。

**动态加载**：

JVM在执行Java程序的时候，并不是一次性把所有用到的class全部加载到内存，而是第一次需要用到class时才加载。

动态加载`class`的特性对于Java程序非常重要。利用JVM动态加载`class`的特性，我们才能在运行期根据条件加载不同的实现类：

```java
// Commons Logging优先使用Log4j:
LogFactory factory = null;
if (isClassPresent("org.apache.logging.log4j.Logger")) {
    factory = createLog4j();
} else {
    factory = createJdkLog();
}

boolean isClassPresent(String name) {
    try {
        Class.forName(name);
        // class.forName 的作用是将加载指定名称的类到 JVM 上，其返回值是一个 Class 实例。
        return true;
    } catch (Exception e) {
        return false;
    }
}
```

这就是为什么我们只需要把Log4j的jar包放到classpath中，Commons Logging就会自动使用Log4j的原因。

### 1.2.访问字段

`Class`类提供了以下几个方法来获取字段：

- Field getField(name)：根据字段名获取某个public的field（包括父类）。
- Field getDeclaredField(name)：根据字段名获取当前类的某个field（不包括父类）。
- Field[] getFields()：获取所有public的field（包括父类）。
- Field[] getDeclaredFields()：获取当前类的所有field（不包括父类）。

通过 Field 实例，我们能够取到指定实例的字段值，同样地也可以设置字段的值。

> 感觉 Class 访问字段有点像 JavaScript 中的 Proxy。

### 1.3.调用方法

我们已经能通过 `Class` 实例获取所有`Field`对象，同样的，可以通过`Class` 实例获取所有`Method`信息。`Class`类提供了以下几个方法来获取`Method`：

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）。
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）。
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）。
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）。

和访问字段类似的，可以通过 Method.invoke(Object) 来调用方法，当调用的是静态方法时，传入参数 null。

在多态情况下，使用反射调用方法时，仍然遵循多态原则：即总是调用实际类型的覆写方法（如果存在）。

### 1.4.调用构造关系

我们通常使用`new`操作符创建新的实例：

```java
Person p = new Person();
```

如果通过反射来创建新的实例，可以调用Class提供的newInstance()方法：

```java
Person p = Person.class.newInstance();
```

调用 Class.newInstance() 的局限是，它只能调用该类的public无参数构造方法。如果构造方法带有参数，或者不是public，就无法直接通过Class.newInstance() 来调用。

为了调用任意的构造方法，Java的反射API提供了Constructor对象，它包含一个构造方法的所有信息，可以创建一个实例。Constructor对象和Method非常类似，不同之处仅在于它是一个构造方法，并且，调用结果总是返回实例。

通过Class实例获取Constructor的方法如下：

- `getConstructor(Class...)`：获取某个`public`的`Constructor`。
- `getDeclaredConstructor(Class...)`：获取某个`Constructor`。
- `getConstructors()`：获取所有`public`的`Constructor`。
- `getDeclaredConstructors()`：获取所有`Constructor`。

注意`Constructor`总是当前类定义的构造方法，和父类无关，因此不存在多态的问题。

调用非`public`的`Constructor`时，必须首先通过`setAccessible(true)`设置允许访问。`setAccessible(true)`可能会失败。

### 1.5.获取继承关系

有了`Class`实例，我们还可以获取它的父类的`Class`，通过 `getSuperclass` 的方式。

由于一个类可能实现一个或多个接口，通过`Class`我们就可以查询到实现的接口类型。例如，查询`Integer`实现的接口：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Class s = Integer.class;
        Class[] is = s.getInterfaces();
        for (Class i : is) {
            System.out.println(i);
        }
    }
}
```

> 此外，对所有`interface`的`Class`调用`getSuperclass()`返回的是`null`，获取接口的父接口要用`getInterfaces()`。

**继承关系**

当我们判断一个实例是否是某个类型时，正常情况下，使用`instanceof`操作符，如果是两个`Class`实例，要判断一个向上转型是否成立，可以调用`isAssignableFrom()`。

### 1.6.动态代理

所有`interface`类型的变量总是通过某个实例向上转型并赋值给接口类型变量的。有没有可能不编写实现类，直接在运行期创建某个`interface`的实例呢？

这是可能的，因为Java标准库提供了一种动态代理（Dynamic Proxy）的机制：**可以在运行期动态创建某个`interface`的实例**。

我们仍然预先定义了接口 `Hello`，但是我们并不去编写实现类，而是直接通过 JDK 提供的一个 `Proxy.newProxyInstance()` 创建了一个`Hello`接口对象。**这种没有实现类但是在运行期动态创建了一个接口对象的方式，我们称为动态代码。JDK提供的动态创建接口对象的方式，就叫动态代理**。

> 动态代理实际上是JVM在运行期动态创建class字节码并加载的过程。

动态代理是通过`Proxy`创建代理对象，然后将接口方法“代理”给`InvocationHandler`完成的。

****

动态代理的实现需要借助反射来获取方法信息，此外，反射在这里也起到了生成接口的临时实现类（或者说临时代理类）的作用。

知乎的这个实例解释了动态代理的实现框架：https://www.zhihu.com/question/20794107/answer/23330381。

## 2.注解

参考文章：https://www.liaoxuefeng.com/wiki/1252599548343744/1255945389098144。

### 2.1.使用注解

注解是放在Java源码的类、方法、字段、参数前的一种特殊“注释”。

注释会被编译器直接忽略，注解则可以被编译器打包进入class文件，因此，注解是一种用作标注的“元数据”。

Java 的注解有三类：

* 由编译器使用的注解。
* 由工具处理 `.class` 文件使用的注解。比如有些工具会在加载class的时候，对class做动态修改，实现一些特殊的功能。这类注解会被编译进入`.class`文件，但加载结束后并不会存在于内存中。这类注解只被一些底层库使用，一般我们不必自己处理。
* 在程序运行期能够读取的注解，它们在加载后一直存在于JVM中，这也是最常用的注解。

### 2.2.定义注解

Java语言使用`@interface`语法来定义注解（`Annotation`），它的格式如下：

```java
public @interface Report {
    String value() default "";
}
```

Java 提供了一些元注解，他们用于修饰注解：

* `@Target`，使用`@Target`可以定义`Annotation`能够被应用于源码的哪些位置。
* `@Retention`，定义了`Annotation`的生命周期。
* `@Repeatable`，这个元注解可以定义`Annotation`是否可重复。
* `@Inherited`，定义子类是否可继承父类定义的`Annotation`。

### 2.3.处理注解

根据`@Retention`的配置：

- `SOURCE`类型的注解在编译期就被丢掉了；
- `CLASS`类型的注解仅保存在class文件中，它们不会被加载进JVM；
- `RUNTIME`类型的注解会被加载进JVM，并且在运行期可以被程序读取。

我们只讨论如何读取`RUNTIME`类型的注解。因为注解定义后也是一种`class`，所有的注解都继承自`java.lang.annotation.Annotation`，因此，读取注解，需要使用反射API。

判断某个注解是否存在于`Class`、`Field`、`Method`或`Constructor`：

- `Class.isAnnotationPresent(Class)`。
- `Field.isAnnotationPresent(Class)。`
- `Method.isAnnotationPresent(Class)`。
- `Constructor.isAnnotationPresent(Class)。`

使用反射API读取Annotation：

- `Class.getAnnotation(Class)`。
- `Field.getAnnotation(Class)`。
- `Method.getAnnotation(Class)`。
- `Constructor.getAnnotation(Class)`。

读取方法参数的注解要麻烦一点，因为方法参数本身可以看成一个数组，而每个参数又可以定义多个注解，所以，一次获取方法参数的所有注解就必须用一个二维数组来表示。第一个维度表示参数个数，第二个维度表示对应参数的注解个数。

> 处理注解需要自己编写逻辑，并且是在需要处理的地方手动调用编写逻辑。

## 3.泛型

参考文章：https://www.liaoxuefeng.com/wiki/1252599548343744/1255945193293888。

### 3.1.泛型的定义

模板类，接受一个类作为参数来填充模板的定义：

```java
public class ArrayList<T> {
    private T[] array;
    private int size;
    public void add(T e) {...}
    public void remove(int index) {...}
    public T get(int index) {...}
}
```

泛型在向上转型时需要保证参数类，即 T，完全一致。

> 实际上，编译器根本就不允许把 `ArrayList<Integer>` 转型为 `ArrayList<Number>`。
>
> `ArrayList<Integer>` 和 `ArrayList<Number>` 两者完全没有继承关系。

### 3.2.泛型的使用

编译器看到泛型类型`List<Number>`就可以自动推断出后面的`ArrayList<T>`的泛型类型必须是`ArrayList<Number>`，因此，可以把代码简写为：

```java
// 可以省略后面的Number，编译器可以自动推断泛型类型：
List<Number> list = new ArrayList<>();
```

除了在 List，ArrayList 等类中使用泛型之外，还可以在接口中使用泛型：

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

如果不指定泛型参数类型，编译器会给出警告，且只能将`<T>`视为`Object`类型。

### 3.3.编写泛型

编写语法比较简单：

```java
public class Pair<T> {
    private T first;
   	private T last;
}
```

但是在编写泛型类时要特别注意，泛型类 `<T>` 不能用于静态方法。这种约束是很好理解的，静态方法可以直接通过类名进行调用，而类名直接调用的代价就是无法指定泛型参数 T。

不过，我们可以在 `static` 修饰符后面独立加一个泛型参数 `<K>`，将其改写为泛型方法。

```java
public static <K> Pair<K> create(K first, K last) {
    return new Pair<K>(first, last);
}
```

> 但是在调用 create 方法时并不能指定 K 的具体类型，这和泛型类不一样，类型 K 的确定取决于传入的参数 first。

此外，泛型还可以定义多种类型：

```java
public class Pair<T, K> {}
Pair<String, Integer> p = new Pair<>("test", 123);
```

### 3.4.擦试法

泛型是一种类似”模板代码“的技术，不同语言的泛型实现方式不一定相同。Java语言的泛型实现方式是擦拭法（Type Erasure）。

所谓擦拭法是指，虚拟机对泛型其实一无所知，所有的工作都是编译器做的。例如，我们编写了一个泛型类`Pair<T>`，这是编译器看到的代码：

```java
public class Pair<T> {
    private T first;
}
```

而虚拟机根本不知道泛型。这是虚拟机执行的代码：

```java
public class Pair {
    private Object first;
}
```

因此，Java 使用擦拭法实现泛型，导致了：

* 编译器把类型`<T>`视为`Object`；
* 编译器根据`<T>`实现安全的强制转型（父转子）。

使用泛型的时候，我们编写的代码也是编译器看到的代码：

```java
Pair<String> p = new Pair<>("Hello", "world");
String first = p.getFirst();
String last = p.getLast();
```

而虚拟机执行的代码并没有泛型：

```java
Pair p = new Pair("Hello", "world");
String first = (String) p.getFirst();
String last = (String) p.getLast();
```

所以，Java的泛型是由编译器在编译时实行的，编译器内部永远把所有类型`T`视为`Object`处理，但是，在需要转型的时候，编译器会根据`T`的类型自动为我们实行安全地强制转型。

了解了Java泛型的实现方式——擦拭法，我们就知道了Java泛型的局限：

* `<T>`不能是基本类型，例如`int`，因为实际类型是`Object`，`Object`类型无法持有基本类型。更主要的是，无法从 Object 转化为 int。

* 泛型类的`Class`实例没带有泛型。因为`T`是`Object`，我们对`Pair<String>`和`Pair<Integer>`类型获取`Class`时，获取到的是同一个`Class`，也就是`Pair`类的`Class`。换句话说，所有泛型实例，无论`T`的类型是什么，`getClass()`返回同一个`Class`实例，因为编译后它们全部都是`Pair<Object>`。

* 无法判断带泛型的类型，原因和前面一样，并不存在`Pair<String>.class`，而是只有唯一的`Pair.class`。

* 不能实例化`T`类型，我们必须借助额外的`Class<T>`参数：
  ```java
  Pair<String> pair = new Pair<>(String.class);
  public class Pair<T> {
      public Pair(Class<T> clazz) {}
  }
  ```

此外，定义泛型方法时要防止重复定义方法，例如 `public boolean equals(T obj)`。

一个类可以继承自一个泛型类，例如继承式 `public class IntPair extends Pair<Integer>`，子类可以获取父类的泛型类型 `<T>`。

### 3.5.extends 通配符

假设有方法：

```java
static int add(Pair<Number> p) {}
```

我们可以传入 `new Pair<Number>(1, 2)`，但是实际参数类型是 `(Integer, Integer)`。但是当我们传入 `new Pair<Integer>(1, 2)` 时，编译报错了。

原因很明显，因为`Pair<Integer>`不是`Pair<Number>`的子类，因此，`add(Pair<Number>)`不接受参数类型`Pair<Integer>`。

>实际类型是`Integer`，引用类型是`Number`，没有问题。问题在于方法参数类型定死了只能传入`Pair<Number>`。

有没有办法使得方法参数接受`Pair<Integer>`？办法是有的，这就是使用`Pair<? extends Number>`使得方法接收所有泛型类型为`Number`或`Number`子类的`Pair`类型。

> 注意，`? extends Number` 只能用来定义方法参数，类不能使用。

这样一来，给方法传入`Pair<Integer>`类型时，它符合参数`Pair<? extends Number>`类型。**这种使用`<? extends Number>`的泛型定义称之为上界通配符（Upper Bounds Wildcards），即把泛型类型`T`的上界限定在`Number`了**。

但是如果将泛型参数由 T 改为 `? extends Number`，那么对于返回值是 `T` 的方法，其返回值也变成了 `? extends Number`，所以下面的代码是无法通过编译的：

```java
Integer x = p.getFirst();
```

这是因为实际的返回类型可能是`Integer`，也可能是`Double`或者其他类型，编译器只能确定类型一定是`Number`的子类（包括`Number`类型本身），但具体类型无法确定。

> 但是改写为 `Number x = p.getFirst();` 就可以通过编译了。

**对于泛型类变量赋值转移的情况，该变量最终的泛型类由其声明类决定**。举一个例子：

```java
Pair<Integer> p = new Pair<>(123, 456);
int n = add(p);
// ...
static int add(Pair<? extends Number> p) {
    // 原本 Pair<Integer> 的 p 传给 add 之后，其声明类变成了
    // Pair<? extends Number>，因此 Pair<T> 中的 T 也变成了
    // Pair<? extends Number>。
}
```

对于泛型类 `Pair<? extends Number>` 的任意一个含有参数类型为 `? extends Number` 的方法，我们将无法向其传递除 null 以外的参数，就算这个参数是 Number 或其子类，这是通配符泛型参数的一个重要限制。

> 因为本质上的类型，和允许你传进来的类型，可能是不匹配的，例如 `<Integer>` 和 `Double`。

使用类似`<? extends Number>`通配符作为方法参数时表示：

- 方法内部可以调用获取`Number`引用的方法，例如：`Number n = obj.getFirst();`；
- 方法内部无法调用传入`Number`引用的方法（`null`除外），例如：`obj.setFirst(Number n);`。

在定义泛型类型`Pair<T>`的时候，也可以使用`extends`通配符来限定`T`的类型：

```java
public class Pair<T extends Number> { ... }
```

`? extends Number` 和 `T extends Number` 两者最大的不同是，前者是任意 T 满足继承自 Number，而后者是选定一个 T 满足继承自 Number。

### 3.6.super 通配符

和 extends 的背景类似，已经有了 `Pair<Integer> p`，但是我们希望接受`Pair<Integer>`类型，以及`Pair<Number>`、`Pair<Object>`，因为`Number`和`Object`是`Integer`的父类。

我们使用 `? super Integer` 通配符来实现这一点。

>`Pair<? super Integer>`表示，方法参数接受所有泛型类型为`Integer`或`Integer`父类的`Pair`类型。

使用`<? super Integer>`通配符有如下限制：

- 允许调用`set(? super Integer)`方法传入`Integer`的引用；
- 不允许调用`get()`方法获得`Integer`的引用。唯一例外是可以获取`Object`的引用：`Object o = p.getFirst()`。

换句话说，使用`<? super Integer>`通配符作为方法参数，表示方法内部代码对于参数只能写，不能读。这和 `? extends Integer` 恰恰相反。

> `? extends` 允许都读不允许写，`? super` 允许写不允许读，这二者特性的最典型的应用就是 Java 标准库中 `copy()` 的实现。

何时使用`extends`，何时使用`super`？为了便于记忆，**我们可以用PECS原则：Producer Extends Consumer Super**。即：如果需要返回`T`，它是生产者（Producer），要使用`extends`通配符；如果需要写入`T`，它是消费者（Consumer），要使用`super`通配符。

我们已经讨论了`<? extends T>`和`<? super T>`作为方法参数的作用。实际上，Java的泛型还允许使用无限定通配符（Unbounded Wildcard Type），即只定义一个`?`：

```java
void sample(Pair<?> p) {}
```

因为`<?>`通配符既没有`extends`，也没有`super`，因此：

- 不允许调用`set(T)`方法并传入引用（`null`除外）；
- 不允许调用`T get()`方法并获取`T`引用（只能获取`Object`引用）。

换句话说，既不能读，也不能写，那只能做一些`null`判断。此外，`<?>`通配符有一个独特的特点，就是 `Pair<?>`是所有 `Pair<T>` 的超类。

### 3.7.泛型与反射

部分反射API是泛型，例如：`Class<T>`，`Constructor<T>`。

可以声明带泛型的数组，但不能直接创建带泛型的数组，必须强制转型（例如，添加转型 `Pair<String>[]`）。此外，可以通过`Array.newInstance(Class<T>, int)`创建`T[]`数组，需要强制转型。

> 使用泛型和可变参数时需要特别小心，如果在方法内部创建了泛型数组，最好不要将它返回给外部使用。





























