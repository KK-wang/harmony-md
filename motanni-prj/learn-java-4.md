# learn-java-4

## 1.什么是 stomp，它与 websocket 有什么区别?







## 2.完全理解TCP/UDP、HTTP长连接、Websocket、SockJS/Socket.IO以及STOMP的区别和联系

https://blog.csdn.net/qq_32331073/article/details/82665419





## 3.websocket 和 socket 之间的关系

https://juejin.cn/post/6844903534446526471





## 4.WebSocket 能和 Socket 一样传输 raw 数据么？

WebSocket 协议是一种基于 TCP 的协议，它可以在客户端和服务器之间建立双向通信通道，支持传输文本和二进制数据，因此完全可以像 Socket 一样传输 raw 数据。WebSocket 协议还支持分片传输，可以将大型的数据分成多个小片段进行传输，以减少网络传输的负载和延迟。但是需要注意的是，WebSocket 协议使用了一些特定的数据帧格式和协议头，因此在使用 WebSocket 传输 raw 数据时，需要在客户端和服务器之间协商好数据的格式和编码方式，以确保数据的正确传输和解析。

## 5.Springboot websocket 中的 @Autowired 不能注入

spring 或 springboot 的 websocket 里面使用 @Autowired 注入 service 或 bean 时,报java.lang.NullPointerException异常,service为null(实际上并不是不能被注入)。

将要注入的 service 改成 static，然后添加一个set方法(重要的事情说三遍:这个set方法一定不能是static的,这个set方法一定不能是static的,这个set方法一定不能是static的)就不会为null了。

出现这种情况的本质原因是 **websocket 是多对象的，每个用户的聊天客户端对应 java 后台的一个 websocket 对象，前后台一对一实时连接，所以 websocket 不可能像 servlet 一样做成单例的，让所有聊天用户连接到一个 websocket对象，这样无法保存所有用户的实时连接信息。可能 spring 开发者考虑到这个问题，没有让 spring 创建管理 websocket ，而是由 java 原来的机制管理websocket ，所以用户聊天时创建的 websocket 连接对象不是 spring 创建的，spring 也不会为不是他创建的对象进行依赖注入，所以如果不用static关键字，每个 websocket 对象的 service 都是 null**。



## 6.为什么删除了 pod 还会自动启动？

这是因为 deployment 和 replicaset 的原因，他们会保证 pod 的数量。将二者均删除就可以保证被删除的 pod 不会再启动了。

另外，replicaset 还可以用于控制 deployment 的版本信息：

在 Kubernetes 中，Deployment 用于管理应用程序的部署和更新。它定义了所需的副本数和容器映像等信息，并确保所需状态与实际状态匹配。Deployment 在内部使用 `replicaSets` 对象来实现副本管理。`replicaSets` 定义了期望的副本数量，并根据需要创建、删除或调整副本。它还负责监视副本的运行状况，并在副本失败或被删除时进行自动修复。

当进行应用程序的更新时，Deployment 会创建一个新的 `replicaSet`，该 `replicaSet` 使用更新后的容器映像。然后，Deployment 会逐渐调整副本数量，将新的副本逐步引入，同时逐步停止旧的副本，从而实现无缝的更新过程。通过管理 `replicaSets`，Deployment 可以轻松地管理应用程序的版本信息。它可以跟踪每个版本的 `replicaSet`，并提供回滚、扩展和缩减的功能。这样，您可以轻松地切换到先前的版本或者扩展正在运行的特定版本，以满足应用程序需求。

## 7.面向命令行和 chatGPT 使用 git

Git 的 rebase（变基）操作用于将一系列提交应用到另一个分支上。在进行 rebase 时，如果发生冲突，Git 将会暂停 rebase 进程，并提示您如何处理。如果我们希望删除已经进行的变基工作，那么可以采取命令：

```bash
rm -fr ".git/rebase-merge"
```

当 Git 在应用一系列提交时，如果发现冲突，它会暂停 rebase 进程，并提醒您手动解决冲突。解决合并冲突的过程需要您编辑相关文件，标记冲突已解决，并告知 Git 如何继续 rebase 进程。对于每个冲突文件，使用以下命令将其标记为已解决：

```bash
git add <conflicted_file>
```

一旦您解决了所有的冲突并将它们标记为已解决，继续执行 rebase 进程：

```bash
git rebase --continue
```

## 8.如何通过 Replicaset 获取 Development 的版本？版本什么时候会变化？

ReplicaSet 的 deployment.kubernetes.io/revision 注解反映了 Development 的版本。它会在以下情况下会发生变化：

1. 创建新的Deployment：当您创建一个新的Deployment对象时，Kubernetes会自动分配一个初始的版本号，并将其赋值给 deployment.kubernetes.io/revision 注解。
2. 更新Deployment的配置：当您对Deployment进行更新，例如更改Pod模板、副本数或其他相关配置时，Kubernetes会自动增加 deployment.kubernetes.io/revision 的值，以反映所做的更改。
3. 回滚到先前版本：如果您回滚Deployment以恢复到先前的版本，Kubernetes将更新 deployment.kubernetes.io/revision 字段以反映回滚操作。

## 9.什么是 k8s 中的 system:anonymous？

在 Kubernetes 中，system:anonymous 是一个预定义的用户身份（user identity）。它表示一个未经过身份验证或者未经过授权的用户。当一个请求没有提供任何身份验证信息时，Kubernetes 将自动将该请求关联到 system:anonymous 这个用户身份。

system:anonymous 用户身份的权限非常有限，通常只能访问一些公共资源，比如一些只读的 API 或者 Kubernetes Dashboard 中的一些页面。对于需要进行更高级别操作的请求，必须提供有效的身份验证信息，并且该身份必须经过授权，才能被 Kubernetes 接受。

需要注意的是，如果 Kubernetes 集群的访问控制（Access Control）机制被正确配置，那么 system:anonymous 用户身份将无法访问任何敏感资源。因此，在生产环境中，应该尽可能避免使用 system:anonymous 用户身份，而是使用有效的身份验证信息来提高安全性。

默认情况下，system:anonymous 没有权限访问 k8s 中的自定义资源（crd）。























