# 国产麒麟系统部署Ambari + HDP

系统版本	Kylin Linux Advanced Server release V10 (Tercel)
系统架构	x86_64
ambari	ambari-2.7.5.0-centos7.tar.gz
HDP	HDP-3.1.5.0-centos7-rpm.tar.gz
HDP-GPL-3.1.5.0-centos7-gpl.tar.gz
HDP-UTILS-1.1.0.22-centos7.tar.gz

## 一、安装麒麟系统



## 二、安装包准备



## 三、环境设置

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

检查selinux状态

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
rpm -ivh jdk-8u152-linux-x64.rpm

vim /etc/profile

JAVA_HOME=/usr/java/jdk1.8.0_152
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

### 3.3、YUM源设置(离线安装需要配置)

#### 1.配置本地Ambari+HDP的yum源

```shell
yum  install yum-utils -y
yum repolist
yum install createrepo -y
```

#### 2.安装httpd服务器

```shell
yum -y install httpd
systemctl restart httpd
systemctl enable httpd
```

#### 3.将准备的HDP安装包放到/var/www/html目录下

```shell
[root@master ~]# cd /var/www/html/
[root@master html]# mkdir ambari
拷贝文件到ambari下面
[root@master html]# cd ambari/
[root@master ambari]# ls
ambari-2.6.0.0-centos7.tar.gz  HDP-2.6.3.0-centos7-rpm.tar.gz  HDP-UTILS-1.1.0.21-centos7.tar.gz
[root@master ambari]# tar -zxvf ambari-2.6.0.0-centos7.tar.gz
[root@master ambari]# tar -zxvf HDP-2.6.3.0-centos7-rpm.tar.gz 
[root@master ambari]# mkdir HDP-UTILS
[root@master ambari]# tar -zxvf HDP-UTILS-1.1.0.21-centos7.tar.gz -C HDP-UTILS
[root@master ambari]# rm -rf ambari-2.6.0.0-centos7.tar.gz HDP-2.6.3.0-centos7-rpm.tar.gz HDP-UTILS-1.1.0.21-centos7.tar.gz 
[root@master ambari]# ls
ambari  HDP  HDP-UTILS
```

```shell
tar -zxvf ambari-2.7.5.0-centos7.tar.gz -C /var/www/html/
tar -zxvf HDP-3.1.5.0-centos7-rpm.tar.gz -C /var/www/html/
tar -zxvf HDP-GPL-3.1.5.0-centos7-gpl.tar.gz -C /var/www/html/
tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/
mkdir /var/www/html/libtrpc
mv libtirpc-0.2.4-0.16.el7.x86_64.rpm /var/www/html/libtrpc
mv libtirpc-devel-0.2.4-0.16.el7.x86_64.rpm /var/www/html/libtrpc
cd /var/www/html/libtrpc
createrepo . 
```



#### 制作yum源

```shell
# 制作本地YUM源
vim /etc/yum.repos.d/ambari.repo

[Ambari-2.7.5.0]
name=Ambari-2.7.5.0
baseurl=http://vm60/ambari/centos7/2.7.5.0-72/
gpgcheck=0
enabled=1
priority=1


vim /etc/yum.repos.d/HDP.repo
[HDP-3.1.5.0]
name=HDP Version - HDP-3.1.5.0
baseurl=http://vm60/HDP/centos7/3.1.5.0-152/
gpgcheck=0
enabled=1
priority=1

[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://vm60/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=0
enabled=1
priority=1

[HDP-GPL-3.1.5.0]
name=HDP-GPL Version - HDP-GPL-3.1.5.0
baseurl=http://vm60/HDP-GPL/centos7/3.1.5.0-152
gpgcheck=0
enabled=1
priority=1

vim /etc/yum.repos.d/libtrpc.repo
[libtirpc_repo]
name=libtirpc-0.2.4-0.16
baseurl=http://vm60/libtrpc/
gpgcheck=0
enabled=1
priority=1


```



## 

#### 4.yum文件分发

```shell
scp  /etc/yum.repos.d/ambari.repo vm61:/etc/yum.repos.d/
scp  /etc/yum.repos.d/HDP.repo vm61:/etc/yum.repos.d/
scp  /etc/yum.repos.d/libtrpc.repo vm61:/etc/yum.repos.d/

scp  /etc/yum.repos.d/ambari.repo vm62:/etc/yum.repos.d/
scp  /etc/yum.repos.d/HDP.repo vm62:/etc/yum.repos.d/
scp  /etc/yum.repos.d/libtrpc.repo vm62:/etc/yum.repos.d/
```



#### 5.更新makecache

```shell
#分别在每台机器上执行：
cd /etc/yum.repos.d/
yum clean all 
yum makecache
```







