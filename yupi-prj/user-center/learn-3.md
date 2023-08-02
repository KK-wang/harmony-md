# learn-3

## 1.Serializable 有什么用？

> Java 序列化是一种将 Java 对象转换为字节流的机制，以便可以在网络上传输或保存到磁盘上。序列化是将对象的状态转换为字节流的过程，反序列化则是将字节流转换回对象的状态。
>
> 在 Java 中，可以使用 Serializable 接口来指示一个类可以被序列化。当一个类实现了 Serializable 接口时，它的对象可以被序列化和反序列化。

Serializable 是一个空接口，里面没有定义任何字段或者方法。

参考文章 1：https://www.jianshu.com/p/4935a87976e5。

参考文章 2：https://mp.weixin.qq.com/s/Zpb2OuZxJpWX2mow3qd-xg。

**总结**

实现序列化的原因：

1. 将对象的状态保存在存储媒体中以便可以在以后重新创建出完全相同的副本。
2. 按值将对象从一个应用程序域发送至另一个应用程序域。

> 实现serializable接口的作用是就是可以把对象存到字节流，然后可以恢复，所以你想如果你的对象没实现序列化怎么才能进行持久化和网络传输呢，要持久化和网络传输就得转为字节流，所以在分布式应用中及设计数据持久化的场景中，你就得实现序列化。

是不是每个实体类都要进行序列化？

看实体类是否需要持久化存储（**例如直接存储到文件中**）以及是否需要传输给另一个应用（**不经过 JSON 字符串转化直接传输**），没有的话就不需要，例如我们利用 fastjson 将实体类转化成 json 字符串时，并不涉及到转化为字节流，所以其实跟序列化没有关系。

有时没有实现序列化，依然可以持久化到数据库中，什么原因？

这个其实我们可以看看实体类中常用的数据类型，例如Date、String等等，它们已经实现了序列化，而一些基本类型，数据库里面有与之对应的数据结构，从我们的类声明来看，我们没有实现serializabel接口，其实是在声明的各个不同变量的时候，由具体的数据类型帮助我们实现了序列化操作。另外需要注意的是，在NoSql数据库中，并没有与我们java基本类型对应的数据结构，所以在往nosql数据库中存储时，我们就必须将对象进行序列化，同时在网络传输中我们要注意到两个应用中实体类的serialVersionUID要保持一致，不然就不能正常的进行反序列化。

## 2.docker rmi、docker kill 和 docker tag

* docker rmi，即删除本地一个或者多个镜像，其可有参数 `-f`，表示强制删除。
* docker kill，立刻强制终止容器，相当于加强版 docker stop。
* docker tag，用于标记镜像的版本，或者给镜像另外一个更为规范的命名。（主要是用于镜像的版本管理）

## 3.k8s ConfigMap 和 Secret

ConfigMap（类似于 Linux 系统中的 `/etc` 目录，专门用来存储配置文件的目录），参考文章：https://jimmysong.io/kubernetes-handbook/concepts/configmap.html。

> 其实 ConfigMap 功能在 Kubernetes1.2 版本的时候就有了，许多应用程序会从配置文件、命令行参数或环境变量中读取配置信息。这些配置信息需要与 docker image 解耦，你总不能每修改一个配置就重做一个 image 吧？**ConfigMap API 给我们提供了向容器中注入配置信息的机制，ConfigMap 可以被用来保存单个属性，也可以用来保存整个配置文件或者 JSON 二进制大对象**。

Secret，参考文章：https://jimmysong.io/kubernetes-handbook/concepts/secret.html。

>Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec 中。Secret 可以以 Volume 或者环境变量的方式使用。
>
>Secret 有三种类型：
>
>* **Service Account** ：用来访问 Kubernetes API，由 Kubernetes 自动创建，并且会自动挂载到 Pod 的 `/run/secrets/kubernetes.io/serviceaccount` 目录中；
>* **Opaque** ：base64 编码格式的 Secret，用来存储密码、密钥等。
>* **kubernetes.io/dockerconfigjson** ：用来存储私有 docker registry 的认证信息。

## 4.Spring 项目多环境实践

在 SpringBoot 项目中，可以通过配置文件 application.yaml 来实现 Spring 项目的多环境配置。

