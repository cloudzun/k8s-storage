# 安装Rook

本次实验的环境：

- 服务器数量：三台
- 服务器OS：Ubuntu 20.04
- 服务器硬件配置：4C8G双硬盘
- K8S版本：1.23.0



因为三台机器都要承担工作负载，因此需要去掉master上的污点

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```



检查服务器磁盘配置

```bash
lsblk -f
```



```bash
root@node1:~# lsblk -f
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
fd0
sda
└─sda1 ext4         bbd3ea56-da3b-4e1a-b14e-159e41299ea3    112G     6% /
sdb
sr0
```

如输出所示,sdb为一块未格式化的裸盘,用户后续ceph群集的搭建



下载Rook Repo

```bash
git clone --single-branch --branch v1.6.3 https://github.com/rook/rook.git
```



```bash
cd rook/cluster/examples/kubernetes/ceph
```



修改 operator.yaml

```bash
nano operator.yaml
```



启用自动发现

```bash
  # This daemon does not need to run if you are only going to create your OSDs based on StorageClassDeviceSets with PVCs.
  ROOK_ENABLE_DISCOVERY_DAEMON: "true" #修改为true
```



使用国内镜像

```bash
  # these images to the desired release of the CSI driver.
  # ROOK_CSI_CEPH_IMAGE: "quay.io/cephcsi/cephcsi:v3.3.1"
  # ROOK_CSI_REGISTRAR_IMAGE: "k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.0.1"
  # ROOK_CSI_RESIZER_IMAGE: "k8s.gcr.io/sig-storage/csi-resizer:v1.0.1"
  # ROOK_CSI_PROVISIONER_IMAGE: "k8s.gcr.io/sig-storage/csi-provisioner:v2.0.4"
  # ROOK_CSI_SNAPSHOTTER_IMAGE: "k8s.gcr.io/sig-storage/csi-snapshotter:v4.0.0"
  # ROOK_CSI_ATTACHER_IMAGE: "k8s.gcr.io/sig-storage/csi-attacher:v3.0.2"
```

替换为

```bash
ROOK_CSI_REGISTRAR_IMAGE: "registry.cn-beijing.aliyuncs.com/dotbalo/csi-node-driver-registrar:v2.0.1"
ROOK_CSI_RESIZER_IMAGE: "registry.cn-beijing.aliyuncs.com/dotbalo/csi-resizer:v1.0.1"
ROOK_CSI_PROVISIONER_IMAGE: "registry.cn-beijing.aliyuncs.com/dotbalo/csi-provisioner:v2.0.4"
ROOK_CSI_SNAPSHOTTER_IMAGE: "registry.cn-beijing.aliyuncs.com/dotbalo/csi-snapshotter:v4.0.0"
ROOK_CSI_ATTACHER_IMAGE: "registry.cn-beijing.aliyuncs.com/dotbalo/csi-attacher:v3.0.2"
```



部署rook

```bash
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```



查看创建的crd

```
kubectl get crd | grep rook
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get crd | grep rook
cephblockpools.ceph.rook.io                                 2022-12-06T01:22:46Z
cephclients.ceph.rook.io                                    2022-12-06T01:22:46Z
cephclusters.ceph.rook.io                                   2022-12-06T01:22:46Z
cephfilesystemmirrors.ceph.rook.io                          2022-12-06T01:22:46Z
cephfilesystems.ceph.rook.io                                2022-12-06T01:22:46Z
cephnfses.ceph.rook.io                                      2022-12-06T01:22:46Z
cephobjectrealms.ceph.rook.io                               2022-12-06T01:22:46Z
cephobjectstores.ceph.rook.io                               2022-12-06T01:22:46Z
cephobjectstoreusers.ceph.rook.io                           2022-12-06T01:22:46Z
cephobjectzonegroups.ceph.rook.io                           2022-12-06T01:22:46Z
cephobjectzones.ceph.rook.io                                2022-12-06T01:22:46Z
cephrbdmirrors.ceph.rook.io                                 2022-12-06T01:22:46Z
volumes.rook.io                                             2022-12-06T01:22:46Z
```



查看ceph群集信息

```bash
kubectl get cephcluster -n rook-ceph
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get cephcluster -n rook-ceph
No resources found in rook-ceph namespace.
```





查看pod

```bash
kubectl -n rook-ceph get pod
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl -n rook-ceph get pod
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-operator-7958854bd7-wbl6x   1/1     Running   0          7m31s
rook-discover-j94xk                   1/1     Running   0          6m49s
rook-discover-p5czf                   1/1     Running   0          6m49s
rook-discover-qvh52                   1/1     Running   0          6m49s
```



创建ceph群集

```bash
kubectl create -f cluster.yaml
```



查看创建过程

```
kubectl get cephcluster -n rook-ceph
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get cephcluster -n rook-ceph
NAME        DATADIRHOSTPATH   MONCOUNT   AGE   PHASE         MESSAGE                  HEALTH   EXTERNAL
rook-ceph   /var/lib/rook     3          54s   Progressing   Detecting Ceph version
```



查看operator日志

```bash
 kubectl logs rook-ceph-operator-7958854bd7-wbl6x -n rook-ceph
