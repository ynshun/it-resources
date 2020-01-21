> 今天打算将现有项目中用于收集日志的ActiveMQ换成最近流行的Kafka，通过其学习，发现使用到Zookeeper，平时开发/测试环境的Zookeeper都是自己搭建的单机模式，生产又一直是运维团队在维护，今天在搭建的时候就心血来潮 ，顺便把Zookeeper的集群搭一搭，接下来我们就来看一下具体的操作吧！！

### 1 下载Zookeeper

Zookeeper的下载可自行去官网下载http://mirror.bit.edu.cn/apache/zookeeper/

* 作者撰写本文时最新版本为```zookeeper-3.5.6```，故本文以此版本为基准。
* 温馨提示：下载时一定要下载```apache-zookeeper-3.5.6-bin.tar.gz```，千万不要下载```apache-zookeeper-3.5.6.tar.gz```，经作者采坑 ```apache-zookeeper-3.5.6.tar.gz```根本不完整，启动不了

### 2 配置Zookeeper

#### 2.1 安装JDK

作者使用的JDK1.8 ```Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)```

```properties
vim /etc/profile

export JAVA_HOME=/usr/local/tools/jdk1.8.0_172
export JRE_HOME=/usr/local/tools/jdk1.8.0_172/jre
export CLASS_PATH=$CLASS_PATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

source /etc/profile
```

配置好了，通过```java -version```验证一下

```properties
java version "1.8.0_172"
Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
```



#### 2.2 解压下载下来的包

``` properties
tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz
```



#### 2.3 配置环境变量

```properties
vim /etc/profile

export ZOOKEEPER_HOME=/data/software/zookeeper-2181
export PATH=$PATH:$ZOOKEEPER_HOME/bin

source /etc/profile
```

#### 2.4 配置zoo.cfg

进入到zookeeper目录``` /data/software/zookeeper-2181/conf ```，复制```zoo_sample.cfg```

```properties
cp zoo_sample.cfg zoo.cfg
```

编辑```zoo.cfg```

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/software/zookeeper-2181/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=172.16.20.112:2888:3888
server.2=172.16.20.147:2888:3888
server.3=172.16.20.148:2888:3888
```

* initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
* syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
* server.A=B:C:D  其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号

#### 2.5 配置myid

配置zookeeper集群，除了修改 zoo.cfg 配置文件，集群模式下还要配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面就有一个数据就是 A 的值，Zookeeper 启动时会读取这个文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是那个 server。

### 3 启动服务

对应每个集群节点均配置完成后，通过```zkServer.sh start ```依次将服务启动起来，待服务启动完成后，再分别在节点中查看其状态```zkServer.sh status ```

```properties
ZooKeeper JMX enabled by default
Using config: /data/software/zookeeper-2181/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
```

```properties
ZooKeeper JMX enabled by default
Using config: /data/software/zookeeper-2181/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

```properties
ZooKeeper JMX enabled by default
Using config: /root/zookeeper-2181/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

