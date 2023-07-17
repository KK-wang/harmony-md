# learn-java-3

## 1.如何通过节点亲和性将应用指定到某一个节点上去？

节点亲和性（Node Affinity）是Kubernetes中用来指定Pod与Node亲和性关系的一种机制，通过指定Pod的亲和性，可以让Kubernetes将Pod调度到特定的Node上。在Kubernetes中，有两种方式可以实现节点亲和性：

1. 节点亲和性（Node Affinity）：通过指定Pod应该部署到哪些Node上来实现亲和性。**可以通过指定Label Selector的方式来实现节点亲和性，比如将Pod部署到具有特定Label的Node上。**
2. 节点亲和性预选（Node Affinity Preemption）：如果没有满足Pod的节点亲和性条件的Node，Kubernetes可以将其他不符合Pod要求的节点上的Pod驱逐出去，以满足这个Pod的节点亲和性需求。

下面是一个示例，演示如何将一个Pod部署到具有特定Label的Node上。在这个示例中，我们将创建一个Pod，并将其部署到具有Label "my-label" 的Node上。首先，我们需要在Node上打上需要用到的Label，这可以通过kubectl命令来完成：

```
kubectl label nodes <node-name> my-label=enabled
```

接下来，我们需要为Pod指定节点亲和性。这可以通过Pod的spec属性来完成，具体如下所示：

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: my-label
            operator: In
            values:
            - enabled
  containers:
  - name: my-container
    image: nginx
```

在上面的YAML文件中，我们为Pod指定了节点亲和性，将Pod部署到具有Label "my-label=enabled" 的Node上。这可以通过requiredDuringSchedulingIgnoredDuringExecution参数来实现。在这个参数中，我们使用了nodeSelectorTerms来指定一个Node Selector，然后使用matchExpressions来指定Label Selector，将Pod部署到符合条件的Node上。

最后，我们可以使用kubectl命令来创建Pod：

```bash
$ kubectl apply -f my-pod.yaml
```

这样，Kubernetes就会将这个Pod部署到具有Label "my-label=enabled" 的Node上。

## 2.一文了解 k8s 中的 14 个概念

https://www.cnblogs.com/rancherlabs/p/14860697.html

### 2.1.node（节点）

节点是 Kubernetes 中的 worker 机器，可以是任何具有 CPU 和 RAM 的设备。例如，智能手表、智能手机或者笔记本，甚至是树莓派都可以成为一个节点。当我们使用云时，节点就是一个虚拟机（VM）。所以，简单来说，节点是单一设备的抽象概念。这种抽象的好处是，我们不需要知道底层的硬件结构。我们只使用节点，这样一来，我们的基础设施就独立于平台。

### 2.2.cluster（集群）











## 3.k8s 中，将 pod 部署到 deployment 上有什么好处？

https://www.zhihu.com/question/360252629







## 4.什么是 Helm？





## 5.Vimium 插件的使用方法

### 5.1.Vimium 的快捷键

1. j：向下细微滚动窗口；k：向上细微滚动窗口。
2. J：（Shift+j 的意思，以下大写全部表示加 Shift) 下一个标签页；K：上一个标签页。
3. d：向下滚动半个屏幕；u：向上移动半个屏幕。
4. 



https://www.cnplugins.com/office/vimium/

Vimium 的 insert 模式

## 6.Linux 中的 source 命令是什么？















