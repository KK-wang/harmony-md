# learn-java-2

## pvc、pv 与 pod 申领的 volume 分别是什么？

pvc 和 pod 申领的 volume 都是抽象的概念。

只有 pv 才是实际的磁盘。

对于如下 yaml 文件：

```yaml
kind: Pod
metadata:
  name: my-pod
spec:
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: my-volume
          mountPath: /data
```

其中的 volumes 配置表示，pod 将会申领一个 volume，规格按照名称为 my-pvc 的 pvc 配置文件来，其名称为 my-volume。这将会让该 pod 通过拥有 pv 来实际占据一部分磁盘空间。 

这块 volume 将会供 pod 中的 container 使用。

## 主机网络和端口映射？

