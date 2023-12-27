# 准备软件



# 集群规划

## hadoop

|      | Vm60             | Vm61                       | Vm62                      |
| ---- | ---------------- | -------------------------- | ------------------------- |
| HDFS | NameNodeDataNode | DataNode                   | SecondaryNameNodeDataNode |
| YARN | NodeManager      | ResourceManagerNodeManager | NodeManager               |
|      |                  |                            |                           |
|      |                  |                            |                           |
|      |                  |                            |                           |



# 配置机器基本环境

## 基础环境设置

### 3.1 环境设置（所有节点）

#### 1.关闭防火墙

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service
```

#### 2.关闭selinux服务(必须重启才能生效，所有节点)

```shell
vim /etc/selinux/config

SELINUX=disabled
```

vim 检查selinux状态

```
sestatus -v
SELinux status: disabled 表示已经关闭了
```

#### 3.修改主机映射(所有节点)

```bash
vim /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   ::1     localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.111 master
192.168.0.112 node1
192.168.0.113 node2
192.168.0.114 node3
192.168.0.115 node4


10.8.40.60 vm60
10.8.40.61 vm61
10.8.40.62 vm62


```

#### 4.修改主机名称(所有节点)

```shell
vim /etc/hostname

master
```

#### 5.重启服务(所有节点)

`/etc/init.d/network restart(等价与service network restart)`

#### 6.配置limits.conf

```
vim /etc/security/limits.conf

* soft nofile 65536 
* hard nofile 65536
* soft nproc 401408 
* hard nproc 401408 
nexus - nofile 65536
```

#### 7.永久配置sysctl.conf（所有节点）

设置此参数主要是对linux和类Unix系统服务器交换区进行调整，通常设置的小是为了减少使用交换空间的概率)

```
echo "vm.swappiness=2" >> /etc/sysctl.conf 

vim /etc/rc.d/rc.local 
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```

#### 8.ssh免密登陆配置

```java
//生成密钥
ssh-keygen -t rsa

//在主节点将id_rsa.pub写入authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

ssh localhost (需要输入密码)

//将所有的节点伤的id_rsa.pub文件拷到 ~/.ssh文件夹下(子节点上面操作,需要输入密码)
scp ~/.ssh/id_rsa.pub  root@master:~/.ssh/id_rsa_node1.pub
scp ~/.ssh/id_rsa.pub  root@master:~/.ssh/id_rsa_node2.pub
scp ~/.ssh/id_rsa.pub  root@master:~/.ssh/id_rsa_node3.pub
scp ~/.ssh/id_rsa.pub  root@master:~/.ssh/id_rsa_node4.pub

//主节点上执行命令,将其他的.pub文件写入authorized_keys
cat ~/.ssh/id_rsa_node1.pub >>  ~/.ssh/authorized_keys
cat  ~/.ssh/id_rsa_node2.pub >>  ~/.ssh/authorized_keys
cat  ~/.ssh/id_rsa_node3.pub >>  ~/.ssh/authorized_keys
cat  ~/.ssh/id_rsa_node4.pub >>  ~/.ssh/authorized_keys

//将主节点的authorized_keys分发至每个从节点上，并给予此文件600的权限
scp ~/.ssh/authorized_keys  root@node1:~/.ssh/
scp ~/.ssh/authorized_keys  root@node2:~/.ssh/
scp ~/.ssh/authorized_keys  root@node3:~/.ssh/
scp ~/.ssh/authorized_keys  root@node4:~/.ssh/
chmod 600 ~/.ssh/authorized_keys
```

```shell
//生成密钥
ssh-keygen -t rsa

//在主节点将id_rsa.pub写入authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

ssh localhost (需要输入密码)

//将所有的节点伤的id_rsa.pub文件拷到 ~/.ssh文件夹下(子节点上面操作,需要输入密码)
scp ~/.ssh/id_rsa.pub  root@vm60:~/.ssh/id_rsa_vm60.pub
scp ~/.ssh/id_rsa.pub  root@vm60:~/.ssh/id_rsa_vm61.pub
scp ~/.ssh/id_rsa.pub  root@vm60:~/.ssh/id_rsa_vm62.pub


