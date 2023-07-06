# learn-java-1

## 1.使用 Maven 命令将 JAR 文件安装到本地 Maven 仓库

如果我们无法从中央仓库下载一个依赖，但是已经拿到了它的 JAR 包，我们可以将它手动导入（或者说安装）到 Maven 项目中。下面是一些步骤可以帮助你完成这个过程：

1. 在我们的 Maven 项目中创建一个名为 lib（或者其他名称）的目录，用于存放第三方 JAR 文件。

2. 将我们获得的 JAR 文件复制到 lib 目录中。

3. 打开命令行终端，进入到我们的项目根目录。

4. 运行以下 Maven 命令将该 JAR 文件安装到本地 Maven 仓库：

```bash
mvn install:install-file -Dfile=lib/your-jar-file.jar -DgroupId=com.example -DartifactId=your-artifact-id -Dversion=1.0 -Dpackaging=jar
```

5. 替换 your-jar-file.jar、com.example、your-artifact-id 和 1.0 为我们实际的值。这个命令会将 JAR 文件安装到本地 Maven 仓库中，使其可在项目中使用。

6. 保存并关闭 pom.xml 文件。

现在，Maven 将能够从本地仓库加载该 JAR 文件并将其包含在项目构建中。确保我们可以成功编译和运行项目，并且能够使用该依赖的功能。

需要注意的是，这种方式只适用于将无法从中央仓库下载的特定依赖添加到项目中。如果有可能，最好还是尽量从官方或信任的仓库获取依赖。

> 例如安装 purejavacomm-0.0.11.jar，如下：
>
> ```bash
> mvn install:install-file -Dfile=lib/purejavacomm-0.0.11.1.jar -DgroupId=org.jetbrains.pty4j -DartifactId=purejavacomm -Dversion=0.0.11.1 -Dpackaging=jar
> ```


## 2.常用注解介绍

### 2.1.@RequestMapping 与 @XxxMapping

`@RequestMapping` 是一个在 Spring 框架中使用的注解。它用于将 HTTP 请求映射到特定的处理方法（或控制器方法）上。通过 `@RequestMapping` 注解，我们可以指定处理请求的方法、URI 路径和请求的 HTTP 方法类型。当客户端发送具有匹配路径和方法类型的请求时，Spring 将调用带有 `@RequestMapping` 注解的方法来处理该请求。

`@RequestMapping` 注解可用于类级别时，用于为整个控制器类指定基本的请求映射路径。另外，`@GetMapping` 是 `@RequestMapping(method = RequestMethod.GET)` 的缩写，它是一个特定的注解，用于将 HTTP GET 请求映射到处理方法上。

>得注意的是，除了 `@GetMapping`，Spring 还提供了其他类似的 HTTP 方法映射注解，如 `@PostMapping`、`@PutMapping`、`@DeleteMapping` 等，分别用于映射 POST、PUT、DELETE 等不同的 HTTP 方法。

在Spring框架中，类级别上的`@RequestMapping`注解和方法级别上的`@XxxMapping`注解的路径信息**会进行拼接**。

### 2.2.@RestController，它与 @Controller 有什么区别？

`@RestController` 是一个注解，用于在 Spring MVC 中标记一个类为 RESTful Web 服务的控制器。它是 Spring Framework 中的一个特定类型的 `@Controller` 注解。在 Spring MVC 中，`@Controller` 注解用于标记一个类为控制器类，表示该类扮演着处理请求和返回响应的角色。通常，该类中的方法会使用 `@RequestMapping` 或其他相关注解来映射特定的 URL 请求路径，并根据请求进行处理并返回相应的视图或数据。

`@RestController` 注解是`@Controller` 的一个特殊变体。与 `@Controller` 不同的是，**`@RestController` 用于创建 RESTful Web 服务的控制器类，其中的方法会直接返回数据，而不是通过视图解析器渲染视图。这些方法的返回值会自动序列化为 JSON、XML 或其他格式的响应体。**

所以，两者都是用于标记类的注解，`@Controller` 用于传统的 Web 应用程序开发，而 `@RestController` 则适用于构建 RESTful Web 服务。总的来说，二者的区别如下所示：

