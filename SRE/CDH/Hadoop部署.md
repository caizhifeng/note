#### 1. 准备工作

1.1. Master到其他机器的无密
1.2. Hostname访问(localhost不包含主机名)
1.3. 
```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8*-openjdk*
export HADOOP_HOME(bin sbin)
export HADOOP_CLASSPATH=`hadoop classpath`
```
1.4. mkdir -p /data/flink/ /data/hadoop

---
#### 2. Hadoop配置

```xml
core-site.xml

<configuration>
<property>
<name>hadoop.tmp.dir</name>
<value>/data/hadoop/tmp</value>
</property>

<property>
<name>fs.defaultFS</name>
<value>hdfs://hostname:port</value>
</property>

<property>
<name>dfs.journalnode.edits.dir</name>
<value>/data/hadoop/journal</value>
</property>

<property>
<name>ha.zookeeper.quorum</name>
<value>hostname:2181,</value>
</property>
</configuration>
```

```xml
hdfs-site.xml

<configuration>
<property>
<name>dfs.replication</name>
<value>1</value>
</property>

<property>
<name>dfs.namenode.name.dir</name>
<value>/data/hadoop/hdfs/name</value>
</property>

<property>
<name>dfs.datanode.data.dir</name>
<value>/data/hadoop/hdfs/data</value>
</property>

<property>
<name>dfs.permissions</name>
<value>false</value>
</property>

<property>
<name>dfs.nameservices</name>
<value>ns</value>
</property>

<property>
<name>dfs.ha.namenodes.ns</name>
<value>nn1,nn2</value>
</property>

<property>
<name>dfs.namenode.rpc-address.ns.nn1</name>
<value>rexel-ids001:9000</value>
</property>

<property>
<name>dfs.namenode.rpc-address.ns.nn2</name>
<value>rexel-ids002:9000</value>
</property>

<property>
<name>dfs.namenode.http-address.ns.nn1</name>
<value>rexel-ids001:50070</value>
</property>

<property>
<name>dfs.namenode.http-address.ns.nn2</name>
<value>rexel-ids002:50070</value>
</property>

<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://rexel-ids001:8485;rexel-ids002:8485;rexel-ids003:8485/ns</value>
</property>

<property>
<name>dfs.ha.automatic-failover.enabled.ns</name>
<value>true</value>
</property>

<property>
<name>dfs.client.failover.proxy.provider.ns</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>

<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>

<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>~/.ssh/id_rsa</value>
</property>
</configuration>
```

```xml
mapred-site.xml

<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

<property>
<name>mapreduce.reduce.memory.mb</name>
<value>54</value>
</property>

<property>
<name>mapreduce.map.memory.mb</name>
<value>128</value>
</property>
</configuration>
```

```xml
yarn-site.xml

<configuration>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>

<property>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>

<property>
<name>yarn.resourcemanager.cluster-id</name>
<value>ns</value>
</property>

<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>

<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>hostname1</value>
</property> 

<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>hostname2</value>
</property>

<property>
<name>yarn.resourcemanager.webapp.address.rm1</name>
<value>hostname1:8088</value>
</property>

<property>
<name>yarn.resourcemanager.webapp.address.rm2</name>
<value>hostname2:8088</value>
</property>

<property>
<name>yarn.resourcemanager.zk-address</name>
<value>zk:2181,</value>
</property>

<property>
<name>yarn.resourcemanager.recovery.enabled</name>
<value>true</value>
</property>

<property>
<name>yarn.resourcemanager.store.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>

<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>4096</value>
</property>

<property>
<name>yarn.scheduler.minimum-allocation-mb</name>
<value>1024</value>
</property>

<property>
<name>yarn.scheduler.maximum-allocation-mb</name>
<value>4096</value>
</property>

<property>
<name>yarn.resourcemanager.am.max-attempts</name>
<value>4</value>
</property></configuration>
```

masters
slaves
 
#### 3. Hadoop启动

3.1. 在master1上
```sh
hdfs zkfc -formatZK
```
3.2. 在所有节点分别启动：
```sh
hadoop-daemon.sh start journalnode
```
3.3. 在master1上
```sh
hdfs namenode -format
hadoop-daemon.sh start namenode
```
3.4. 在master2上：
```sh
hdfs namenode -bootstrapStandby
hadoop-daemon.sh start namenode
```
3.5. 在master1/2上：
```sh
hadoop-daemon.sh start zkfc
```
3.6. 在所有节点分别启动：
```sh
hadoop-daemon.sh start datanode
```
3.7. 在master1/2上：
```sh
yarn-daemon.sh start resourcemanager
```
3.8. 在所有节点分别启动：
```sh
yarn-daemon.sh start nodemanager
```
3.9. 在master1上启动：
```sh
mr-jobhistory-daemon.sh start historyserver
```