//主节点上执行命令,将其他的.pub文件写入authorized_keys
cat ~/.ssh/id_rsa_vm60.pub >>  ~/.ssh/authorized_keys
cat  ~/.ssh/id_rsa_vm61.pub >>  ~/.ssh/authorized_keys
cat  ~/.ssh/id_rsa_vm62.pub >>  ~/.ssh/authorized_keys


//将主节点的authorized_keys分发至每个从节点上，并给予此文件600的权限
scp ~/.ssh/authorized_keys  root@vm60:~/.ssh/
scp ~/.ssh/authorized_keys  root@vm61:~/.ssh/
scp ~/.ssh/authorized_keys  root@vm62:~/.ssh/

chmod 600 ~/.ssh/authorized_keys
```







#### 9.修改系统最大打开文件数

```
vim /etc/systemd/system.conf
[Manager]
DefaultLimitNOFILE=1024000          
DefaultLimitNPROC=1024000
```

ulimit -a 查看

#### 10.重启机器

```
reboot
```



### 3.2 JDK卸载与安装（所有节点执行）

麒麟系统自带有JDK1.8和JDK11两个版本，这两个版本都需要卸载并重新安装

```bash
rpm -qa|grep java
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.242.b08-1.h5.ky10.x86_64
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.242.b08-1.h5.ky10.x86_64
rpm -e --nodeps java-11-openjdk-headless-11.0.6.10-4.ky10.ky10.x86_64
rpm -e --nodeps java-11-openjdk-11.0.6.10-4.ky10.ky10.x86_64
```

部署

```shell
vim /etc/profile

JAVA_HOME=/usr/java/jdk1.8.0_152
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

### 3.3  MySQL安装部署和元数据初始化（主节点执行）

mysql-5.7.17-linux-glibc2.5-x86_64.tar安装

#### 1.初始化配置

##### 解压

```shell
mkdir -p /usr/local/mysql
tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
mv mysql-5.7.17-linux-glibc2.5-x86_64/* /usr/local/mysql/
```

##### 创建data目录

```shell
//二、创建data目录
mkdir -p /usr/local/mysql/data
mkdir -p /usr/local/mysql/log
mkdir -p /usr/local/mysql/tmp
```

##### 创建用户和组

```shell
//三、创建用户和组
groupadd mysql
useradd mysql -g mysql
chown -R mysql.mysql /usr/local/mysql/
```

##### 初始化

```shell
//四、初始化
//1、调整操作系统的open files限制
vim /etc/security/limits.conf

hard nofile 65535
soft nofile 65535

cd /usr/local/mysql
./bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/
```

##### 复制配置文件

```shell
//五、复制配置文件到 /etc/my.cnf
cp -a /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf (选择y)
```

#### 2.修改配置文件

##### 修改my.cnf

vim /etc/my.cnf

```bash
[mysqld]

basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
socket = /tmp/mysql.sock
character-set-server = utf8
max_allowed_packet = 200M
group_concat_max_len=4294967295
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION





七、启动mysql
/usr/local/mysql/support-files/mysql.server start

//设置开机自启动
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
//赋予可执行权限
chmod +x /etc/init.d/mysql   
//添加服务
chkconfig --add mysql 
//显示服务列表
chkconfig --list
//如果看到mysql服务3,4,5都是on,则成功.否则执行如下语句
chkconfig --level 345 mysql on

//重启机器,查看开机启动是否设置成功
netstat -na | grep 3306

cat /root/.mysql_secret --查看初始化自动生成的密码,并记录下来,等会登陆mysql需要

八、登录并修改密码

skip-grant-tables

bin/mysql -uroot -p  --把刚刚复制的密码粘贴上来

mysql> SET PASSWORD  FOR 'root'@localhost = PASSWORD('root');--重置密码

use mysql;
update user set host='%' where user='root';
flush privileges;

netstat -nlp  --查看3306端口是否开通
```

