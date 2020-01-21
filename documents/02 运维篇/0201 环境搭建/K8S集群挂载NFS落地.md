# K8S集群挂载NFS

> 由于最近项目上的事情较多，好久没有解锁新“姿势”了，今天偶得片刻闲暇，记录一次前几天搭建测试环境的过程，主要是在之前一篇《Kubernets环境搭建 & Nginx搭建》文章基础上挂载nfs，便于日后查阅。
>
> 说明：由于公司测试环境服务器操作系统使用CentOS7，所以本文主要是CentOS7为前提。

## 1. 安装NFS

在需要使用到NFS服务的所有服务器上安装NFS服务：

```properties
yum install -y nfs-utils
```

### 1.1. 服务端配置

```properties
vim /etc/exports
/data/nfs/ 172.16.20.0/24(rw,sync,fsid=0)
```

**说明：**

* 同172.16.20.0/24一个网络号的主机可以挂载NFS服务器上的/data/nfs/目录到自己的文件系统中
* rw表示可读写；sync表示同步写，fsid=0表示将/data找个目录包装成根目录



#### 1.1.1. 启动nfs服务

**先为rpcbind和nfs做开机启动：(必须先启动rpcbind服务)**

```properties
systemctl enable rpcbind
systemctl enable nfs-server

systemctl restart rpcbind
systemctl restart nfs-server

rpcinfo -p
# 检查 NFS 服务器是否挂载我们想共享的目录 /data/nfs/：
exportfs -r
##使配置生效

exportfs
#可以查看到已经ok
#/data/nfs 172.16.20.0/24

```

### 1.2. 客户端配置

安装完nfs服务后，同服务端配置，启动rpcbind服务

```properties
#先为rpcbind做开机启动
systemctl enable rpcbind
#然后启动rpcbind服务
systemctl restart rpcbind
```

**注意：**客户端不需要启动nfs服务



检查 NFS 服务器端是否有目录共享：``` showmount -e nfs服务器的IP ```

```properties
showmount -e 172.16.20.140
Export list for 172.16.20.140:
/data/nfs 172.16.20.0/24
```

#### 1.2.1. 挂载服务器的目录

在从机上使用 mount 挂载服务器端的目录/data/nfs到客户端某个目录下：

```properties
cd /home && mkdir nfs
mount -t nfs 172.16.20.140:/data/nfs /home/nfs
```

```df -h``` 查看是否挂载成功

```properties
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   36G  8.5G   27G  25% /
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G  185M  1.7G  10% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1               1014M  146M  869M  15% /boot
tmpfs                    379M     0  379M   0% /run/user/0
172.16.20.140:/data/nfs   36G   14G   23G  38% /home/nfs
```

可以看出172.16.20.140:/data/nfs已经成功挂载到本地的 /home/nfs 目录上面了。

## 2. 部署K8S

### 2.1. nfs-pv-121-20gi.yaml（随意取的文件名）

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: charge-nfs-pv-10gi
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: charge-storage
  nfs:
    path: /data/nfs
    server: 172.16.20.140
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: charge-nfs
  namespace: default
spec:
  storageClassName: charge-storage
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 1Gi
---
```

**说明：**

* charge-nfs-pv-10gi : PersistentVolume名称
* charge-storage : 仓库名称
* charge-nfs :  申明



### 2.2. 部署nfs服务

```properties
kubectl apply -f nfs-pv-121-20gi.yaml
```

查看是否成功

```properties
kubectl get pv
```



### 2.3. 容器挂载nfs

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: www-kavip
spec:
  selector:
    matchLabels:
      app: www-kavip
  replicas: 2
  template:
    metadata:
      labels:
        app: www-kavip
    spec:
      containers:
      - name: www-kavip
        image: registry.letgo.fun/charge/www.kavip.com:1.0-test-t00
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: charge-storage
          mountPath: "/usr/local/tomcat/logs/app/"
      volumes:
      - name: charge-storage
        persistentVolumeClaim:
          claimName: charge-nfs
---
apiVersion: v1
kind: Service
metadata:
  name: www-kavip-lb
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  type: NodePort
  selector:
    app: www-kavip
---
```