```shell
# 安装HTTP服务

yum -y install httpd
systemctl restart httpd
systemctl enable httpd
# 将ambari + HDP安装包上传至服务器
tar -zxvf ambari-2.7.5.0-centos7.tar.gz -C /var/www/html/
tar -zxvf HDP-3.1.5.0-centos7-rpm.tar.gz -C /var/www/html/
tar -zxvf HDP-GPL-3.1.5.0-centos7-gpl.tar.gz -C /var/www/html/
tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/
mkdir /var/www/html/libtrpc
mv libtirpc-0.2.4-0.16.el7.x86_64.rpm /var/www/html/libtrpc
mv libtirpc-devel-0.2.4-0.16.el7.x86_64.rpm /var/www/html/libtrpc
cd /var/www/html/libtrpc
createrepo . 


# 制作本地YUM源
vim /etc/yum.repos.d/ambari.repo

[Ambari-2.7.5.0]
name=Ambari-2.7.5.0
baseurl=http://vm60/ambari/centos7/2.7.5.0-72/
gpgcheck=0
enabled=1
priority=1


vim /etc/yum.repos.d/HDP.repo
[HDP-3.1.5.0]
name=HDP Version - HDP-3.1.5.0
baseurl=http://vm60/HDP/centos7/3.1.5.0-152/
gpgcheck=0
enabled=1
priority=1

[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://vm60/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=0
enabled=1
priority=1

[HDP-GPL-3.1.5.0]
name=HDP-GPL Version - HDP-GPL-3.1.5.0
baseurl=http://vm60/HDP-GPL/centos7/3.1.5.0-152
gpgcheck=0
enabled=1
priority=1

vim /etc/yum.repos.d/libtrpc.repo
[libtirpc_repo]
name=libtirpc-0.2.4-0.16
baseurl=http://vm60/libtrpc/
gpgcheck=0
enabled=1
priority=1


scp  /etc/yum.repos.d/ambari.repo vm61:/etc/yum.repos.d/
scp  /etc/yum.repos.d/HDP.repo vm61:/etc/yum.repos.d/
scp  /etc/yum.repos.d/libtrpc.repo vm61:/etc/yum.repos.d/

scp  /etc/yum.repos.d/ambari.repo vm62:/etc/yum.repos.d/
scp  /etc/yum.repos.d/HDP.repo vm62:/etc/yum.repos.d/
scp  /etc/yum.repos.d/libtrpc.repo vm62:/etc/yum.repos.d/

#分别在每台机器上执行：
cd /etc/yum.repos.d/
yum clean all 
yum makecache

```

##  四、安装基础服务（主节点执行）

### 4.1 MySQL安装部署和元数据初始化（主节点执行）

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



#### 3.创建hdp数据库和配置

```shell
mysql -uroot -proot 

use mysql;
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;

create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database monitor DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on *.* to root@"%" Identified by "root";
```



```shell
# 修改root密码
use mysql
update mysql.user set authentication_string=password('Klbr9780') where user='root' and host='%';
flush privileges;
```















### 4.2 时间服务器同步

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



### 



## 五、Ambari-Server部署



#### 5.1、安装ambari-server

```shell
sudo yum install -y ambari-server --installroot=/ztgx-jx/hdp/ambari
```

```shell
cp /ztgx-jx/HDP3.1.5/mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar  /usr/share/java/
cp /ztgx-jx/HDP3.1.5/mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar  /ztgx-jx/hdp/ambari/var/lib/ambari-server/resources

vim /ztgx-jx/hdp/ambari/etc/ambari-server/conf/ambari.properties
server.jdbc.driver.path=/ztgx-jx/HDP3.1.5/mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar 

```



#### 5.2、解决系统上的问题

关键信息来了：yum 安装完server后，需要修改ambari-server的系统检查设置
一共需要修改三处信息:

##### os_check.py

**vim /ztgx-jx/hdp/ambari/usr/lib/ambari-server/lib/ambari_commons/os_check.py**
第一处：在80行左右相似格式的地方插入如下代码(注意格式对齐)

```shell
_IS_KYLIN_LINUX = os.path.exists('/etc/kylin-release')
```

![image-20231221105532920](images/image-20231221105532920.png)

第二处：在90行左右相似格式的地方插入如下代码

```python
def _is_kylin_linux():
    return _IS_KYLIN_LINUX
```

<img src="images/image-20231221105725322.png" alt="image-20231221105725322" style="zoom:50%;" />

第三处：在210行左右相似格式的地方插入如下代码

```python
elif _is_kylin_linux():
	distribution =("centos","7","core")
```

<img src="images/image-20231221105821570.png" alt="image-20231221105821570" style="zoom:50%;" />







#### 5.3、配置ambari-server

```shell
ambari-server setup --jdbc-db=mysql --jdbc-driver=/ztgx-jx/HDP3.1.5/mysql-connector-java-5.1.40/mysql-connector-java-5.1.40-bin.jar 
ambari-server setup
[root@master software]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? 
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
==============================================================================
Enter choice (1): 2
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/java/jdk1.8.0_152
Validating JDK on Ambari Server...done.
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? 
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 3
Hostname (localhost): 
Port (3306): 
Database name (ambari): 
Username (ambari): root
Enter Database Password (bigdata): 
Re-enter password: 
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL directly from the database shell to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
ambari-admin-2.7.5.0.72.jar
....
Ambari repo file doesn't contain latest json url, skipping repoinfos modification
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.

```







#### 5.4、配置MySQL表

mysql -uroot -proot

```sql
create database ambari;
use ambari;
source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;
```

#### 安装ambari-agent (所有节点)

```shell
yum -y install ambari-agent
vim /etc/ambari-agent/conf/ambari-agent.ini

systemctl start ambari-agent
```

![image-20231221115742679](images/image-20231221115742679.png)



#### 5.6、启动ambari-server



## 六、可视化界面配置