##### 加入环境变量

```shell
vim /etc/profile
export PATH=$PATH:/usr/local/mysql/bin
source /etc/profile


mysql文件备份
mysqldump -uroot -p db.name > aa.sql
```

### 3.3  时间服务器同步

#### 1.master节点

```bash
# master节点执行
yum -y install chrony

vim /etc/chrony.conf
  server 192.168.110.130 iburst
  allow 192.168.119.0/24 
  local stratum 10

systemctl restart chronyd.service
systemctl enable chronyd.service

# 查看时间同步状态
chronyc -a makestep
chronyc sourcestats
chronyc sources -v

```



#### 2.其他节点

```shell
# 其他节点执行
yum -y install chrony
vim /etc/chrony.conf
	server 192.168.110.130 iburst

systemctl restart chronyd.service
systemctl enable chronyd.service

```



## 环境测试

###  防火墙

```shell
[root@vm60 ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@vm61 ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@vm62 ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```



### 关闭selinux服务(必须重启才能生效，所有节点)

```shell
[root@vm60 ~]# sestatus -v
SELinux status:                 disabled
[root@vm61 ~]# sestatus -v
SELinux status:                 disabled
[root@vm62 ~]# sestatus -v
SELinux status:                 disabled
```



### 修改主机映射

```shell
[root@vm60 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.8.40.60 vm60
10.8.40.61 vm61
10.8.40.62 vm62
[root@vm61 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.8.40.60 vm60
10.8.40.61 vm61
10.8.40.62 vm62
[root@vm62 ~]# sestatus -v
SELinux status:                 disabled
[root@vm62 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.8.40.60 vm60
10.8.40.61 vm61
10.8.40.62 vm62
```



### 修改主机名称

```shell
[root@vm60 ~]# cat /etc/hostname 
vm60
[root@vm61 ~]# cat /etc/hostname
vm61
[root@vm62 ~]# cat /etc/hostname
vm62
```

### 配置limits.conf

```shell
vim /etc/security/limits.conf

* soft nofile 65536 
* hard nofile 65536
* soft nproc 401408 
* hard nproc 401408 
nexus - nofile 65536
```

### ssh免密登陆配置

```shell


[root@vm60 ~]# ssh vm61
Last login: Tue Dec 26 09:39:07 2023 from 10.5.27.208
[root@vm61 ~]# ssh vm62
Last login: Tue Dec 26 09:43:34 2023 from 10.5.27.208
[root@vm62 ~]# ssh vm61
Last login: Tue Dec 26 09:59:22 2023 from vm60
[root@vm61 ~]# ssh vm60
Last login: Tue Dec 26 09:30:06 2023 from 10.5.27.208
[root@vm60 ~]# ssh vm62
Last login: Tue Dec 26 09:59:31 2023 from vm61
[root@vm62 ~]# ssh vm60
Last login: Tue Dec 26 09:59:47 2023 from vm61
```

### jdk

```shell
[root@vm60 ~]# java -version
java version "1.8.0_212"
Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
[root@vm60 ~]# 
[root@vm61 ~]# date
Tue Dec 26 10:00:52 CST 2023
[root@vm61 ~]# java -version
java version "1.8.0_212"
Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
[root@vm61 ~]# 
[root@vm62 ~]# date
Tue Dec 26 10:01:03 CST 2023
[root@vm62 ~]# java -version
java version "1.8.0_212"
Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
```













# 安装

## 1、安装Hadoop

### 解压安装 

root@vm60

```shell
cd /opt/software/hadoop/02.hadoop
tar -zxvf hadoop-3.1.3.tar.gz -C /ztgx-jx/cluster/
```

查看是否解压成功

```shell
[root@vm60 02.hadoop]# ls /ztgx-jx/cluster/
hadoop-3.1.3
```



### 将hadoop添加到环境变量

#### jps获取hadoop的安装路径

```shell
[root@vm60 hadoop-3.1.3]# pwd
/ztgx-jx/cluster/hadoop-3.1.3
```

#### 打开/etc/profile.d/hadoop_env.sh文件