```



```bash
2022-12-06 01:35:54.520275 I | op-mgr: successfully set ceph dashboard creds
2022-12-06 01:35:56.130913 I | op-osd: updating OSD 0 on node "node2"
2022-12-06 01:35:56.166867 I | op-osd: OSD orchestration status for node node2 is "completed"
2022-12-06 01:35:56.171823 I | op-osd: OSD orchestration status for node node3 is "orchestrating"
2022-12-06 01:35:58.069999 I | op-osd: updating OSD 1 on node "node3"
2022-12-06 01:35:58.099479 I | op-osd: OSD orchestration status for node node3 is "completed"
2022-12-06 01:35:58.339327 I | op-mgr: dashboard config has changed. restarting the dashboard module
2022-12-06 01:35:58.339436 I | op-mgr: restarting the mgr module
2022-12-06 01:35:59.347193 I | op-osd: updating OSD 2 on node "node1"
2022-12-06 01:36:00.266395 I | cephclient: successfully disallowed pre-octopus osds and enabled all new octopus-only functionality
2022-12-06 01:36:00.266434 I | op-osd: finished running OSDs in namespace "rook-ceph"
2022-12-06 01:36:00.266442 I | ceph-cluster-controller: done reconciling ceph cluster in namespace "rook-ceph"
2022-12-06 01:36:00.539515 I | op-mgr: successful modules: dashboard
```



确认群集安装状态

```
kubectl get cephcluster -n rook-ceph
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get cephcluster -n rook-ceph
NAME        DATADIRHOSTPATH   MONCOUNT   AGE   PHASE   MESSAGE                        HEALTH        EXTERNAL
rook-ceph   /var/lib/rook     3          13m   Ready   Cluster created successfully   HEALTH_WARN