1. **返回值处理方式不同**: 使用 `@Controller` 注解的类的方法通常返回视图名称或者 `ModelAndView` 对象，用于渲染视图。而使用 `@RestController` 注解的方法会直接将对象序列化为 JSON 或 XML 格式的响应，不会进行视图解析。
2. **默认行为不同**: `@Controller` 默认情况下会将方法返回的字符串解析为视图名称，并通过视图解析器将其转换为实际的视图。而 `@RestController` 默认情况下会将方法返回的对象直接序列化为 HTTP 响应体。
3. **使用场景不同**: `@Controller` 适用于传统的基于视图的 Web 应用程序开发，其中视图负责渲染 HTML 页面。而 `@RestController` 更适合用于构建 RESTful 风格的 Web 服务，其中数据以 JSON 或 XML 形式进行传输。

### 2.3.@Resource，它与 @Autowired 有什么区别？

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是 javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

两者有如下共同点：**两者都可以写在字段和setter方法上。两者如果都写在字段上（即字段注入），那么就不需要再写setter方法**。

同时，两者也有如下不同点：

1. @Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired，只按照byType注入。**@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false**。如果我们想使用按照名称（**byName**）来装配，可以结合 @Qualifier 注解一起使用。

   > @Qualifier 的使用示例如下所示，首先我们定义多个类：
   >
   > ```java
   > @Component
   >    @Qualifier("fooFormatter")
   > public class FooFormatter implements Formatter {
   >    public String format() {
   >        return "foo";
   >    }
   > }
   >    
   >    @Component
   >    @Qualifier("barFormatter")
   >    public class BarFormatter implements Formatter {
   >    public String format() {
   >        return "bar";
   >    }
   > }
   > ```
   >    
   >    之后，我们使用 `@Qualifier` 注解进行注入（记着一定要搭配 @Autowired）：
   >    
   >    ```java
   > @Component
   > public class FooService {
   > @Autowired
   >  @Qualifier("fooFormatter") // 指定注入。
   > private Formatter formatter;
   > 
   >  //todo 
   > }
   >    ```
   >    
   
2. @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。**所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略**。

   > @Resource 的使用示例如下所示：
   >
   > ```java
   > public class MyClass {
   >     @Resource
   >     private MyDependency myDependency;
   > 
   >     // ...
   > }
   > 
   > public class MyDependency {
   >     // ...
   > }
   > ```
   >
   > 需要注意的是 byName 的 name 是谁的 name，事实上是 Spring Bean 的 name，在 Spring 框架下，由于 IoC 的作用，所有的对象实时化均交由 Spring 管理，因此命名工作也交由 Spring 管理，Spring bean 的名称将根据类名生成，生成的默认名称遵循驼峰命名规则，即将类名的首字母小写，并且每个单词的首字母大写。 
   >
   > 对于 `public class MyDependency` 这个类，默认生成的bean名称将是 `myDependency`。如果我们想自定义 bean 的名称，您可以在 bean 上使用 `@Component` 注解或其衍生注解（如 `@Service`、`@Repository` 等），并在括号内指定所需的名称。如下所示：
   >
   > ```java
   > @Component("customName")
   > public class MyDependency {
   >     // ...
   > }
   > ```
   >
   > 在上述示例中，`MyDependency` 类将被注册为一个 bean，并且它的名称将是 `customName`。**需要注意的是，在整个应用程序上下文中，bean 的名称必须是唯一的。如果有多个 bean 具有相同的名称，那么将会抛出命名冲突的异常**。因此，请确保 bean 的名称在整个应用程序上下文中是唯一的。

   @Resource装配顺序如下：

   * 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。
   * 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。
   * 如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。
   * 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

   @Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

#### 2.3.1.为什么不推荐使用 @Autowired 进行字段注入？

在使用 IDEA 开发时，常可以看见 @Autowired 报警告如下，**不建议直接在字段上进行依赖注入。`Spring`开发团队建议：在`Java Bean`中永远使用构造方法进行依赖注入**。Spring 有三种依赖注入的方式如下：

1. 基于属性（filed，又称字段）的注入。这种注入方式就是在 bean 的变量上使用注解进行依赖注入。本质上是通过反射的方式直接注入到 field 。这是我平常开发中看的最多也是最熟悉的一种方式。比如：

   ```java
   @Autowired
   UserService userService;
   ```

2. 基于 set 方法的注入。通过对应变量的`setXXX()`方法以及在方法上面使用注解，来完成依赖注入。比如：

   ```java
   private UserService userService;
   
   @Autowired
   public void setUserService(UserService userService) {
       this.userService = userService;
   }
   ```