```shell
vim /etc/profile.d/hadoop_env.sh
```

在hadoop_env.sh文件末尾添加如下内容：

```shell
#HADOOP_HOME
export HADOOP_HOME=/ztgx-jx/cluster/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

```

#### 让修改的文件生效

```shell
source /etc/profile
```

####  测试是否安装成功

```shell
[root@vm60 hadoop-3.1.3]# hadoop version
Hadoop 3.1.3
Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
Compiled by ztang on 2019-09-12T02:47Z
Compiled with protoc 2.5.0
From source with checksum ec785077c385118ac91aadde5ec9799
This command was run using /ztgx-jx/cluster/hadoop-3.1.3/share/hadoop/common/hadoop-common-3.1.3.jar
```

#### 分发环境变量文件

```shell
[root@vm60 bin]# ./sync /etc/profile.d/hadoop_env.sh
==================== vm60 ====================
sending incremental file list

sent 51 bytes  received 12 bytes  126.00 bytes/sec
total size is 135  speedup is 2.14
==================== vm61 ====================
sending incremental file list

sent 51 bytes  received 12 bytes  126.00 bytes/sec
total size is 135  speedup is 2.14
==================== vm62 ====================
sending incremental file list

sent 51 bytes  received 12 bytes  126.00 bytes/sec
total size is 135  speedup is 2.14
```

### 配置集群

#### 核心文件配置core-site.xml

```shell
[root@vm60 bin]# cd $HADOOP_HOME/etc/hadoop
[root@vm60 hadoop]# ls
capacity-scheduler.xml            httpfs-log4j.properties     mapred-site.xml
configuration.xsl                 httpfs-signature.secret     shellprofile.d
container-executor.cfg            httpfs-site.xml             ssl-client.xml.example
core-site.xml                     kms-acls.xml                ssl-server.xml.example
hadoop-env.cmd                    kms-env.sh                  user_ec_policies.xml.template
hadoop-env.sh                     kms-log4j.properties        workers
hadoop-metrics2.properties        kms-site.xml                yarn-env.cmd
hadoop-policy.xml                 log4j.properties            yarn-env.sh
hadoop-user-functions.sh.example  mapred-env.cmd              yarnservice-log4j.properties
hdfs-site.xml                     mapred-env.sh               yarn-site.xml
httpfs-env.sh                     mapred-queues.xml.template
```

文件内容：

```shell
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://vm60:8020</value>
    </property>
    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
    </property>
    
    <!-- 配置HDFS网页登录使用的静态用户为root-->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
    </property>
    
    <!-- 配置该root(superUser)允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <!-- 配置该root(superUser)允许通过代理用户所属组 -->
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
    <!-- 配置该root(superUser)允许通过代理的用户-->
    <property>
        <name>hadoop.proxyuser.root.users</name>
        <value>*</value>
    </property>
</configuration>

```



#### HDFS配置文件

```shell
vim hdfs-site.xml
```

文件内容

```shell
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>vm60:9870</value>
    </property>
    
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>vm61:9868</value>
    </property>
    
    <!-- 测试环境指定HDFS副本的数量1 -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

```

#### YARN配置文件

```shell
vim yarn-site.xml
```

文件内容如下：

```shell
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    
    <!-- 指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>vm61</value>
    </property>
    
    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    
    <!-- yarn单个容器允许分配的最大最小内存 -->
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>
    
    <!-- yarn容器允许管理的物理内存大小 -->
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    
    <!-- 关闭yarn对虚拟内存的限制检查 -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
</configuration>

```

#### **MapReduce配置文件**

```shell
vim mapred-site.xml
```

文件内容如下：

```shell
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定MapReduce程序运行在Yarn上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

```



#### **配置workers**

```shell
vim workers
```

文件内容如下：

```shell
vm60
vm61
vm62
```



### 配置历史服务器



```shell
vi mapred-site.xml
```



```shell
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>vm60:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>vm60:19888</value>
</property>

```



### 配置日志的聚集

```shell
vim yarn-site.xml
```



