以RabbitMQ集群代理配置为例，替换IP为节点IP

maxconn可以调整为60000

```c
listen rabbitmq_admin 
    bind 0.0.0.0:15672
    server node1 IP1:15672
    server node2 IP2:15672
    server node3 IP3:15672

listen rabbitmq_cluster 
    bind 0.0.0.0:5672
    option tcplog
    mode tcp
    timeout client 3h
    timeout server 3h
    option clitcpka
    balance roundrobin
    server node1 IP1:5672 check inter 5s rise 2 fall 3
    server node2 IP2:5672 check inter 5s rise 2 fall 3
    server node3 IP2:5672 check inter 5s rise 2 fall 3
```