### 4.1.SpringBoot 多环境配置方法

参考文章：https://www.cnblogs.com/Chenjiabing/p/13968781.html

配置多环境主要可以分为两种方式：

1. SpringBoot 的多环境配置：

   * 创建三个不同的配置文件 `application.properties`、`application-dev.properties`、`application-prod.properties`。

   * 在 `application.properties` 中指定 `spring.profiles.active=dev`。如此一来，如果没有指定运行的环境，`Spring Boot` 默认会加载`application.properties`文件，而这个的文件又告诉`Spring Boot` 去找`dev`环境的配置文件。

     > 此外，也可以在运行 jar 包的时候指定环境，命令如下：
     >
     > `java -jar xxx.jar --spring.profiles.active=test`

2. Maven 的多环境配置。`Maven`本身也提供了对多环境的支持，不仅仅支持`Spring Boot`项目，只要是基于`Maven`的项目都可以配置。此外，`Maven`对于多环境的支持在功能方面更加强大，支持`JDK版本`、`资源文件`、`操作系统`等等因素来选择环境。详情戳文章。

### 4.2.多环境设计

参考文章：https://blog.csdn.net/weixin_41701290/article/details/120173283。

常用的多环境分为如下几种：

* 本地环境，local，是指前端或后端独立开发、自主测试的环境。通常就是让项目和依赖在我们自己的电脑上运行，比如数据库、缓存、队列等各种服务，可能需要自己在本地搭建。

  ![image-20230802170849689](./learn-3.assets/image-20230802170849689.png)

* 开发环境，dev，是指前端和后端（或者多个程序员）一起协作开发、联调的环境。通常将项目和依赖放在员工电脑可以直接访问的开发机上，不用自己搭建，直接跑起来项目，提高开发和协作效率。

  ![image-20230802170901514](./learn-3.assets/image-20230802170901514.png)

  >对规模不大的团队来说，开发和本地环境其实有一套就够了，毕竟本地也可以连接公用的数据库等服务。

* 测试环境，test， 是指前端和后端开发和联调完成，做出完整的新功能后，交给测试同学去找 Bug 的环境。

  ![image-20230802171023480](./learn-3.assets/image-20230802171023480.png)

  > 通常在测试环境需要有独立的测试数据库和其他服务，让测试同学大显身手。每次修改完 Bug 后，也都要再次发布项目到测试环境，让测试同学重新验证。

* 预发布环境，pre，这是和线上项目最接近的环境，一般是测试验证通过、产品经理体验过后，才能将项目发布到这个环境。

  ![image-20230802171222733](./learn-3.assets/image-20230802171222733.png)

* 生产环境，prod，又叫线上环境，是给所有真实用户使用的环境。其不能随意修改，且发布项目到该环境时必须格外小心。线上的数据库、机器等资源一般也是由专业的运维来负责，想要登录机器、修改配置，都需要经过严格审批。

  ![image-20230802171300337](./learn-3.assets/image-20230802171300337.png)

## 5.session-cookie 和 jwt 鉴权区别

>狭义上，我们通常认为 session 是「种在 cookie 上、数据存在服务端」的认证方案，token 是「客户端存哪都行、数据存在 token 里」的认证方案。对 session 和 token 的对比本质上是「客户端存 cookie / 存别地儿」、「服务端存数据 / 不存数据」的对比。

参考文章：https://zhuanlan.zhihu.com/p/281414244。

典型的 session-cookie 鉴权流程，如下：

![image-20230802172822176](./learn-3.assets/image-20230802172822176.png)

以上的**库（session）**有三种存储方式：

* redis：内存型数据库。以 key-value 的形式存，正合 sessionId-sessionData 的场景且访问快。
* 内存：直接放到变量里。一旦服务重启就没了。
* 数据库：普通数据库。性能不高。

典型的 token 鉴权流程：

![image-20230802173422552](./learn-3.assets/image-20230802173422552.png)

早期 token 的编码是不规范的，例如：

![image-20230802174031798](./learn-3.assets/image-20230802174031798.png)

后来的 JSON Web Token，即 JWT 解决了这个问题。一串典型的 JWT 为 `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJhIiwiaWF0IjoxNTUxOTUxOTk4fQ.2jf3kl_uKWRkwjOP6uQRJFqMlwSABcgqqcJofFH5XCo`。其生成方式如下：

