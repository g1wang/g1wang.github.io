## ETCD

### ETCD 简介

A highly-available key value store for shared configuration and service discovery

受到 [Apache ZooKeeper](https://zookeeper.apache.org/) 项目和 [doozer](https://github.com/ha/doozerd) 项目的启发
-   简单：具有定义良好、面向用户的 `API` ([gRPC](https://github.com/grpc/grpc))
-   安全：支持 `HTTPS` 方式的访问
-   快速：支持并发 `10 k/s` 的写操作
-   可靠：支持分布式结构，基于 `Raft` 的一致性算法

Apache ZooKeeper 是一套知名的分布式系统中进行同步和一致性管理的工具。
doozer 是一个一致性分布式数据库。
[Raft_](https://raft.github.io/) _是一套通过选举主节点来实现分布式系统一致性的算法

#### 安装
```
 docker run \
-p 2379:2379 \
-p 2380:2380 \
--mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
--name etcd-gcr-v3.4.0 \
quay.io/coreos/etcd:v3.4.0 \
/usr/local/bin/etcd \
--name s1 \
--data-dir /etcd-data \
--listen-client-urls http://0.0.0.0:2379 \
--advertise-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380 \
--initial-advertise-peer-urls http://0.0.0.0:2380 \
--initial-cluster s1=http://0.0.0.0:2380 \
--initial-cluster-token tkn \
--initial-cluster-state new \
--log-level info \
--logger zap \
--log-outputs stderr
```
