# DaemonSet

## 简介

DaemonSet 在 Kubernetes 集群里，运行一个 Daemon Pod。

Daemon Pod 的特点：
1. 这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；
2. 每个节点上只有一个这样的 Pod 实例；
3. 当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

Daemon Pod 的应用：
1. 各种**网络插件的 Agent 组件**，都必须运行在每一个节点上，用来处理这个节点上的容器网络；
2. 各种**存储插件的 Agent 组件**，也必须运行在每一个节点上，用来在这个节点上挂载远程存储目录，操作容器的 Volume 目录；
3. 各种**监控组件**和**日志组件**，也必须运行在每一个节点上，负责这个节点上的监控信息和日志搜集。

## 如何保证每个 Node 上有且只有一个被管理的 Pod ?

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch # 标签名
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

DaemonSet Controller，首先从 Etcd 里获取所有的 Node 列表，然后遍历所有的 Node。这时，它就可以很容易地去检查，当前这个 Node 上是不是有一个携带了 `name=fluentd-elasticsearch` 标签的 Pod 在运行。

而检查的结果，可能有这么三种情况：
- 没有这种 Pod，那么就意味着要在这个 Node 上创建这样一个 Pod；
- 有这种 Pod，但是数量大于 1，那就说明要把多余的 Pod 从这个 Node 上删除掉；
- 正好只有一个这种 Pod，那说明这个节点是正常的。

## 参考资料

- [容器化守护进程的意义：DaemonSet](https://time.geekbang.org/column/article/41366)
