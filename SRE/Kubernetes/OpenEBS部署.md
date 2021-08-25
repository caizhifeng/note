#### 官方文档:
https://docs.openebs.io/docs/next/installation.html#installation-through-kubectl

#### 部署yaml:
https://openebs.github.io/charts/openebs-operator.yaml


1. 修改OPENEBS_IO_LOCALPV_HOSTPATH_DIR以改变存储位置。
2. 部署后，修改hostpath
```sh
kubectl edit sc openebs-hostpath
value: /data/openebs/local
```
3. 配置为默认存储类
```sh
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