3. 基于构造器的注入。将各个必需的依赖全部放在带有注解构造方法的参数中，并在构造方法中完成对应变量的初始化，这种方式，就是基于构造方法的注入。比如：

   ```java
   private final UserService userService;
   
   @Autowired
   public UserController(UserService userService) {
       this.userService = userService;
   }
   ```

变量（filed）注入的方式是如此的简洁。但实际上他是有问题的，下面给出该问题的示例。

```java
@Autowired
private UserService userService;

private String company;

public UserServiceImpl() {
    this.company = userService.getCompany();
}
```

编译过程不会报错，但是运行之后报`NullPointerException`

```java
Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [...]: Constructor threw exception; nested exception is java.lang.NullPointerException
```

Java 在初始化一个类时，是按照静态变量或静态语句块 –> 实例变量或初始化语句块 –> 构造方法 -> @Autowired 的顺序。所以在执行这个类的构造方法时，user对象尚未被注入，它的值还是 null。

> 一个初始化类的代码流程示例如下所示：
>
> ```java
> public class MyClass {
>     // 静态变量
>     private static int staticVariable;
>     
>     // 实例变量
>     private int instanceVariable;
>     
>     // 静态语句块
>     static {
>         staticVariable = 10;
>         System.out.println("Static block executed.");
>     }
>     
>     // 初始化语句块
>     {
>         instanceVariable = 20;
>         System.out.println("Instance block executed.");
>     }
>     
>     // 构造方法
>     public MyClass() {
>         System.out.println("Constructor executed.");
>     }
>     
>     // 使用 @Autowired 注解注入依赖项
>     @Autowired
>     private SomeDependency dependency;
> }
> ```
>
> 当你创建一个 `MyClass` 的实例时，按照上述顺序执行。输出结果将会是：
>
> ```java
> Static block executed.
> Instance block executed.
> Constructor executed.
> ```

为了避免这样的问题，**我们可以使用 @Autowired 构造器注入**：

```java
private UserService userService;
private String company;

@Autowired
public UserServiceImpl(UserService userService) {
    this.userService = userService;
    this.company = userService.getCompany();
}
```

另外，我们还可以考虑将对 `userService` 的使用移至构造函数之外的其他方法中，或者使用`@PostConstruct`注解标记一个初始化方法，在该方法中执行与`userService`相关的逻辑。这样可以确保在初始化阶段完成后再使用`userService`。例如：

```java
@Autowired
private UserService userService;

private String company;

@PostConstruct
public void init() {
    this.company = userService.getCompany();
}
```

> `@PostConstruct` 注解是Java中的一种注解，它用于标记一个方法，指示在依赖注入完成后要执行该方法，该方法将在对象创建并完成依赖项注入后被自动调用。具体来说，`@PostConstruct` 注解有以下作用：
>
> 1. 初始化方法：通过将`@PostConstruct`注解应用于某个方法，可以指定该方法作为对象的初始化方法。在对象创建和依赖注入完成后，容器会自动调用带有`@PostConstruct`注解的方法，以执行对象的初始化逻辑。
> 2. 依赖项完全注入后执行：`@PostConstruct`注解确保在依赖项注入完成后才执行标记的方法。这可以防止在对象还没有完全初始化或依赖项尚未准备好时访问不正确的状态。
> 3. 替代构造函数中的逻辑：在一些情况下，由于某些限制或设计约束，无法使用构造函数来进行初始化逻辑。此时，可以使用`@PostConstruct`注解的方法来替代构造函数中的逻辑，以确保在对象实例化后执行必要的初始化操作。
>
> 需要注意的是，`@PostConstruct`注解通常用于受容器管理的Bean中，如Spring框架的组件、服务或控制器等。它只能应用于非静态方法，并且不能与静态初始化块或构造函数一起使用。

### 2.4.@PathVariable

`@PathVariable` 是一个 Spring 框架中的注解，用于将 URL 中的变量值映射到方法参数上。在 Spring MVC 或 Spring WebFlux 中，我们经常需要从 URL 中获取参数值。使用 `@PathVariable` 注解，我们可以直接在方法参数上指定要获取的参数，并且框架会自动将 URL 中对应的部分值绑定到该参数上。

例如，假设有一个 RESTful API 的 URL：`/users/{userId}`，其中 `{userId}` 是一个动态的变量部分，表示用户的 ID。我们可以在处理该请求的方法中使用 `@PathVariable` 注解来获取这个变量的值。下面是一个示例：