![image-20230802174223061](./learn-3.assets/image-20230802174223061.png)

为了防止用户频繁登录，可以引入双 token 机制，即一个 access token，一个 refresh token。

* access token 用来访问业务接口，由于有效期足够短，盗用风险小，也可以使请求方式更宽松灵活
* refresh token 用来获取 access token，有效期可以长一些，通过独立服务和严格的请求方式增加安全性。另外，refresh token 暴露的频率也更低。

有了 refresh token 后，几种情况的请求流程变成这样：

![image-20230802175826778](./learn-3.assets/image-20230802175826778.png)

如果 refresh token 也过期了，就只能重新登录了。

### 5.1.单点登录











## 6.为什么需要通用的返回对象







## 7.为什么需要自定义异常以及全局异常处理





## 8.Java 枚举





## 9.@RestControllerAdvice 和 @Slf4j 是什么？





## 10.MyBatis Plus 中的 IService

为什么 Service 接口需要继承 IService？

为什么 Service 实现需要继承 ServiceImpl？

为什么 ServiceImpl 要引入 Mapper 泛型？

## 11.kubectl apply 相关命令

kubectl delete -f some-config.yaml。

这条命令是使用 Kubernetes 命令行工具 `kubectl` 删除一个 Kubernetes 资源对象，该资源对象的定义在文件 `some-config.yaml` 中。

具体来说，`-f` 标志告诉 `kubectl` 从文件中读取资源定义，而 `delete` 命令表示要删除资源。因此，`kubectl delete -f some-config.yaml` 命令会将 `some-config.yaml` 中定义的 Kubernetes 资源删除。



## 12.application.yaml 配置相关







# 附录

## 1.将应用部署到 k8s 上

将应用部署在 k8s 上有诸多好处，不论是容灾部署，还是热更新配置文件，此处不再赘述，下面以将 edgesec-adm 部署在 k8s 集群为例展示部署流程：

假设必要的部署环境已就绪，首先编写部署文件：

* edgesec-adm-cm.yaml，应用相关配置文件（此处即是 application.yaml），详见 ConfigMap 的作用。
* edgesec-adm-svc.yaml，应用网络配置文件（例如完成端口映射的工作），详见 Service 的作用。
* edgesec-adm.yaml，将应用部署到 k8s 上的关键文件，一般是 Deployment。

edgesec-adm.yaml 的文件内容如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: edgesec-adm
  namespace: hc-edgesec
  labels:
    app: edgesec-adm
spec:
  selector:
    matchLabels:
      app: edgesec-adm
  replicas: 1
  template:
    metadata:
      labels:
        app: edgesec-adm
    spec:
      containers:
        - name: edgesec-adm
          image: 10.10.102.224:8443/edgesec/edgesec-adm:v3.0 # 镜像。
          imagePullPolicy: IfNotPresent
          args: ["--spring.config.location=application.properties"]
          resources:
            requests:
              memory: "100Mi"
              cpu: "10m"
            limits:
              memory: "800Mi"
              cpu: "600m"
          volumeMounts:
            - name: edgesec-adm-config # 呼应下面的 valumes。
              mountPath: /etc/application.properties
              subPath: application.properties
      volumes:
        - name: edgesec-adm-config
          configMap:
            name: edgesec-adm-configmap # ConfigMap
            items:
              - key: application.properties
                path: application.properties
      nodeSelector:
        node: edgesec-master # 将 deployment 内的 pod 调度到具有该 label 的 node 上。
```

> service 选择 deployment，deployment 选择 configmap。

完成配置文件之后，如果之前使用过 edgesec-adm.yaml，那么使用 `kubectl delete -f edgesec-adm.yaml` 将过往影响删除。

之后为了符合 edgesec-adm.yaml 中的定义，将原有镜像打上 tag：

```bash
$ docker tag edgesec-adm:1.0  10.10.102.224:8443/edgesec/edgesec-adm:v3.0
```

由于项目使用的镜像仓库是 harbor，因此需要 `docker push 10.10.102.224:8443/edgesec/edgesec-adm`。

最后只需要再执行：`kubectl apply -f edgesec-adm.yaml` 即可。



























