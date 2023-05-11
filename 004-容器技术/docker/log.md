## 查看 docker 服务日志

### 查看所有日志

```shell
sudo journalctl -u docker.service
```

### 查看前十的错误日志

```shell
sudo journalctl -u docker.service --since yesterday --until now | grep -i error | head -n 10
```

### 查看实时日志

```shell
sudo journalctl -u docker.serivce -f
```

### 按日期和时间范围查看日志

```shell
sudo journalctl -u docker.service --since "2023-05-11 09:00:00" --until "2023-05-11 12:00:00"
```