```java
@RestController
public class UserController {

    @GetMapping("/users/{userId}")
    public String getUserById(@PathVariable("userId") Long userId) {
        // 根据用户 ID 查询用户信息并返回
        return "User ID: " + userId;
    }
}
```

在上面的示例中，方法 `getUserById` 使用了 `@PathVariable` 注解，并指定了参数名为 `"userId"`。当收到 `/users/123` 这样的请求时，Spring 框架会自动将路径中的 `123` 值绑定到 `userId` 参数上，然后我们就可以在方法体中使用该值进行业务逻辑的处理。需要注意的是，`@PathVariable` 注解还支持更多的选项，比如可以指定默认值、正则表达式等，以满足不同的需求。

对于参数获取的注解，除了 @PathVariable 之外，还有：

* @RequestParam，用来获取 query 里的参数。此外，它也可以接受请求体中的参数，使用 `application/x-www-form-urlencoded` 或 `multipart/form-data` 编码格式。
* @RequestBody，用来获取请求体里的 json 参数。
* @RequestHeader，用来获取请求头里的参数。

## 3.Swagger 是什么

在前后端分离开发的过程中，前端和后端需要进行api对接进行交互，就需要一个api规范文档，方便前后端的交互，**但api文档不能根据代码的变化发生实时动态的改变，这样后端修改了接口，前端不能及时获取最新的接口**，导致调用出错，需要手动维护api文档，加大了开发的工作量和困难，而swagger的出现就是为了解决这一系列的问题。

Swagger 是一个开源的 API 设计和文档工具，它可以帮助开发人员更快、更简单地设计、构建、文档化和测试 RESTful API。Swagger 可以自动生成交互式 API 文档、客户端 SDK、服务器 stub 代码等，从而使开发人员更加容易地开发、测试和部署 API。

Swagger 基于OpenAPI规范构建，使用RestApi，具有如下特性：

1. 代码变，文档变。
2. 跨语言，支持多种语言。
3. swagger-ui 呈现出来的是一份可交互式的API文档，可以直接在文档页面尝试API的调用。
4. 可以将文档规范导入相关工具（postman、soapui），这些工具将会为我们自动地创建自动化测试。

使用 Swagger 时如果碰见版本更新迭代时，只需要更改 Swagger 的描述文件即可，但是在频繁的更新项目版本时很多开发人员认为即使修改描述文件（yml或json文件）也是一定的工作负担，久而久之就直接修改代码，而不去修改描述文件了，这样基于描述文件生成接口文档也失去了意义。

Marty Pitt 编写了一个基于 Spring 的组件 `swagger-springmvc`，Spring-fox 就是根据这个组件发展而来的全新项目；Spring-fox 是根据代码生成接口文档，所以正常的进行更新项目版本，修改代码即可，而不需要跟随修改描述文件（yml 或 json 文件）。Spring-fox 利用自身的 AOP 特性，把 Swagger 集成进来，底层还是 Swagger，但是使用起来却方便很多，所以在实际开发中，都是直接使用 Spring-fox。

>Springfox是Swagger在Java Spring框架下的一个实现。它提供了与Spring集成的功能，使得在Spring Boot项目中使用Swagger变得更加容易。**Springfox通过自动扫描Spring应用程序的注解，生成Swagger文档，并提供了一个内嵌的Swagger UI界面**。
>
>因此，Swagger和Springfox之间的区别在于它们的定位和使用方式。**Swagger是一种通用的API描述规范和工具集，可以在不同的平台和语言中使用。而Springfox是Swagger在Java Spring框架下的具体实现，提供了与Spring集成的特性和便利。**

### 3.1.Swagger 的常用注解分析

Swagger 注解主要是用来给 Swagger 生成的接口文档说明用的，下面将按照列表顺序进行分别介绍：

- @Api：修饰整个类，描述 Controller 的作用
- @ApiOperation：描述一个类的一个方法，或者说一个接口
- @ApiParam：单个参数描述
- @ApiModel：用对象来接收参数
- @ApiProperty：用对象接收参数时，描述对象的一个字段
- @ApiResponse：HTTP 响应其中 1 个描述
- @ApiResponses：HTTP 响应整体描述
- @ApiIgnore：使用该注解忽略这个API
- @ApiError：发生错误返回的信息
- @ApiImplicitParam：一个请求参数
- @ApiImplicitParams：多个请求参数

#### 3.1.1.@Api（一般用于 Controller）

