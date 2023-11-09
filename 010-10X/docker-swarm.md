### 0. 根据{服务名}筛选出运行中的容器信息

```
docker service ps {服务名} --no-trunc --filter="desired-state=running" 
```

### 1. 在 etcd 节点执行查询 key 为 'config' 的值

```
docker exec -it $(
  docker inspect $(
    docker service ps intranet-etcd_node01 --no-trunc \
    --filter="desired-state=running" \
    --format "{{.ID}}"
  ) --format "{{.Status.ContainerStatus.ContainerID}}"
) etcdctl get config
```

### 2. 在 pg 节点执行查询指令

```shell
docker exec -it $(
  docker inspect $(
    docker service ps intranet-pg_postgres --no-trunc \
    --filter="desired-state=running" \
    --format "{{.ID}}"
  ) --format "{{.Status.ContainerStatus.ContainerID}}"
) psql -h 127.0.0.1 -U postgres -d {库名} \
-c "SELECT * FROM app_locales WHERE id = 11450;"
```