## 3. 附录

#### 3.1. Jenkins部署配置

```properties
docker login -u admin -p Softisland registry.letgo.fun

cd com-softisland-admin
docker build -t registry.letgo.fun/charge/admin:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/admin:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/admin:1.0-test-t${BUILD_NUMBER}

cd ../com-softisland-client/com-softisland-client-web
docker build -t registry.letgo.fun/charge/www.kavip.com:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/www.kavip.com:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/www.kavip.com:1.0-test-t${BUILD_NUMBER}

cd ../com-softisland-client-wap
docker build -t registry.letgo.fun/charge/wap.kavip.com:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/wap.kavip.com:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/wap.kavip.com:1.0-test-t${BUILD_NUMBER}

cd ../com-softisland-client-wechatka
docker build -t registry.letgo.fun/charge/www.wechatka.com:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/www.wechatka.com:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/www.wechatka.com:1.0-test-t${BUILD_NUMBER}

cd ../com-softisland-client-wechatka-wap
docker build -t registry.letgo.fun/charge/wap.wechatka.com:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/wap.wechatka.com:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/wap.wechatka.com:1.0-test-t${BUILD_NUMBER}

cd ../../com-softisland-scheduled
docker build -t registry.letgo.fun/charge/scheduled:1.0-test-t${BUILD_NUMBER} -f Dockerfile .
docker push registry.letgo.fun/charge/scheduled:1.0-test-t${BUILD_NUMBER}
docker rmi registry.letgo.fun/charge/scheduled:1.0-test-t${BUILD_NUMBER}


####################################################################################


cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-admin-kavip-test.yaml
cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-scheduled-kavip-test.yaml
cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-wap-kavip-test.yaml
cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-wap-wechatka-test.yaml
cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-www-kavip-test.yaml
cd /var/lib/jenkins/jobs/charge-platform/workspace/ && sed -i.bak 's/1.0-test-t\([0-9]\)*/1.0-test-t'${BUILD_NUMBER}'/g' ./deployment-www-wechatka-test.yaml


kubectl apply -f deployment-admin-kavip-test.yaml
kubectl apply -f deployment-scheduled-kavip-test.yaml
kubectl apply -f deployment-wap-kavip-test.yaml
kubectl apply -f deployment-wap-wechatka-test.yaml
kubectl apply -f deployment-www-kavip-test.yaml
kubectl apply -f deployment-www-wechatka-test.yaml



#kubectl delete deployment admin-kavip || true
#kubectl delete deployment www-kavip || true
#kubectl delete deployment wap-kavip || true
#kubectl delete deployment www-wechatka || true
#kubectl delete deployment wap-wechatka || true
#kubectl delete deployment charge-scheduled || true

#kubectl run admin-kavip --image=registry.letgo.fun/charge/admin:${BUILD_NUMBER} --replicas=1 --port=8080
#kubectl run www-kavip --image=registry.letgo.fun/charge/www.kavip.com:${BUILD_NUMBER} --replicas=1 --port=8080
#kubectl run wap-kavip --image=registry.letgo.fun/charge/wap.kavip.com:${BUILD_NUMBER} --replicas=1 --port=8080
#kubectl run www-wechatka --image=registry.letgo.fun/charge/www.wechatka.com:${BUILD_NUMBER} --replicas=1 --port=8080
#kubectl run wap-wechatka --image=registry.letgo.fun/charge/wap.wechatka.com:${BUILD_NUMBER} --replicas=1 --port=8080
#kubectl run charge-scheduled --image=registry.letgo.fun/charge/scheduled:${BUILD_NUMBER} --replicas=1 --port=8080

#kubectl expose deployment admin-kavip --port=8080 --type=LoadBalancer || true
#kubectl expose deployment www-kavip --port=8080 --type=LoadBalancer || true
#kubectl expose deployment wap-kavip --port=8080 --type=LoadBalancer || true
#kubectl expose deployment www-wechatka --port=8080 --type=LoadBalancer || true
#kubectl expose deployment wap-wechatka --port=8080 --type=LoadBalancer || true
#kubectl expose deployment charge-scheduled --port=8080 --type=LoadBalancer || true
```