@Api 是类上注解，控制整个类生成接口信息的内容，表示对类的说明。该注解有如下参数：

* tags=”说明该类的作用，非空时将覆盖 value 的值”。
* value=”描述类的作用” 其他参数。
* description 对 api 资源的描述，在 1.5 版本后不再支持。
* basePath 基本路径可以不配置，在 1.5 版本后不再支持。
* position 如果配置多个 Api 想改变显示的顺序位置，在 1.5 版本后不再支持。
* produces 设置 MIME 类型列表（output），例：”application/json, application/xml”，默认为空。
* consumes 设置 MIME 类型列表（input），例：”application/json, application/xml”，默认为空。
* protocols 设置特定协议，例：http， https， ws， wss。
* authorizations 获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
* hidden 默认为 false，配置为 true 将在文档中隐藏。

#### 3.1.2.@ApiOperation（一般用于 XxxMapping，即 Controller 的方法）

@ApiOperation 用在请求的方法上，说明方法的用途、作用。该注解有如下参数：

- value=”说明方法的用途、作用”。
- notes=”方法的备注说明” 其他参数。
- tags 操作标签，非空时将覆盖value的值。
- response 响应类型（即返回对象）。
- responseContainer 声明包装的响应容器（返回对象类型）。有效值为 “List”, “Set” or “Map”。
- responseReference 指定对响应类型的引用。将覆盖任何指定的response（）类。
- httpMethod 指定HTTP方法，”GET”, “HEAD”, “POST”, “PUT”, “DELETE”, “OPTIONS” and “PATCH”。
- position 如果配置多个Api 想改变显示的顺序位置，在1.5版本后不再支持。
- nickname 第三方工具唯一标识，默认为空。
- produces 设置MIME类型列表（output），例：”application/json, application/xml”，默认为空。
- consumes 设置MIME类型列表（input），例：”application/json, application/xml”，默认为空。
- protocols 设置特定协议，例：http， https， ws， wss。
- authorizations 获取授权列表（安全声明），如果未设置，则返回一个空的授权值。
- hidden 默认为false， 配置为true 将在文档中隐藏。
- responseHeaders 响应头列表。
- code 响应的HTTP状态代码。默认 200。
- extensions 扩展属性列表数组。

#### 3.1.3.@ApiImplicitParam（一般用于 XxxMapping，即 Controller 的方法）

@ApiImplicitParam 用在请求的方法上，表示一组参数说明。另外有 @ApiImplicitParam，其用在 @ApiImplicitParams 注解中，指定一个请求参数的各个方面。该注解有如下参数：

- name：参数名，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致
- value：参数的汉字说明、解释
- required：参数是否必须传，默认为 false （路径参数必填）
- paramType：参数放在哪个地方
- header 请求参数的获取：@RequestHeader
- query 请求参数的获取：@RequestParam
- path（用于 restful 接口）–> 请求参数的获取：@PathVariable
- body（不常用）
- form（不常用）
- dataType：参数类型，默认 String，其它值 dataType=”Integer”
- defaultValue：参数的默认值 其他参数（@ApiImplicitParam）：
- allowableValues 限制参数的可接受值。1.以逗号分隔的列表 2.范围值 3.设置最小值/最大值
- access 允许从API文档中过滤参数。
- allowMultiple 指定参数是否可以通过具有多个事件接受多个值，默认为 false
- example 单个示例
- examples 参数示例。仅适用于 BodyParameters

一个使用实例如下：

```java
@ResponseBody
@PostMapping(value="/login")
@ApiOperation(value = "登录检测", notes="根据用户名、密码判断该用户是否存在")
@ApiImplicitParams({
  @ApiImplicitParam(name = "name", value = "用户名", required = false, paramType = "query", dataType = "String"),
  @ApiImplicitParam(name = "pass", value = "密码", required = false, paramType = "query", dataType = "String")
})
public UserModel login(@RequestParam(value = "name", required = false) String account,
@RequestParam(value = "pass", required = false) String password){}
```

> 有一个与 @ApiImplicitParam 相类似的 @ApiParam，但是由于其使用与 @RequestParam 冲突，因此很少被使用，此处我们就不做过多介绍了。

#### 3.1.4.@ApiModel（一般用于 Pojo）

@ApiModel 一般用于响应类上，表示一个返回响应数据的信息（这种一般用在 POST 创建的时候，使用 @RequestBody 这样的场景，请求参数无法使用 @ApiImplicitParam 注解进行描述的时候）。

