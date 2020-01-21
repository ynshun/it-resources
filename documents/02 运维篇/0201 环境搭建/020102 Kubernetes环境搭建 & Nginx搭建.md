> 自公司运维团队从2年前引入了K8S来管理生产环境，经本人学习了解顿时就对他产生了浓厚的兴趣，作为一个不安分的主，怎能不放手一玩呢，所以得此闲暇时间，也在测试环境中搭建了一套，在此也留下足印，便于日后查阅。
>
> 特别鸣谢：千峰教育[李卫民](https://funtl.com/)老师，本文大量借鉴其博客内容。

# 1 理论篇

## 1.1 什么是 Kubernetes

### 1.1.1 概述

Kubernetes 是 **Google 2014 年创建管理的**，是 Google 10 多年大规模容器管理技术 Borg 的开源版本。

Kubernetes 是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。使用 Kubernetes 我们可以：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用

Kubernetes 的目标是促进完善组件和工具的生态系统，以减轻应用程序在公有云或私有云中运行的负担。

### 1.1.2 特点

- **可移植：** 支持公有云，私有云，混合云，多重云（多个公共云）
- **可扩展：** 模块化，插件化，可挂载，可组合
- **自动化：** 自动部署，自动重启，自动复制，自动伸缩/扩展



## 1.2 凭什么 Kubernetes这么火？

### 1.2.1 传统的部署方式的缺点

传统的应用部署方式是通过插件或脚本来安装应用。这样做的缺点是应用的运行、配置、管理、所有生存周期将与当前操作系统绑定，这样做并不利于应用的升级更新/回滚等操作，当然也可以通过创建虚机的方式来实现某些功能，但是虚拟机非常重，并不利于可移植性

### 1.2.2 容器化部署的优势

- **快速创建/部署应用：** 与虚拟机相比，容器镜像的创建更加容易。
- **持续开发、集成和部署：** 提供可靠且频繁的容器镜像构建/部署，并使用快速和简单的回滚(由于镜像不可变性)。
- **开发和运行相分离：** 在 build 或者 release 阶段创建容器镜像，使得应用和基础设施解耦。
- **开发，测试和生产环境一致性：** 在本地或外网（生产环境）运行的一致性。
- **云平台或其他操作系统：** 可以在 Ubuntu、RHEL、CoreOS、on-prem、Google Container Engine 或其它任何环境中运行。
- **分布式，弹性，微服务化：** 应用程序分为更小的、独立的部件，可以动态部署和管理。
- **资源隔离**
- **资源利用更高效**

### 1.3 为什么需要 Kubernetes

可以在物理或虚拟机的 Kubernetes 集群上运行容器化应用，Kubernetes 能提供一个以 **“容器为中心的基础架构”**，满足在生产环境中运行应用的一些常见需求，如：

- 多个进程协同工作
- 存储系统挂载
- 应用健康检查
- 应用实例的复制
- 自动伸缩/扩展
- 注册与发现
- 负载均衡
- 滚动更新
- 资源监控
- 日志访问
- 调试应用程序
- 提供认证和授权

# 2 准备篇

## 2.1 硬件资源

本人在搭建的时候使用了三台 CentOS 7.7系统的测试服务器来安装Kubernetes集群环境，集群节点为1主2从模式，其服务器配置要求如下：

- OS：Ubuntu Server X64 18.04 LTS（16.04 版本步骤相同，再之前则不同）
- CPU：最低要求，1 CPU 2 核
- 内存：最低要求，2GB
- 磁盘：最低要求，20GB



三台机器基本信息如下：

- 10.10.10.129 - kubernetes-master（```hostnamectl set-hostname kubernetes-master```）
- 10.10.10.130 - kubernetes-slave1（```hostnamectl set-hostname kubernetes-slave1```）
- 10.10.10.131 - kubernetes-slave2（```hostnamectl set-hostname kubernetes-slave2```）

**温馨提示：** 本人在安装过程中遇到过软件默认配置使用了服务器名来进行通信，但是又提示网络不通，故配置了Hosts，不知道是不是这里的问题，反正后面问题就莫名的消失了。



作者设置完成后的信息

```properties
Static hostname: kubernetes-master
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 3f2e77d166694051ab975fae0c0379f3
           Boot ID: a31bdc7ece524d8e8bcc7d3662f56b32
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1062.el7.x86_64
      Architecture: x86-64

```

```properties
Static hostname: kubernetes-slave1
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 3f2e77d166694051ab975fae0c0379f3
           Boot ID: 81ad4f6cedab44d6b03853152bb0482f
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1062.el7.x86_64
      Architecture: x86-64
```

```properties
Static hostname: kubernetes-slave2
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 3f2e77d166694051ab975fae0c0379f3
           Boot ID: 0b85da0d211142069b157ddef3142539
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1062.el7.x86_64
      Architecture: x86-64
```



## 2.2  调整系统参数

```properties
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

swapoff -a
```





## 2.3 软件资源

### 2.3.1 Docker安装

本人作者安装时使用Docker-ce 18.09.6

#### 2.3.1.1 查看已安装版本

如果新机器从未安装过可忽略

```properties
yum list installed | grep docker
```

#### 2.3.1.2 删除已安装的docker版本

如果新机器从未安装过可忽略

``` properties
yum -y remove docker-*
```

#### 2.3.1.3 安装国内阿里云镜像

```properties
yum install -y yum-utils device-mapper-persistent-data lvm2
```

```properties
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



#### 2.3.1.4 查看可用版本

```properties
yum list docker-ce --showduplicates
```

#### 2.3.1.5 安装Docker

```properties
yum -y install docker-ce
```

**温馨提示：**安装完成后查看一下当前安装的信息``` yum list installed | grep docker ```

```properties
containerd.io.x86_64                 1.2.10-3.2.el7                 @docker-ce-stable
docker-ce.x86_64                     3:18.09.0-3.el7                @docker-ce-stable
docker-ce-cli.x86_64                 1:18.09.0-3.el7                @docker-ce-stable
```



最后安装完成后的信息：``` docker version```

```properties
Client:
 Version:           18.09.0
 API version:       1.39
 Go version:        go1.10.4
 Git commit:        4d60db4
 Built:             Wed Nov  7 00:48:22 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.0
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       4d60db4
  Built:            Wed Nov  7 00:19:08 2018
  OS/Arch:          linux/amd64
  Experimental:     false
```

#### 2.3.1.6 配置Docker加速器

对于使用 **systemd** 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```

验证一下是否配置成功：

```properties
[root@kubernetes-master ~]# systemctl enable docker && systemctl restart docker
[root@kubernetes-master ~]# docker info

Containers: 39
 Running: 36
 Paused: 0
 Stopped: 3
Images: 12
Server Version: 18.09.0
Storage Driver: devicemapper
 Pool Name: docker-253:0-67202723-pool
 Pool Blocksize: 65.54kB
 Base Device Size: 10.74GB
 Backing Filesystem: xfs
 Udev Sync Supported: true
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
 Data Space Used: 1.533GB
 Data Space Total: 107.4GB
 Data Space Available: 49.75GB
 Metadata Space Used: 19.86MB
 Metadata Space Total: 2.147GB
 Metadata Space Available: 2.128GB
 Thin Pool Minimum Free Space: 10.74GB
 Deferred Removal Enabled: true
 Deferred Deletion Enabled: true
 Deferred Deleted Device Count: 0
 Library Version: 1.02.158-RHEL7 (2019-05-13)
Logging Driver: json-file
Cgroup Driver: systemd
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-1062.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 1.777GiB
Name: kubernetes-master
ID: FUWT:KP5W:HWVP:AZXB:FQMJ:47J3:G4WG:H66D:OBQI:U24L:VIY6:5EOW
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 registry.letgo.fun
 127.0.0.0/8
Registry Mirrors:
 https://registry.docker-cn.com/
Live Restore Enabled: false
Product License: Community Engine
```

# 3 实战篇

## 3.1 安装kubeadm，kubelet，kubectl

安装阿里镜像源

```properties
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

开始安装

```properties
yum -y install kubelet kubeadm kubectl
```

设置kubelet自启动，并启动kubelet

```properties
systemctl enable kubelet && systemctl start kubelet
```

- kubeadm：用于初始化 Kubernetes 集群
- kubectl：Kubernetes 的命令行工具，主要作用是部署和管理应用，查看各种资源，创建，删除和更新组件
- kubelet：主要负责启动 Pod 和容器

**特别说明：**该步骤在Master & Slave节点均需要执行。



## 3.2 配置kubeadm(仅Master操作)

安装 kubernetes 主要是安装它的各个镜像，而 kubeadm 已经为我们集成好了运行 kubernetes 所需的基本镜像。但由于国内的网络原因，在搭建环境时，无法拉取到这些镜像。此时我们只需要修改为阿里云提供的镜像服务即可解决该问题。

### 3.2.1 导出配置文件

```properties
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml
```

### 3.2.2 修改配置文件

```properties
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.10.10.129
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: kubernetes-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.17.0
networking:
  dnsDomain: cluster.local
  podSubnet: 192.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

* 修改文件中```advertiseAddress``` 为Master的IP
* 修改文件中```imageRepository```为 ```registry.aliyuncs.com/google_containers```
* 修改/新增文件中```podSubnet```为``` 192.168.0.0/16```（本文的网络使用Calico，默认配置）

### 3.2.3 查看和拉取镜像

```properties
# 查看所需镜像列表
kubeadm config images list --config kubeadm.yml

# 拉取镜像
kubeadm config images pull --config kubeadm.yml
```

### 3.2.4 安装Kubernetes主节点

```properties
kubeadm init --config=kubeadm.yml
```

```properties
# 安装成功则会有如下输出
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

# 后面子节点加入需要如下命令
kubeadm join 10.10.10.129:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:cab7c86212535adde6b8d1c7415e81847715cfc8629bb1d270b601744d662515
```

**说明：**等待安装完成即可，不过此步骤过程中可能会遇到一些错误，需要自行查看日志信息进行更改。如果遇到错误后可通过``` kubeadm reset```命令重置后再初始化即可。



**kubeadm init 的执行过程**：

- init：指定版本进行初始化操作
- preflight：初始化前的检查和下载所需要的 Docker 镜像文件
- kubelet-start：生成 kubelet 的配置文件 `var/lib/kubelet/config.yaml`，没有这个文件 kubelet 无法启动，所以初始化之前的 kubelet 实际上启动不会成功
- certificates：生成 Kubernetes 使用的证书，存放在 `/etc/kubernetes/pki` 目录中
- kubeconfig：生成 KubeConfig 文件，存放在 `/etc/kubernetes` 目录中，组件之间通信需要使用对应文件
- control-plane：使用 `/etc/kubernetes/manifest` 目录下的 YAML 文件，安装 Master 组件
- etcd：使用 `/etc/kubernetes/manifest/etcd.yaml` 安装 Etcd 服务
- wait-control-plane：等待 control-plan 部署的 Master 组件启动
- apiclient：检查 Master 组件服务状态。
- uploadconfig：更新配置
- kubelet：使用 configMap 配置 kubelet
- patchnode：更新 CNI 信息到 Node 上，通过注释的方式记录
- mark-control-plane：为当前节点打标签，打了角色 Master，和不可调度标签，这样默认就不会使用 Master 节点来运行 Pod
- bootstrap-token：生成 token 记录下来，后边使用 `kubeadm join` 往集群中添加节点时会用到
- addons：安装附加组件 CoreDNS 和 kube-proxy



### 3.2.5 配置kubectl

根据初始化成功时，输出的日志信息，完成基础配置：

```properties
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# 当前用户为非 ROOT用户时需要执行
chown $(id -u):$(id -g) $HOME/.kube/config
```

配置完成后可通过```kubectl get nodes``` 验证一下是否能成功获取如下信息

```properties
NAME                STATUS      ROLES    AGE    VERSION
kubernetes-master   NotReady    master   23h    v1.17.0
```



## 3.3 配置Slave节点

在Master节点中，通过```kubeadm token create --print-join-command```命令生成并查看最新的Slave加入命令

```properties
kubeadm join 10.10.10.129:6443 --token 8ib7l5.s3jj5rvl8ks0p4mr     --discovery-token-ca-cert-hash sha256:b605393cbb67d4dfeb1946e26c7c6f880be8e6fd2f61a4190627b20a00ea96a2
```

复制加入命令，在Slave节点执行即可。完成后在Master节点中执行```kubectl get nodes ··命令查看加入情况

```properties
NAME                STATUS      ROLES    AGE     VERSION
kubernetes-master   NotReady    master   23h     v1.17.0
kubernetes-slave1   NotReady    <none>   22h     v1.17.0
kubernetes-slave2   NotReady    <none>   4h14m   v1.17.0
```

**说明：**如果 slave 节点加入 master 时配置有问题可以在 slave 节点上使用 `kubeadm reset` 重置配置再使用 `kubeadm join` 命令重新加入即可。希望在 master 节点删除 node ，可以使用 `kubeadm delete nodes ` 删除。

## 3.4 安装网络插件(仅Master操作)

通过```kubectl get pod -n kube-system -o wide```查看pod状态

```properties
NAME                                        READY   STATUS    RESTARTS   IP
coredns-8686dcc4fd-gwrmb                    0/1     Pending   0          <none>
coredns-8686dcc4fd-j6gfk                    0/1     Pending   0          <none>
etcd-kubernetes-master                      1/1     Running   1          10.10.10.129
kube-apiserver-kubernetes-master            1/1     Running   1          10.10.10.129
kube-controller-manager-kubernetes-master   1/1     Running   1          10.10.10.129
kube-proxy-496dr                            1/1     Running   0          10.10.10.130
kube-proxy-db3ef                            1/1     Running   0          10.10.10.131
kube-proxy-rsnb6                            1/1     Running   1          10.10.10.129
kube-scheduler-kubernetes-master            1/1     Running   1          10.10.10.129
```

由此可以看出 coredns 尚未运行，此时我们还需要安装网络插件。

### 3.4.1  概述

容器网络是容器选择连接到其他容器、主机和外部网络的机制。容器的 runtime 提供了各种网络模式，每种模式都会产生不同的体验。例如，Docker 默认情况下可以为容器配置以下网络：

- **none：** 将容器添加到一个容器专门的网络堆栈中，没有对外连接。
- **host：** 将容器添加到主机的网络堆栈中，没有隔离。
- **default bridge：** 默认网络模式。每个容器可以通过 IP 地址相互连接。
- **自定义网桥：** 用户定义的网桥，具有更多的灵活性、隔离性和其他便利功能。

### 3.4.2 什么是 CNI

CNI(Container Network Interface) 是一个标准的，通用的接口。在容器平台，Docker，Kubernetes，Mesos 容器网络解决方案 flannel，calico，weave。只要提供一个标准的接口，就能为同样满足该协议的所有容器平台提供网络功能，而 CNI 正是这样的一个标准接口协议。

### 3.4.3 Kubernetes 中的 CNI 插件

CNI 的初衷是创建一个框架，用于在配置或销毁容器时动态配置适当的网络配置和资源。插件负责为接口配置和管理 IP 地址，并且通常提供与 IP 管理、每个容器的 IP 分配、以及多主机连接相关的功能。容器运行时会调用网络插件，从而在容器启动时分配 IP 地址并配置网络，并在删除容器时再次调用它以清理这些资源。

运行时或协调器决定了容器应该加入哪个网络以及它需要调用哪个插件。然后，插件会将接口添加到容器网络命名空间中，作为一个 veth 对的一侧。接着，它会在主机上进行更改，包括将 veth 的其他部分连接到网桥。再之后，它会通过调用单独的 IPAM（IP地址管理）插件来分配 IP 地址并设置路由。

在 Kubernetes 中，kubelet 可以在适当的时间调用它找到的插件，为通过 kubelet 启动的 pod进行自动的网络配置。

Kubernetes 中可选的 CNI 插件如下：

- Flannel
- Calico
- Canal
- Weave

### 3.4.4 什么是 Calico

Calico 为容器和虚拟机提供了安全的网络连接解决方案，并经过了大规模生产验证（在公有云和跨数千个集群节点中），可与 Kubernetes，OpenShift，Docker，Mesos，DC / OS 和 OpenStack 集成。

Calico 还提供网络安全规则的动态实施。使用 Calico 的简单策略语言，您可以实现对容器，虚拟机工作负载和裸机主机端点之间通信的细粒度控制。

### 3.4.5 安装网络插件 Calico

> **注意：**截止2019年12月19日，Calico官方最新版本为3.10

参考官方文档安装：https://docs.projectcalico.org/v3.10/getting-started/kubernetes/

```properties
# 在 Master 节点操作即可
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml

# 安装时显示如下输出
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.extensions/calico-node created
serviceaccount/calico-node created
deployment.extensions/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
```

确认安装是否成功

```properties
watch kubectl get pods --all-namespaces

# 需要等待所有状态为 Running，注意时间可能较久，3 - 5 分钟的样子
NAMESPACE     NAME                                        READY   STATUS    RESTARTS
kube-system   calico-kube-controllers-8646dd497f-g2lln    1/1     Running   0
kube-system   calico-node-8jrtp                           1/1     Running   0
kube-system   coredns-8686dcc4fd-mhwfn                    1/1     Running   0
kube-system   coredns-8686dcc4fd-xsxwk                    1/1     Running   0
kube-system   etcd-kubernetes-master                      1/1     Running   0
kube-system   kube-apiserver-kubernetes-master            1/1     Running   0
kube-system   kube-controller-manager-kubernetes-master   1/1     Running   0
kube-system   kube-proxy-p8mdw                            1/1     Running   0
kube-system   kube-scheduler-kubernetes-master            1/1     Running   0
```

至此基本环境已部署完毕

## 3.5 附录

### 3.5.1 解决 ImagePullBackOff

在使用 `watch kubectl get pods --all-namespaces` 命令观察 Pods 状态时如果出现 `ImagePullBackOff` 无法 Running 的情况，请尝试使用如下步骤处理：

- Master 中删除 Nodes：`kubectl delete nodes `
- Slave 中重置配置：`kubeadm reset`
- Slave 重启计算机：`reboot`
- Slave 重新加入集群：`kubeadm join`

### 3.5.2 解决 Unable to update cni config

Master在执行kubeadm-init 时可能会一直不成功，根据```journalctl -xefu kubelet```命令查看到如下错误

```properties
Unable to update cni config: No networks found in /etc/cni/net.d
```

可以通过如下方法来解决：

```properties
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

添加
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/ --cni-bin-dir=/opt/cni/bin"
```



### 3.5.3 解决Service NodePort不能互通

如果Service运行起来后，遇到对Node能访问，但是集群中其他Node不能访问的情况，这是由于linux还有底层的iptables，所以在node上分别执行：

```properties
splittable-P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
iptables -L -n
```








# 4 应用篇

## 4.1 部署第一个应用（Nginx）

### 4.1.1 检查组件运行状态

```properties
kubectl get cs

# 输出如下
NAME                 STATUS    MESSAGE  ERROR
scheduler            Healthy   ok                 # 调度服务，主要作用是将 POD 调度到 Node
controller-manager   Healthy   ok                 # 自动化修复服务，主要作用是Node宕机后自动修复
etcd-0               Healthy   {"health":"true"}  # 服务注册与发现
```

### 4.1.2 检查 Master 状态

```properties
kubectl cluster-info

# 输出如下
# 主节点状态
Kubernetes master is running at https://10.10.10.129:6443
# DNS 状态
KubeDNS is running at https://10.10.10.129:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```



### 4.1.3 检查 Nodes 状态

```properties
NAME                STATUS   ROLES    AGE     VERSION
kubernetes-master   Ready    master   23h     v1.17.0
kubernetes-slave1   Ready    <none>   23h     v1.17.0
kubernetes-slave2   Ready    <none>   4h39m   v1.17.0
```

### 4.1.4 运行第一个容器实例

```properties
# 使用 kubectl 命令创建两个监听 80 端口的 Nginx Pod（Kubernetes 运行容器的最小单元）
kubectl run nginx --image=nginx --replicas=2 --port=80

# 输出如下
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

### 4.1.5 查看全部 Pods 的状态

```properties
kubectl get pods

# 输出如下，需要等待一小段实践，STATUS 为 Running 即为运行成功
NAME                     READY   STATUS    RESTARTS   AGE
nginx-755464dd6c-qnmwp   1/1     Running   0          90m
nginx-755464dd6c-shqrp   1/1     Running   0          90m
```

### 4.1.6 查看已部署的服务

```properties
kubectl get deployment

# 输出如下
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           91m
```

### 4.1.7  映射服务，让用户可以访问

```properties
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# 输出如下
service/nginx exposed
```

### 4.1.8  查看已发布的服务

```properties
kubectl get services

# 输出如下
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP        23h
nginx        LoadBalancer   10.96.38.20   <pending>     80:30651/TCP   4h18m
# 由此可见，Nginx 服务已成功发布并将 80 端口映射为 30651
```

### 4.1.9 查看服务详情

```properties
kubectl describe service nginx

# 输出如下
Name:                     nginx
Namespace:                default
Labels:                   run=nginx
Annotations:              <none>
Selector:                 run=nginx
Type:                     LoadBalancer
IP:                       10.96.38.20
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30651/TCP
Endpoints:                192.168.17.1:80,192.168.8.129:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### 4.1.10 验证是否成功

通过浏览器访问 Master 服务器 ```http://10.10.10.129:30651/```

此时 Kubernetes 会以负载均衡的方式访问部署的 Nginx 服务，能够正常看到 Nginx 的欢迎页即表示成功。容器实际部署在其它 Node 节点上，通过访问 Node 节点的 IP:Port 也是可以的。

### 4.1.11 停止服务

```properties
kubectl delete deployment nginx

# 输出如下
deployment.extensions "nginx" deleted
```

```properties
kubectl delete service nginx

# 输出如下
service "nginx" deleted
```

### 4.1.12 缩放Deployment

```
kubectl scale --replicas=1 deployment/nginx
```

