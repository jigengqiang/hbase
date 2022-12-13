# 单机环境安装Zookeeper+Hadoop+Hbase+Phoenix

压缩包版本

```
hadoop-3.3.4.tar.gz
hbase-2.4.14-bin.tar.gz
apache-zookeeper-3.8.0-bin.tar.gz
phoenix-hbase-2.4-5.1.2-bin.tar.gz
```

## 安装zookeeper

### 1、解压

```
tar -zxvf apache-zookeeper-3.8.0-bin.tar.gz
```

### 2、配置环境变量

```
vi /etc/profile
```

### 3、添加环境变量

```
export JAVA_HOME=/usr/java/jdk1.8.0_341-amd64
export ZOOKEEPER_HOME=/root/zookeeper/apache-zookeeper-3.8.0-bin
export PATH=$ZOOKEEPER_HOME/bin:$PATH
```

### 4、使配置的环境变量生效

```
source /etc/profile
```

### 5、进入zookeeper安装目录下的conf 目录，复制配置样本并修改

```
cp zoo_sample.cfg  zoo.cfg
```

### 6、修改后，如下【但是我直接复制，没有做修改】

```
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
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpHost=0.0.0.0
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true
```

### 7、启动zookeeper，进入/root/zookeeper/apache-zookeeper-3.8.0-bin/bin目录，执行下面命令

```
./zkServer.sh start
```

### 8、成功启动后，输出如下

```
ZooKeeper JMX enabled by default
Using config: /root/zookeeper/apache-zookeeper-3.8.0-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

### 9、用jps查看一下，输出QuorumPeerMain代表成功，一般会输出如下

```
3157 QuorumPeerMain
3230 Jps
```

## 安装hadoop

### 1、先设置hosts文件

```
cd /etc
vi hosts
```

### 2、添加hosts内容，然后保存退出

```
127.0.0.1   hadoop001
```

### 3、配置免密登录

先生成公钥和私钥

```
ssh-keygen -t rsa
```

输出如下

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:39Qzt9Ltsisxn4oDmJuq/hZziY/CLHG//mKUkut5Tqo root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|             .   |
|   . o +S   . + .|
|. + * = .. oo .+o|
| = =.* o .. .= +.|
|. =+B +   ... =. |
|E=*X*=.   ...oo+.|
+----[SHA256]-----+
```

查看生成的公钥私钥

```
cd /root/.ssh
ls
```

输出如下

```
id_rsa  id_rsa.pub  known_hosts
```

执行命令

```
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
```

### 4、解压

```
tar -zxvf hadoop-3.3.4.tar.gz
```

配置环境变量

```
vi /etc/profile
```

在profile文件中添加

```
export HADOOP_HOME=/root/hadoop/hadoop-3.3.4
export PATH=${HADOOP_HOME}/bin:$PATH
```

使环境变量生效

```
source /etc/profile
```

### 5、修改hadoop配置

进入hadoop安装目录/etc/hadoop 目录，修改配置

①、hadoop-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_341-amd64
```

②、core-site.xml

```
<configuration>
	<property>
        <!--指定 namenode 的 hdfs 协议文件系统的通信地址-->
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop001:8020</value>
    </property>
    <property>
        <!--指定 hadoop 存储临时文件的目录-->
        <name>hadoop.tmp.dir</name>
        <value>/root/hadoop/hadoop-3.3.4/hadoopData/tmp</value>
    </property>
</configuration>
```

③、hdfs-site.xml

```
<configuration>
	<property>
        <!--由于我们这里搭建是单机版本，所以指定 dfs 的副本系数为 1-->
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

④、workers，先 vi workers，再添加如下内容，保存退出

```
hadoop001
```

⑤、关闭防火墙

```
#查看防火墙状态
sudo firewall-cmd --state
#关闭防火墙
sudo systemctl stop firewalld.service
```

### 6、初始化

进入hadoop安装目录/bin目录

```
./hdfs namenode -format
```

### 7、修改文件的头信息

在hadoop安装目录/sbin路径下，将start-dfs.sh，stop-dfs.sh两个文件顶部添加以下参数

```
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

start-yarn.sh，stop-yarn.sh顶部需添加以下

```
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

### 8、启动HDFS

进入hadoop安装目录/sbin目录，执行命令

```
./start-dfs.sh
```

输出如下，

```
WARNING: HADOOP_SECURE_DN_USER has been replaced by HDFS_DATANODE_SECURE_USER. Using value of HADOOP_SECURE_DN_USER.
Starting namenodes on [hadoop001]
上一次登录：四 9月 29 12:48:31 CST 2022从 192.168.3.11pts/2 上
hadoop001: Warning: Permanently added 'hadoop001' (ECDSA) to the list of known hosts.
Starting datanodes
上一次登录：四 9月 29 12:49:23 CST 2022pts/1 上
Starting secondary namenodes [localhost.localdomain]
上一次登录：四 9月 29 12:49:26 CST 2022pts/1 上
```

使用jps查看启动是否成功，【NameNode】、【DataNode】成功会输出如下

```
4244 Jps
3157 QuorumPeerMain
4021 SecondaryNameNode
3704 DataNode
3513 NameNode
```

### 9、修改配置

进入hadoop安装目录/etc/hadoop目录

编辑文件mapred-site.xml

添加如下内容，保存退出

```
<configuration>
	<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

编辑文件yarn-site.xml

添加如下内容，保存退出

```
<configuration>
<!-- Site specific YARN configuration properties -->
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

### 10、启动yarn

进入hadoop安装目录/sbin目录

```
./start-yarn.sh
```

验证是否启动成功，查看NodeManager、ResourceManager是否启动成功

```
jps
```