```





查看pod

```
kubectl get pod -n rook-ceph
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get pod -n rook-ceph
NAME                                              READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-77g4f                            3/3     Running     0          4m55s
csi-cephfsplugin-cn9cc                            3/3     Running     0          4m55s
csi-cephfsplugin-cxd82                            3/3     Running     0          4m55s
csi-cephfsplugin-provisioner-7594b68bcb-2t575     6/6     Running     0          4m55s
csi-cephfsplugin-provisioner-7594b68bcb-wwn2j     6/6     Running     0          4m55s
csi-rbdplugin-2v2fc                               3/3     Running     0          4m56s
csi-rbdplugin-nntm2                               3/3     Running     0          4m56s
csi-rbdplugin-provisioner-757bdc7649-4qrtz        6/6     Running     0          4m56s
csi-rbdplugin-provisioner-757bdc7649-tlwcc        6/6     Running     0          4m56s
csi-rbdplugin-px7rf                               3/3     Running     0          4m56s
rook-ceph-crashcollector-node1-59767675f6-8dv7p   1/1     Running     0          3m9s
rook-ceph-crashcollector-node2-5f9466c4df-xl2g9   1/1     Running     0          3m20s
rook-ceph-crashcollector-node3-6c68cb765d-mjnp2   1/1     Running     0          3m20s
rook-ceph-mgr-a-7bf8f98c4b-4hsbm                  1/1     Running     0          3m23s
rook-ceph-mon-a-74558fc57d-wstvp                  1/1     Running     0          4m7s
rook-ceph-mon-b-84fc75c787-zwhcl                  1/1     Running     0          4m
rook-ceph-mon-c-6d46b45cb8-fbj4k                  1/1     Running     0          3m50s
rook-ceph-operator-7958854bd7-wbl6x               1/1     Running     0          15m
rook-ceph-osd-0-5b4699757c-nfprg                  1/1     Running     0          3m10s
rook-ceph-osd-1-86549d7674-8pw2g                  1/1     Running     0          3m10s
rook-ceph-osd-2-57ccd6fdb8-gbkdg                  1/1     Running     0          3m9s
rook-ceph-osd-prepare-node1-7zn7h                 0/1     Completed   0          2m51s
rook-ceph-osd-prepare-node2-9tpvj                 0/1     Completed   0          2m49s
rook-ceph-osd-prepare-node3-f6hf9                 0/1     Completed   0          2m47s
rook-discover-j94xk                               1/1     Running     0          15m
rook-discover-p5czf                               1/1     Running     0          15m
rook-discover-qvh52                               1/1     Running     0          15m
```



查看服务

```bash
kubectl get svc -n rook-ceph
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get svc -n rook-ceph
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
csi-cephfsplugin-metrics   ClusterIP   10.106.215.152   <none>        8080/TCP,8081/TCP   9m39s
csi-rbdplugin-metrics      ClusterIP   10.97.66.233     <none>        8080/TCP,8081/TCP   9m39s
rook-ceph-mgr              ClusterIP   10.109.198.134   <none>        9283/TCP            8m4s
rook-ceph-mgr-dashboard    ClusterIP   10.101.144.82    <none>        8443/TCP            8m4s
rook-ceph-mon-a            ClusterIP   10.101.130.249   <none>        6789/TCP,3300/TCP   8m52s
rook-ceph-mon-b            ClusterIP   10.106.237.229   <none>        6789/TCP,3300/TCP   8m45s
rook-ceph-mon-c            ClusterIP   10.102.106.129   <none>        6789/TCP,3300/TCP   8m35s
```



安装ceph客户端工具

```bash
kubectl  create -f toolbox.yaml -n rook-ceph
```



查看ceph tools pod

```bash
 kubectl get pod -n rook-ceph | grep tools
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get pod -n rook-ceph | grep tools
rook-ceph-tools-5f6c9f465-8lr4x                   1/1     Running     0          3m45s
```



使用tools pod执行ceph状态检查命令

```bash
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```



```bash
ceph status
ceph osd status
ceph df
rados df
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# ceph status
  cluster:
    id:     c4a85273-6a13-4449-b05d-3ffde2fc6042
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum a,b,c (age 19m)
    mgr: a(active, since 19m)
    osd: 3 osds: 3 up (since 19m), 3 in (since 19m)

  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 378 GiB / 381 GiB avail
    pgs:     1 active+clean
```



```bash
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# ceph osd status
ID  HOST    USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  node2  1026M   125G      0        0       0        0   exists,up
 1  node3  1026M   125G      0        0       0        0   exists,up
 2  node1  1026M   125G      0        0       0        0   exists,up
```



```bash
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# ceph df
--- RAW STORAGE ---
CLASS  SIZE     AVAIL    USED     RAW USED  %RAW USED
hdd    381 GiB  378 GiB  6.6 MiB   3.0 GiB       0.79
TOTAL  381 GiB  378 GiB  6.6 MiB   3.0 GiB       0.79

--- POOLS ---
POOL                   ID  PGS  STORED  OBJECTS  USED  %USED  MAX AVAIL
device_health_metrics   1    1     0 B        0   0 B      0    120 GiB
```



```bash
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# rados df
POOL_NAME              USED  OBJECTS  CLONES  COPIES  MISSING_ON_PRIMARY  UNFOUND  DEGRADED  RD_OPS   RD  WR_OPS   WR  USED COMPR  UNDER COMPR
device_health_metrics   0 B        0       0       0                   0        0         0       0  0 B       0  0 B         0 B          0 B

