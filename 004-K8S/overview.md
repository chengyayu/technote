# 概览

## 作业副本、水平伸缩、版本控制

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

Deployment 实际上是一个两层控制器。首先，它通过 ReplicaSet 的个数来描述应用的版本；然后，它再通过 ReplicaSet 的属性（比如 replicas 的值），来保证 Pod 的副本数量。

Kubernetes 项目对 Deployment 的设计，实际上是代替我们完成了对“应用”的抽象，使得我们可以使用这个 Deployment 对象来描述应用，使用 `kubectl rollout` 命令控制应用的版本。