#### 3.1.5.@ApiModelProperty（一般用于 Pojo 的属性上）

@ApiModelProperty 用在属性上，描述响应类的属性。该注解有如下参数：

- value 此属性的简要说明。
- name 允许覆盖属性名称。
- allowableValues 限制参数的可接受值。1.以逗号分隔的列表 2.范围值 3.设置最小值/最大值。
- access 允许从 API 文档中过滤属性。
- notes 目前尚未使用。
- dataType 参数的数据类型。可以是类名或者参数名，会覆盖类的属性名称。
- required 参数是否必传，默认为 false。
- position 允许在类中对属性进行排序，默认为 0。
- hidden 允许在 Swagger 模型定义中隐藏该属性。
- example 属性的示例。
- readOnly 将属性设定为只读。
- reference 指定对相应类型定义的引用，覆盖指定的任何参数值。

#### 3.1.6.@ApiResponse（一般用于 XxxMapping，即 Controller 的方法）

@ApiResponse 用在请求的方法上，表示一组响应。此外，@ApiResponse 将会用在 @ApiResponses 中，一般用于表达一个错误的响应信息。该注解有如下参数：

- code：数字，例如 400。
- message：信息，例如 “请求参数没填好”。
- response：响应，如果为错误则为抛出异常的类。

一个代码示例如下：

```java
@ResponseBody
@PostMapping(value="/update/{id}")
@ApiOperation(value = "修改用户信息",notes = "打开页面并修改指定用户信息")
@ApiResponses({
    @ApiResponse(code=400,message="请求参数没填好"),
    @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
})
public JsonResult update(@PathVariable String id, UserModel model){}
```

## 4.DTO、DAO 与 POJO

下面以 Java 语言为例，给出 DTO、DAO 与 POJO 的示例：

DTO 示例：

```java
public class UserDTO {
    private String username;
    private String email;
    // ... 其他属性
    
    // 构造函数、Getter 和 Setter 方法
}
```

DAO 示例：

```java
public interface UserDao {
    void save(User user);
    User findById(int userId);
    List<User> findAll();
    void update(User user);
    void delete(int userId);
}
```

```java
public class UserDaoImpl implements UserDao {
    // 实现数据库访问逻辑，例如使用 JDBC 进行数据库操作
    // ...
}
```

POJO 示例：

```java
public class User {
    private int id;
    private String username;
    private String password;
    // ... 其他属性
    
    // 构造函数、Getter 和 Setter 方法
}
```

## 5.什么是 Kubernetes 中的 Deployment

Deployment 是 Kubernetes 中用于管理应用程序部署的重要资源对象。它提供了声明式部署、滚动更新、自愈能力和动态调整副本数等功能，帮助开发人员更方便地管理和控制他们的应用程序在 Kubernetes 集群中的运行状态。

在 motanni 项目中，Deployment 被视为一个应用单元。一个 Deployment 可以管理多个 Pod，这些 Pod 通常具有相同的容器镜像，但在滚动更新过程中，可以逐步替换副本中的镜像以实现应用程序的更新。

> Kubernetes 中的应用一般指代 Deployment。

## 6.kubernetes 中的 admin.conf 是什么?

在 Kubernetes（K8s）应用中，`admin.conf` 是一个配置文件，它包含了管理员权限的配置信息。该文件通常用于控制和管理 Kubernetes 集群。

`admin.conf` 文件是用于访问 Kubernetes API Server 的身份验证凭据和配置信息的一部分。它包含了与集群通信所需的参数，例如 API Server 的地址、证书以及其他安全设置。

通过 `admin.conf` 文件，管理员可以使用命令行工具（如 `kubectl`）或其他客户端工具与 Kubernetes 集群进行交互，并执行管理操作，如创建、修改或删除资源对象（如 Pod、Service、Deployment 等）。

通常情况下，`admin.conf` 文件由 Kubernetes 管理员生成，并分发给有权访问和管理集群的用户。这个文件需要保密，因为它包含了敏感的安全凭据和配置信息，只应该提供给受信任的用户和系统。

请注意，具体的 `admin.conf` 文件路径和内容可能会因不同的 Kubernetes 发行版或部署方式而有所不同。

> k8s 版本要和 client 版本匹配（真是蛋疼。
>
> **如果新项目基于既有项目进行开发，那么一定要注意不要被既有项目的代码给影响到，适当地注释会很有帮助。**











