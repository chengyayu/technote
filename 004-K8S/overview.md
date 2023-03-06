# 概览

## 水平伸缩、滚动更新

> Deployment 控制 ReplicaSet（应用版本），ReplicaSet 控制 Pod（应用副本数）。

```
Deployment
    |
    |-ReplicaSet(V1)
    |       |
    |       |-Pod
    |       |-Pod
    |       
    |-ReplicaSet(V2)
            |
            |-Pod
            |-Pod
```

- ReplicaSet 通过“控制器模式”，保证系统中 Pod 的个数永远等于指定的个数。
- Deployment 通过“控制器模式”，来操作 ReplicaSet 的个数和属性，进而实现“水平扩展 / 收缩”和“滚动更新”这两个编排动作。

```
为什么 Deployment 只允许容器的 restartPolicy=Always ？

只有在容器能保证自己始终是 Running 状态的前提下，ReplicaSet 调整 Pod 的个数才有意义。

restartPolicy 表示容器故障后 Kubelet 对容器的重启策略，共有三种：Always、OnFailure 和 Never。

kubectl get pod {pod-name} -o yaml | grep restartPolicy
```

### 水平伸缩

Deployment Controller 只需要修改它所控制的 ReplicaSet 的 Pod 副本个数就可以实现水平伸缩。比如，把这个值从 3 改成 4，那么 Deployment 所对应的 ReplicaSet，就会根据修改后的值自动创建一个新的 Pod。这就是“水平扩展”了；“水平收缩”则反之。 

### 滚动更新

对 Deployment 配置信息中应用版本进行升级后，会创建新的 rs ，将一个集群中正在运行的多个 Pod 版本，交替地逐一升级，这个过程就是“滚动更新”。

```
$ kubectl describe deployment nginx-deployment
...
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
...
  Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 1
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 2
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 2
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 1
  Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1764197365 to 3
  Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-3167673210 to 0
```

--- 

Deployment 实际上是一个两层控制器。首先，它通过 ReplicaSet 的个数来描述应用的版本；然后，它再通过 ReplicaSet 的属性（比如 replicas 的值），来保证 Pod 的副本数量。

Kubernetes 项目对 Deployment 的设计，实际上是代替我们完成了对“应用”的抽象，使得我们可以使用这个 Deployment 对象来描述应用，使用 `kubectl rollout` 命令控制应用的版本。