total_objects    0
total_used       3.0 GiB
total_avail      378 GiB
total_space      381 GiB
```



配置ceph dashboard

```bash
kubectl apply -f dashboard-external-https.yaml
```



查看dashboard相关服务

```bash
kubectl get svc -n rook-ceph | grep rook-ceph-mgr-dashboard
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl get svc -n rook-ceph | grep rook-ceph-mgr-dashboard
rook-ceph-mgr-dashboard                  ClusterIP   10.101.144.82    <none>        8443/TCP            42m
rook-ceph-mgr-dashboard-external-https   NodePort    10.102.110.0     <none>        8443:31215/TCP      113s
```



查看dashboard密码

```bash
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}"|base64 --decode && echo
```



登录到dashboard页面:`https://node1:31215`

![image-20221206102426556](README.assets/image-20221206102426556.png)



消除dashboard中的health_warn

```bash
 kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```



```bash
ceph config set mon auth_allow_insecure_global_id_reclaim false
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# ceph config set mon auth_allow_insecure_global_id_reclaim false
[root@rook-ceph-tools-5f6c9f465-8lr4x /]# ceph -s
  cluster:
    id:     c4a85273-6a13-4449-b05d-3ffde2fc6042
    health: HEALTH_WARN
            mon is allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum a,b,c (age 57m)
    mgr: a(active, since 56m)
    osd: 3 osds: 3 up (since 57m), 3 in (since 57m)

  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 378 GiB / 381 GiB avail
    pgs:     1 active+clean
```



![image-20221206103605891](README.assets/image-20221206103605891.png)



![image-20221206103650211](README.assets/image-20221206103650211.png)





# 实现块存储

## 创建存储池和存储类



查看rbd 存储类定义文件

```yaml
nano /csi/rbd/storageclass.yaml
```



```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool #存储池名称
  namespace: rook-ceph
spec:
  failureDomain: host # 容错域级别
  replicated:
    size: 3 #副本数
    # Disallow setting pool with replica 1, this could lead to data loss without recovery.
    # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
    requireSafeReplicaSize: true
    # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
    # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
    #targetSizeRatio: .5
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block #存储类名称
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # If you want to use erasure coded pool with RBD, you need to create
  # two pools. one erasure coded and one replicated.
  # You need to specify the replicated pool here in the `pool` parameter, it is
  # used for the metadata of the images.
  # The erasure coded pool must be set as the `dataPool` parameter below.
  #dataPool: ec-data-pool
  pool: replicapool #存储类对应的存储池

  # (optional) mapOptions is a comma-separated list of map options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # mapOptions: lock_on_read,queue_depth=1024

  # (optional) unmapOptions is a comma-separated list of unmap options.
  # For krbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd/#kernel-rbd-krbd-options
  # For nbd options refer
  # https://docs.ceph.com/docs/master/man/8/rbd-nbd/#options
  # unmapOptions: force

  # RBD image format. Defaults to "2".
  imageFormat: "2"

  # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
  imageFeatures: layering

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster
  # Specify the filesystem type of the volume. If not specified, csi-provisioner
  # will set default as `ext4`. Note that `xfs` is not recommended due to potential deadlock
  # in hyperconverged settings where the volume is mounted on the same node as the osds.
  csi.storage.k8s.io/fstype: ext4
# uncomment the following to use rbd-nbd as mounter on supported nodes
# **IMPORTANT**: If you are using rbd-nbd as the mounter, during upgrade you will be hit a ceph-csi
# issue that causes the mount to be disconnected. You will need to follow special upgrade steps
# to restart your application pods. Therefore, this option is not recommended.
#mounter: rbd-nbd
allowVolumeExpansion: true
reclaimPolicy: Delete #回收策略
```



创建存储池和存储类

```bash
kubectl apply -f /csi/rbd/storageclass.yaml
```



查看存储池和存储类

```bash
kubectl get CephBlockPool -n rook-ceph
```



```bash
kubectl get storageclass
```



```bash
root@node1:~/rook/cluster/examples/kubernetes/ceph/csi/rbd# kubectl get CephBlockPool -n rook-ceph
NAME          AGE
replicapool   61s
root@node1:~/rook/cluster/examples/kubernetes/ceph/csi/rbd# kubectl get storageclass
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   78s
```



在dashboard上进行查看

![image-20221206145529601](README.assets/image-20221206145529601.png)



## 挂载测试1: MySQL 数据库存储的挂载