```xml
<!-- 开启日志聚集功能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>

<!-- 设置日志聚集服务器地址 -->
<property>  
    <name>yarn.log.server.url</name>  
    <value>http://vm60:19888/jobhistory/logs</value>
</property>

<!-- 设置日志保留时间为7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>

```



### 检查、分发

```shell
/root/bin/sync /ztgx-jx/cluster/hadoop-3.1.3/
```





### 群起集群

#### 格式化nameNode

***\*如果集群是第一次启动\****，需要在vm60节点格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除data和log数据）

```shell
[root@vm60 hadoop-3.1.3]# pwd
/ztgx-jx/cluster/hadoop-3.1.3
[root@vm60 hadoop-3.1.3]# bin/hdfs namenode -format
```

#### 启动hdfs

```shell
sbin/start-dfs.sh
```



#### 错误处理

##### 1

[root@vm60 sbin]# start-dfs.sh
Starting namenodes on [vm60]
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [vm61]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.

vim /etc/profile.d/hadoop_env.sh

```shell
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_JOURNALNODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

export JAVA_HOME=/usr/local/java/jdk1.8.0_212
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

### 群起脚本

#### hdp.sh

```shell
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi
case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh vm60 "/ztgx-jx/cluster/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh vm61 "/ztgx-jx/cluster/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh vm60 "/ztgx-jx/cluster/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh vm60 "/ztgx-jx/cluster/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh vm61 "/ztgx-jx/cluster/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh vm60 "/ztgx-jx/cluster/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
```



#### 启动

```shell
[root@vm60 bin]# ./hdp.sh start
 =================== 启动 hadoop集群 ===================
 --------------- 启动 hdfs ---------------
Starting namenodes on [vm60]
Last login: Wed Dec 27 13:44:59 CST 2023
Starting datanodes
Last login: Wed Dec 27 13:45:07 CST 2023
Starting secondary namenodes [vm61]
Last login: Wed Dec 27 13:45:10 CST 2023
 --------------- 启动 yarn ---------------
Starting resourcemanager
Last login: Wed Dec 27 13:44:53 CST 2023
Starting nodemanagers
Last login: Wed Dec 27 13:45:18 CST 2023
 --------------- 启动 historyserver ---------------
```



#### 验证

```shell

[root@vm60 bin]# jps -l
216624 org.apache.hadoop.mapreduce.v2.hs.JobHistoryServer
216455 org.apache.hadoop.yarn.server.nodemanager.NodeManager
216085 org.apache.hadoop.hdfs.server.datanode.DataNode
216708 sun.tools.jps.Jps
215929 org.apache.hadoop.hdfs.server.namenode.NameNode
8955 org.tanukisoftware.wrapper.WrapperSimpleApp

[root@vm61 ~]# jps -l
8614 org.tanukisoftware.wrapper.WrapperSimpleApp
209336 org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
209519 org.apache.hadoop.yarn.server.resourcemanager.ResourceManager
210047 sun.tools.jps.Jps
209678 org.apache.hadoop.yarn.server.nodemanager.NodeManager
209213 org.apache.hadoop.hdfs.server.datanode.DataNode

[root@vm62 ~]# jps -l
8594 org.tanukisoftware.wrapper.WrapperSimpleApp
208599 org.apache.hadoop.hdfs.server.datanode.DataNode
208918 sun.tools.jps.Jps
208760 org.apache.hadoop.yarn.server.nodemanager.NodeManager
```



#### 停止

```shell
[root@vm60 bin]# ./hdp.sh stop
```



### 集群查看进程

```shell
#! /bin/bash
 
for i in vm60 vm61 vm62
do
    echo --------- $i ----------
    ssh $i "$*"
