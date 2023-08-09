# docker swarm

### 1.创建集群

任意节点通过 `docker swarm init` 命令创建一个新集群。当前节点作为 manager 节点加入到这个集群。

通常第一个加入集群的管理节点为 `Leader` 后加入的管理节点为 `Reachable`。当 Leader 节点挂掉后，会从 Reachable 节点中选举出新的 Leader。

```shell
sudo docker swarm init --advertise-addr 10.30.36.224
```
结果如下：

```
[mode@s01-PC ~]$ docker swarm init --advertise-addr 10.30.36.224
Swarm initialized: current node (cxmmclaekzeypd7078j9h1sqk) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-20hltwyd5g6c8evsgyjzgxnf5yxlkwhy391rdl98x386bzvzxo-4cxxvjaj6bz38uk3p6djzdoty 10.30.36.224:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### 2.加入集群

Docker 中内置的集群模式自带了公钥基础设施（PKI）系统，使得安全部署容器变得简单。集群中的节点使用传输层安全协议（TLS）对集群中其他节点的通信进行身份验证、授权和加密。

默认情况下，通过 `docker swarm init` 命令创建一个新的 swarm 集群时，manager 节点会生成新的根证书颁发机构（CA）和密钥对，用于保护与加入群集的其他节点之间的通信安全。

manager 节点会生成两个令牌，供其他节点加入集群时使用：一个 worker 令牌，一个 manager 令牌。每个令牌都包括根 CA 证书的摘要和随机生成的密钥。当节点加入群集时，加入的节点使用摘要来验证来自远程管理节点的根 CA 证书。远程管理节点使用密钥来确保加入的节点是批准的节点。

#### Manager

若要向该集群添加 manager 节点，管理节点先运行 `docker swarm join-token manager` 命令查看管理节点的令牌信息。

```
[mode@s01-PC ~]$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-20hltwyd5g6c8evsgyjzgxnf5yxlkwhy391rdl98x386bzvzxo-4r8zd43yqhjinp9by2t5d3f03 10.30.36.224:2377
```

然后在其他节点上运行 `docker swarm join` 并携带令牌参数以 manager 身份加入 swarm 集群。

```
[mode@s02-PC ~]$ docker swarm join --token SWMTKN-1-20hltwyd5g6c8evsgyjzgxnf5yxlkwhy391rdl98x386bzvzxo-4r8zd43yqhjinp9by2t5d3f03 10.30.36.224:2377
This node joined a swarm as a manager.
```

#### Worker 

若要向该集群添加 worker 节点，管理节点先运行 `docker swarm join-token worker` 命令查看管理节点的令牌信息。

```
[mode@s01-PC ~]$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-20hltwyd5g6c8evsgyjzgxnf5yxlkwhy391rdl98x386bzvzxo-4cxxvjaj6bz38uk3p6djzdoty 10.30.36.224:2377
```

然后在其他节点上运行 `docker swarm join` 并携带令牌参数以 worker 身份加入 swarm 集群。

```
[mode@s03-PC ~]$ docker swarm join --token SWMTKN-1-20hltwyd5g6c8evsgyjzgxnf5yxlkwhy391rdl98x386bzvzxo-4cxxvjaj6bz38uk3p6djzdoty 10.30.36.224:2377
This node joined a swarm as a worker.
```

### 3.查看集群信息

在任意 manager 节点执行 `docker info` 可查看当前集群信息。

```
```













==========================================================================================



### 给节点升职

### 脱离集群

```
sudo docker swarm leave --force
```


### 重建集群（利用当前状态数据）

```
docker swarm init --force-new-cluster
```