```
7810 NodeManager
8259 Jps
3157 QuorumPeerMain
4021 SecondaryNameNode
3704 DataNode
3513 NameNode
7630 ResourceManager
```

### hadoop安全模式问题

一定要切记hadoop的安全模式问题，在bin目录下，一旦出现server is not running yet首先就检查一下它

```
#下面命令是离开安全模式，在安全模式下，会出现server is not running yet
./hadoop dfsadmin -safemode leave
```



## 安装Hbase

### 解压

```
tar -zxvf hbase-2.4.14-bin.tar.gz
```

### 配置环境变量

```
export HBASE_HOME=/root/hbase/hbase-2.4.14
export PATH=$HBASE_HOME/bin:$PATH
```

### 使环境变量生效

```
source /etc/profile
```

### 配置Hbase

①修改Hbase安装目录下conf/hbase-env.sh

```
export JAVA_HOME=/usr/java/jdk1.8.0_341-amd64
# 使用外部zookeeper管理hbase
export HBASE_MANAGES_ZK=flase
```

②修改Hbase安装目录下的conf/hbase-site.xml

```
<!--指定 HBase 以分布式模式运行-->   
 <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
 </property>
 <!--指定 HBase 数据存储路径为 HDFS 上的 hbase 目录,hbase 目录不需要预先创建，程序会自动创建-->   
 <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop001:8020/hbase</value>
  </property>
    <!--指定 zookeeper 数据的存储位置-->   
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/root/zookeeper/apache-zookeeper-3.8.0-bin/conf/tmp/zookeeper</value>
  </property>
  <!--指定 Hbase Web UI 默认端口-->  
  <property>
    <name>hbase.master.info.port</name>
    <value>60010</value>
  </property>
  <!--指定外置zookeeper--> 
  <property>
   <name>hbase.zookeeper.quorum</name>
   <value>hadoop001:2181</value>
  </property>
  <property>
  <name>hbase.wal.provider</name>
  <value>filesystem</value>
</property>
```

③修改Hbase安装目录下的conf/regionservers，修改后，其内容如下

```
hadoop001
```

### 启动Hbase

Hbase安装目录的/bin下，输入命令

```
./start-hbase.sh
```

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/hadoop/hadoop-3.3.4/share/hadoop/common/lib/slf4j-reload4j-1.7.36.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/hbase/hbase-2.4.14/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Reload4jLoggerFactory]
running master, logging to /root/hbase/hbase-2.4.14/logs/hbase-root-master-localhost.localdomain.out
hadoop001: running regionserver, logging to /root/hbase/hbase-2.4.14/bin/../logs/hbase-root-regionserver-localhost.localdomain.out
```

验证是否成功启动，依旧是用jps，其中【HMaster】、【HRegionServer】是Hbase的进程

```
[root@localhost bin]# jps
7810 NodeManager
10067 HMaster
3157 QuorumPeerMain
4021 SecondaryNameNode
10391 HRegionServer
3704 DataNode
3513 NameNode
10858 Jps
7630 ResourceManager
```

## 安装phoenix

### 解压

```
tar -zxvf phoenix-hbase-2.4-5.1.2-bin.tar.gz
```

### 复制phoenix-server-hbase-2.4-5.1.2.jar到Hbase安装目录的lib 目录下

```
cp /root/phoenix/phoenix-hbase-2.4-5.1.2-bin/phoenix-server-hbase-2.4-5.1.2.jar /root/hbase/hbase-2.4.14/lib
```

### 重启 Region Servers

```
./stop-hbase.sh

```



备注，/etc/profile 完整的配置如下，这样配置肯定能确定这个文件不存在问题

```
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL
export JAVA_HOME=/usr/java/jdk1.8.0_341-amd64
export ZOOKEEPER_HOME=/root/zookeeper/apache-zookeeper-3.8.0-bin
export PATH=$ZOOKEEPER_HOME/bin:$PATH
export HADOOP_HOME=/root/hadoop/hadoop-3.3.4
export LD_LIBRARY_PATH=/root/hadoop/hadoop-3.3.4/lib/native
export PATH=${HADOOP_HOME}/bin:$PATH
export HBASE_HOME=/root/hbase/hbase-2.4.14
export PATH=$HBASE_HOME/bin:$PATH
export PHOENIX_HOME=/root/phoenix/phoenix-hbase-2.4-5.1.2-bin
export PATH=$PHOENIX_HOME/bin:$PATH
```

20220929后续

```
cp /root/phoenix/phoenix-hbase-2.4-5.1.2-bin/phoenix-pherf-5.1.2.jar /root/hbase/hbase-2.4.14/lib
```

```
export PHOENIX_HOME=/root/phoenix/phoenix-hbase-2.4-5.1.2-bin
export PATH=$PHOENIX_HOME/bin:$PATH
```

```
cp /root/hbase/hbase-2.4.14/conf/hbase-site.xml /root/phoenix/phoenix-hbase-2.4-5.1.2-bin/bin
```

## 写在最后

### 防火墙记得打开

检查防火墙端口是否开启

```
firewall-cmd --query-port=60010/tcp
```

如果未开启，则开启

```
firewall-cmd --add-port=60010/tcp --permanent
```

防火墙状态重新加载一下

```
firewall-cmd --reload
```

为了保险起见，再检查一下端口是否正常开启

```
firewall-cmd --query-port=60010/tcp
```

如果报错的话，还需要再复制两个文件，基本就正常了

```
cp /root/hadoop/hadoop-3.3.4/etc/hadoop/core-site.xml /root/hbase/hbase-2.4.14/conf
cp /root/hadoop/hadoop-3.3.4/etc/hadoop/hdfs-site.xml /root/hbase/hbase-2.4.14/conf
```

