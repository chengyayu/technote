# 常用指令

1. 查询 {namespace} 下的所有资源列表

```shell
kubectl api-resources -o name --verbs=list --namespaced | xargs -n 1 kubectl get --show-kind --ignore-not-found -n {namespace}
```