查看mysql示例配置文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim #PVC名称
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block #存储类型
  accessModes:
    - ReadWriteOnce #访问模式
  resources:
    requests:
      storage: 20Gi #存储大小,建议修改为2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: changeme
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql #挂接点
      volumes:
        - name: mysql-persistent-storage #数据卷
          persistentVolumeClaim:
            claimName: mysql-pv-claim #pvc名称
```



安装mysql

```bash
kubectl apply -f mysql.yaml
```



查看pv pvc

```
kubectl get pv
```



```
kubectl get pvc
```



```bash
root@node1:~/rook/cluster/examples/kubernetes# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS      REASON   AGE
pvc-1b7011a4-e4a4-4018-a61f-1a9fc58e127d   2Gi        RWO            Delete           Bound    default/mysql-pv-claim   rook-ceph-block            2m12s
root@node1:~/rook/cluster/examples/kubernetes# kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
mysql-pv-claim   Bound    pvc-1b7011a4-e4a4-4018-a61f-1a9fc58e127d   2Gi        RWO            rook-ceph-block   2m21s
```



查看pv详细信息

```bash
root@node1:~/rook/cluster/examples/kubernetes# kubectl describe pv pvc-1b7011a4-e4a4-4018-a61f-1a9fc58e127d
Name:            pvc-1b7011a4-e4a4-4018-a61f-1a9fc58e127d
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: rook-ceph.rbd.csi.ceph.com
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    rook-ceph-block
Status:          Bound
Claim:           default/mysql-pv-claim
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            rook-ceph.rbd.csi.ceph.com
    FSType:            ext4
    VolumeHandle:      0001-0009-rook-ceph-0000000000000002-e537da68-7535-11ed-b76e-d6decc20daf2
    ReadOnly:          false
    VolumeAttributes:      clusterID=rook-ceph
                           csi.storage.k8s.io/pv/name=pvc-1b7011a4-e4a4-4018-a61f-1a9fc58e127d
                           csi.storage.k8s.io/pvc/name=mysql-pv-claim
                           csi.storage.k8s.io/pvc/namespace=default
                           imageFeatures=layering
                           imageFormat=2
                           imageName=csi-vol-e537da68-7535-11ed-b76e-d6decc20daf2
                           journalPool=replicapool
                           pool=replicapool
                           storage.kubernetes.io/csiProvisionerIdentity=1670290455468-8081-rook-ceph.rbd.csi.ceph.com
Events:                <none>
```

​	注意观察上述输出的`VolumeHandle`值



从dashboard上进行检查

![image-20221206152607267](README.assets/image-20221206152607267.png)





## 挂载测试2: StatefulSet应用的挂载

创建配置文件

```bash
nano nginxsts.yaml
```



```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx 
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 1Gi
```



安装sts

```bash
kubectl apply -f nginxsts.yaml
```



查看pod

```
kubectl get pod | grep web
```



```bash
root@node1:~# kubectl get pod | grep web
web-0                              1/1     Running   0          2m39s
web-1                              1/1     Running   0          2m9s
web-2                              1/1     Running   0          100s
```



查看pv pvc

```bash
kubectl get pv | grep web
```



```bash
kubectl get pvc | grep web
```



```bash
root@node1:~# kubectl get pv | grep web
pvc-56697229-9847-4140-86cb-abd4a665fdc6   1Gi        RWO            Delete           Bound    default/www-web-0        rook-ceph-block            3m21s
pvc-83f6be05-4088-4847-b21d-4592bb71c497   1Gi        RWO            Delete           Bound    default/www-web-2        rook-ceph-block            2m22s
pvc-c90c79e7-3236-4bac-b042-e0eaef2833d9   1Gi        RWO            Delete           Bound    default/www-web-1        rook-ceph-block            2m52s
root@node1:~# kubectl get pvc | grep web
www-web-0        Bound    pvc-56697229-9847-4140-86cb-abd4a665fdc6   1Gi        RWO            rook-ceph-block   3m33s
www-web-1        Bound    pvc-c90c79e7-3236-4bac-b042-e0eaef2833d9   1Gi        RWO            rook-ceph-block   3m3s
www-web-2        Bound    pvc-83f6be05-4088-4847-b21d-4592bb71c497   1Gi        RWO            rook-ceph-block   2m34s
```



dashboard上进行查看

![image-20221206154530714](README.assets/image-20221206154530714.png)
