#### 1. 主机名
要在/etc/hosts里做好配置，能通过主机名互相ping通。

---

#### 2. 系统配置
1. 如系统文件句柄数等。
2. 关闭selinux，setenforce 0，修改/etc/selinux/config，改为disabled。

---

#### 3. RPM安装
安装erlang，然后安装rabbitmq。

---

#### 4. 复制rabbitmq节点.erlang.cookie
确保集群机器该文件均相同，路径为~/下和/var/lib/rabbitmq下，如有后者以后者为主，设置权限。
    chmod 400 .erlang.cookie

---

#### 5. 在新rabbitmq上执行命令，加入集群。
```sh
rabbitmq-server -detached
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster (--ram) rabbit@rabbitmq主机名
rabbitmqctl start_app
```

---

#### 6. 访问rabbitmq web管理界面查看节点是否加入。
