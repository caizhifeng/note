## 1. 配置hostname达成互访

```sh
ssh-keygen
ssh-copy-id -i /root/.ssh/id_rsa.pub root@<其他节点>
```
---
## 2. 关闭swap，注释swap分区

```sh
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
swapoff -a
sed -i.bak '/swap/s/^/#/' /etc/fstab
```
---
## 3. 系统配置

```sh
# 配置内核参数，将桥接的IPv4流量传递到iptables的链
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置必需的 sysctl 参数，这些参数在重新启动后仍然存在。
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

# 安装IPVS
yum install -y ipvsadm ipset
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4

# 关闭 NetworkManager
systemctl stop NetworkManager
systemctl disable NetworkManager
```

---
## 4. 安装containerd
```sh
# 安装所需包
sudo yum install -y yum-utils bash-completion net-tools gcc device-mapper-persistent-data lvm2 vim lsof

#### 新增 Docker 仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 国内
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装 containerd
subscription-manager repos --enable=rhel-7-server-extras-rpms
sudo yum update -y && sudo yum install -y containerd.io

# 配置 containerd
sudo mkdir -p /etc/containerd
sudo containerd config default > /etc/containerd/config.toml

# 替换配置文件
sed -i '/containerd.runtimes.runc.options/a\ \ \ \ \ \ \ \ \ \ \ \ SystemdCgroup = true' /etc/containerd/config.toml
sed -i "s#k8s.gcr.io#registry.cn-hangzhou.aliyuncs.com/google_containers#g" /etc/containerd/config.toml
sed -i "s#https://registry-1.docker.io#https://registry.cn-hangzhou.aliyuncs.com#g" /etc/containerd/config.toml

# 启动containerd
systemctl start containerd && systemctl enable containerd
```
---
## 5. 安装kubectl、kubelet、kubeadm
```sh
# 添加阿里kubernetes源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装k8s组件
yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9

# 设置运行时
crictl config runtime-endpoint /run/containerd/containerd.sock
crictl config image-endpoint /run/containerd/containerd.sock
systemctl daemon-reload && systemctl enable kubelet;systemctl start kubelet

# kubelet不断重启是正常现象，使用crictl查看k8s容器。
```
---
## 6. 初始化k8s集群

#### 直接启动集群（cgroup检测不正确时有bug）
```sh
kubeadm init --kubernetes-version=1.20.9 --apiserver-advertise-address=192.168.50.11 --image-repository registry.aliyuncs.com/google_containers --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
```

#### 修改配置启动集群
```sh
kubeadm config print init-defaults > kubeadm.yaml

# 修改nodeRegistration:
  criSocket: /run/containerd/containerd.sock
  name: node1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master

# 修改IP
advertiseAddress: 192.168.50.11

# networking下添加
podSubnet: 172.16.0.0/16

# 最后添加
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd

# 初始化
kubeadm init --config=kubeadm.yaml

# 集群初始化成功后返回信息，记录生成的最后部分内容，此内容需要在其它节点加入Kubernetes集群时执行。

# 根据提示创建kubectl配置文件
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 执行下面命令，使kubectl可以自动补充
source <(kubectl completion bash)

# 查看节点，pod
kubectl get node
kubectl get pod --all-namespaces

# node节点为NotReady，因为coredns pod没有启动，缺少网络pod
```

#### 密钥过期重新生成方法备用
```sh
kubeadm token create [token]
# 获取CA证书HASH
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
# 加入集群
kubeadm join 192.168.50.11:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:bc57d7b1b9e2185dec27965fdfd8b0022b6908e4e9e46f3838f9fa0cb56af769
```

---
## 7. 安装calico网络
```sh
kubectl apply -f
wget https://docs.projectcalico.org/manifests/calico.yaml

# 编辑calico.yaml找到name: IP，在它上面写入：
            - name: IP_AUTODETECTION_METHOD
              value: "interface=eth0"
# 查看pod和node
kubectl get pod --all-namespaces
kubectl get node

# 安装Calicoctl
curl -o calicoctl -O -L "https://github.com/projectcalico/calicoctl/releases/download/v3.20.0/calicoctl" && chmod +x calicoctl
```
---
## 8. 部署K8s WEB Dashboard
```sh
# 下载官网新版
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml

# 修改dashboard service，Deployment的args添加
- --token-ttl=43200

kubectl apply -f recommended.yaml

# 创建用户以供访问
# 创建用户
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# 绑定角色
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# 获取Token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

# 命令行方式
admin_account="admin-user"
admin_namespace="kubernetes-dashboard"
kubectl create serviceaccount ${admin_account} -n ${admin_namespace}
kubectl create clusterrolebinding ${admin_account} --clusterrole=cluster-admin --serviceaccount=${admin_namespace}:${admin_account}
kubectl -n ${admin_namespace} describe secrets $(kubectl -n ${admin_namespace} get secret | grep ${admin_account} | awk '{print $1}')
```
---
## 9. 定时备份ETCD防止集群挂掉
```sh
cat etcd_cronjob.yaml
---
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: etcd-backup
spec:
 # 30分钟执行一次备份
 schedule: "*/10 * * * *"
 jobTemplate:
  spec:
    template:
      metadata:
       labels:
        app: etcd-disaster-recovery
      spec:
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                  nodeSelectorTerms:
                  - matchExpressions:
                    - key: kubernetes.io/role
                      operator: In
                      values:
                      - master
        containers:
        - name: etcd
          image: etcd:backup
          command:
          - sh
          - -c
          - "export ETCDCTL_API=3; \
             # 备份前执行下清理的操作，最多保留6个快照
             sh -x /usr/bin/delete_image_reserver_5.sh; \
             etcdctl --endpoints $ENDPOINT snapshot save /snapshot/$(date +%Y%m%d_%H%M%S)_snapshot.db; \
             echo etcd backup sucess"
          env:
          - name: ENDPOINT
            value: "127.0.0.1:2379"
          volumeMounts:
            - mountPath: "/snapshot"
              name: snapshot
              subPath: etcd-snapshot
            - mountPath: /etc/localtime
              name: lt-config
            - mountPath: /etc/timezone
              name: tz-config
        restartPolicy: OnFailure
        volumes:
          - name: snapshot
            hostPath:
              path: /var
          - name: lt-config
            hostPath:
              path: /etc/localtime
          - name: tz-config
            hostPath:
              path: /etc/timezone
        hostNetwork: true
# 设置master节点可调度
kubectl uncordon ${masterip}
# 创建etcd定时备份的job
kubectl apply -f etcd_cronjob.yaml
# 查看备份的快照
ls /var/etcd-snapshot/ -alh
```
---
## 10. 部署Ingress从外部访问