done
```



```shell
[root@vm60 bin]# chmod 777 psc.sh 
[root@vm60 bin]# ./psc.sh jps
--------- vm60 ----------
216624 JobHistoryServer
216455 NodeManager
216085 DataNode
217652 Jps
215929 NameNode
8955 WrapperSimpleApp
--------- vm61 ----------
210998 Jps
8614 WrapperSimpleApp
209336 SecondaryNameNode
209519 ResourceManager
209678 NodeManager
209213 DataNode
--------- vm62 ----------
8594 WrapperSimpleApp
208599 DataNode
209941 Jps
208760 NodeManager
```



## 2、zk安装

### 集群规划

在vm60、vm61和vm62三个节点上部署Zookeeper。

|           | 服务器vm60 | 服务器vm61 | 服务器vm62 |
| --------- | ---------- | ---------- | ---------- |
| Zookeeper | Zookeeper  | Zookeeper  | Zookeeper  |

### 解压安装

```shell
[root@vm60 04.zookeeper]# tar -zxvf  /opt/software/hadoop/04.zookeeper/apache-zookeeper-3.5.7-bin.tar.gz -C /ztgx-j
x/cluster/
```

修改目录名称

```shell
vm60 cluster]# ls
apache-zookeeper-3.5.7-bin  hadoop-3.1.3
[root@vm60 cluster]# mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7/
```

同步zookeeper-3.5.7目录内容到vm61、vm62

```shell
/root/bin/sync zookeeper-3.5.7/
```

### 服务器编号配置

在/ztgx-jx/cluster/zookeeper-3.5.7下创建zkData

```shell
[root@vm60 zookeeper-3.5.7]# pwd
/ztgx-jx/cluster/zookeeper-3.5.7
[root@vm60 zookeeper-3.5.7]# mkdir zkData
```

在/ztgx-jx/cluster/zookeeper-3.5.7/zkData创建myid

```shell
vi myid
```

在文件中添加与server对应的编号：

2

```shell
[root@vm60 zkData]# /root/bin/sync myid 
==================== vm60 ====================
sending incremental file list

sent 42 bytes  received 12 bytes  108.00 bytes/sec
total size is 2  speedup is 0.04
==================== vm61 ====================
sending incremental file list
myid

sent 91 bytes  received 35 bytes  252.00 bytes/sec
total size is 2  speedup is 0.02
==================== vm62 ====================
sending incremental file list
myid

sent 91 bytes  received 35 bytes  252.00 bytes/sec
total size is 2  speedup is 0.02
```



并分别在vm61、vm62上修改myid文件中内容为3、4

### 配置zoo.cfg文件

/ztgx-jx/cluster/zookeeper-3.5.7/conf这个目录下的zoo_sample.cfg为zoo.cfg

```shell
[root@vm60 conf]# pwd
/ztgx-jx/cluster/zookeeper-3.5.7/conf
[root@vm60 conf]# mv zoo_sample.cfg zoo.cfg
[root@vm60 conf]# vim zoo.cfg 
zookeeper-3.5.7/docs/apidocs/zoo
```

修改数据存储路径配置

dataDir=/ztgx-jx/cluster/zookeeper-3.5.7/zkData

```shell
增加如下配置
#######################cluster##########################
server.2=vm60:2888:3888
server.3=vm61:2888:3888
server.4=vm62:2888:3888
```



同步zoo.cfg配置文件

```shell
[root@vm60 conf]# /root/bin/sync zoo.cfg 
```



### 群启脚本

```shell
#!/bin/bash

case $1 in
"start"){
	for i in vm60 vm61 vm62
	do
        echo ---------- zookeeper $i 启动 ------------
		ssh $i "/ztgx-jx/cluster/zookeeper-3.5.7/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in vm60 vm61 vm62
	do
        echo ---------- zookeeper $i 停止 ------------    
		ssh $i "/ztgx-jx/cluster/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in vm60 vm61 vm62
	do
        echo ---------- zookeeper $i 状态 ------------    
		ssh $i "/ztgx-jx/cluster/zookeeper-3.5.7/bin/zkServer.sh status"
	done
};;
esac
```

启动、查看状态、停止

```shell
[root@vm60 conf]# chmod 777 /root/bin/zk.sh 
[root@vm60 conf]# /root/bin/zk.sh start 
---------- zookeeper vm60 启动 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
---------- zookeeper vm61 启动 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
---------- zookeeper vm62 启动 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@vm60 conf]# /root/bin/zk.sh status
---------- zookeeper vm60 状态 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
---------- zookeeper vm61 状态 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
---------- zookeeper vm62 状态 ------------
ZooKeeper JMX enabled by default
Using config: /ztgx-jx/cluster/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower


