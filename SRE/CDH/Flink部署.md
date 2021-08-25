## 0. Standalone

export FLINK_HOME，无密访问各机器。

---
## 1. Flink on Yarn/HA

部署Hadoop。

---
## 2. Flink配置
```yaml
flink-conf.yaml

jobmanager.rpc.address: master1
jobmanager.rpc.port: 6123
jobmanager.heap.size: 1024m
taskmanager.heap.size: 1024m
taskmanager.rpc.port: 51000-52000
taskmanager.numberOfTaskSlots: 10
parallelism.default: 3

restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 9999
restart-strategy.fixed-delay.delay: 60s

high-availability: zookeeper
high-availability.storageDir: hdfs:///flink/ha/
high-availability.zookeeper.quorum: zk:2181
state.backend: filesystem
state.checkpoints.dir: hdfs:///flink-checkpoints
state.savepoints.dir: hdfs:///flink-savepoints
jobmanager.execution.failover-strategy: region
rest.port: 8081
rest.address: 0.0.0.0

web.submit.enable: false
io.tmp.dirs: /data/flink/tmp
taskmanager.memory.network.fraction: 0.1
taskmanager.memory.network.min: 64mb
taskmanager.memory.network.max: 1gb
jobmanager.archive.fs.dir: hdfs:///completed-jobs/
historyserver.web.address: 0.0.0.0
historyserver.web.port: 8082
historyserver.archive.fs.dir: hdfs:///completed-jobs/
historyserver.archive.fs.refresh-interval: 10000
```

```c
zoo.cfg

tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181

server.0=zk:2888:3888
```

```c
masters
hostname:8081
```
slaves

---
## 3.Flink启动
```sh
# 检查
yarn-session.sh -q

# 启动
./bin/start-cluster.sh
```