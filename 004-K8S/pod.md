# Pod

- 容器的本质是进程，K8S 本质是操作系统。
- Pod 是 k8s 项目的原子调度单位。
- Pod 是一个逻辑概念，可以理解成“虚拟机”。它管理者一组“超亲密关系”容器对于资源的使用。
- Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。
- Pod 里的容器是对等关系，不是拓扑关系。资源的共享通过 Infra 容器关联在一起。

## Pod 对象的生命周期

Pod 生命周期的变化，主要体现在 Pod API 对象的 Status 部分，这是它除了 Metadata 和 Spec 之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：
- Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。
- Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
- Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
- Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
- Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

每个状态字段还包含一个细分状态值数组（coditions）

- PodScheduled
- Ready
- Initialized
- Unschedulable

Pod 当前的 Status 是 Pending，对应的 Condition 是 Unschedulable，这就意味着它的调度出现了问题。

而其中，Ready 这个细分状态非常值得我们关注：它意味着 Pod 不仅已经正常启动（Running 状态），而且已经可以对外提供服务了。

## 常用字段

1、 凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。

- NodeSelector：是一个供用户将 Pod 与 Node 进行绑定的字段。
    ```yaml
    # 这个 Pod 永远只能运行在携带了“disktype: ssd”标签（Label）的节点上
    apiVersion: v1
    kind: Pod
    ...
    spec:
        nodeSelector:
        disktype: ssd
    ...
    ```
- NodeName：一旦 Pod 的这个字段被赋值，Kubernetes 项目就会被认为这个 Pod 已经经过了调度，调度的结果就是赋值的节点名字。
- HostAliases：定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容。
    ```go
    apiVersion: v1
    kind: Pod
    ...
    spec:
    hostAliases:
    - ip: "10.1.2.3"
        hostnames:
        - "foo.remote"
        - "bar.remote"
    ...
    ```
    Pod 启动后，/etc/hosts 文件内容如下：
    ```shell
    cat /etc/hosts
    # Kubernetes-managed hosts file.
    127.0.0.1 localhost
    ...
    10.244.135.10 hostaliases-pod
    10.1.2.3 foo.remote # 看这里
    10.1.2.3 bar.remote # 看这里
    ```

2、凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true # 这个 Pod 里的容器要共享 PID Namespace
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true # 看这里
    tty: true # 和这里，在 Pod 的 YAML 文件里声明开启它们俩，其实等同于设置了 docker run 里的 -it（-i 即 stdin，-t 即 tty）参数。
```

3、凡是 Pod 中的容器要共享宿主机的 Namespace，也一定是 Pod 级别的定义。