```



## 3、Hive 安装

### 安装部署

#### 解压

```shell
[root@vm60 conf]# tar -zxvf /opt/software/hadoop/03.Hive/apache-hive-3.1.2-bin.tar.gz -C /ztgx-jx/cluster/
```

#### 重命名

```shell
[root@vm60 conf]# cd  /ztgx-jx/cluster/
[root@vm60 cluster]# ls
apache-hive-3.1.2-bin  hadoop-3.1.3  zookeeper-3.5.7
[root@vm60 cluster]# pwd
/ztgx-jx/cluster
[root@vm60 cluster]# mv apache-hive-3.1.2-bin/ hive/
[root@vm60 cluster]# ls
hadoop-3.1.3  hive  zookeeper-3.5.7
[root@vm60 cluster]# 
```



#### 修改hadoop_env.sh 添加环境变量

```shell
vim /etc/profile.d/hadoop_env.sh
```



```shell
#HIVE_HOME
export HIVE_HOME=/ztgx-jx/cluster/hive
export PATH=$PATH:$HIVE_HOME/bin
```



```shell
[root@vm60 cluster]# source /etc/profile.d/hadoop_env.sh 
```

### **Hive元数据配置到MySQL**

#### 拷贝驱动

```shell
[root@vm60 HDP3.1.5]# pwd
/ztgx-jx/backup_soft/HDP3.1.5
[root@vm60 HDP3.1.5]# cp mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar /ztgx-jx/cluster/hive/lib/
[root@vm60 HDP3.1.5]#
```



#### 配置Metastore到MySQL

在$HIVE_HOME/conf目录下新建hive-site.xml文件

```shell
vim hive-site.xml
```

添加如下内容

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://vm60:3306/metastore?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>root</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>vm60</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
</configuration>
```



### 启动Hive

#### **初始化元数据库**

```shell
mysql> create database metastore;
```

mysql> create database metastore;
Query OK, 1 row affected (0.00 sec)



```shell
schematool -initSchema -dbType mysql -verbose
```

[root@vm60 conf]# pwd
/ztgx-jx/cluster/hive/conf
[root@vm60 conf]# schematool -initSchema -dbType mysql -verbose



#### 启动Hive客户端

```shell
[root@vm60 bin]# pwd
/ztgx-jx/cluster/hive/bin
[root@vm60 bin]# hive
which: no hbase in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/ztgx-jx/cluster/hadoop-3.1.3/bin:/ztgx-jx/cl
uster/hadoop-3.1.3/sbin:/usr/local/java/jdk1.8.0_212/bin:/ztgx-jx/cluster/hive/bin:/usr/local/java/jdk1.8.0_212/bin
:/ztgx-jx/mysql/app/mysql/bin:/root/bin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/ztgx-jx/cluster/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLogge
rBinder.class]
SLF4J: Found binding in [jar:file:/ztgx-jx/cluster/hadoop-3.1.3/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/o
rg/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Hive Session ID = 9f023788-2958-4dd3-9ae2-8667b9be4e45

Logging initialized using configuration in jar:file:/ztgx-jx/cluster/hive/lib/hive-common-3.1.2.jar!/hive-log4j2.pr
operties Async: true
Hive Session ID = 7773ae09-9a27-4412-b8d7-4e3934b4aace
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different exec
ution engine (i.e. spark, tez) or using Hive 1.X releases.
hive (default)> show databases;
OK
database_name
default
Time taken: 1.471 seconds, Fetched: 1 row(s)
hive (default)> 
```



## 4、Hbase安装

### HBase的解压



### 配置文件



### 远程发送到其他集群



#### HBase服务的启动







## 5、phoenix安装



## 6、Spark安装







# 总群启脚本

