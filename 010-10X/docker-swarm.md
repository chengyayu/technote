### 0. 根据{服务名}筛选出运行中的容器信息

```
docker service ps {服务名} --no-trunc --filter="desired-state=running"
```

### 1. 在etcd节点内执行指令

```
docker exec -it $(docker inspect $(docker service ps intranet-etcd_node01 --no-trunc --format "{{.ID}}") --format "{{.Status.ContainerStatus.ContainerID}}") etcdctl get config
```

### 2. 进入 pg 执行指令
