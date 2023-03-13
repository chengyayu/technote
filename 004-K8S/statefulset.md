# StatefulSet

## 有状态应用

实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态应用”（Stateful Application）。

具体状态可分两类：

1. **拓扑状态**。应用的多个实例之间不是完全对等的关系。这些应用实例，**必须按照某些顺序启动**，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的**网络标识一样**，这样原先的访问者才能使用同样的方法，访问到这个新 Pod。
2. **存储状态**。**应用的多个实例分别绑定了不同的存储数据**。对于这些应用实例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的多个存储实例。 

StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。

## 拓扑状态

## 存储状态