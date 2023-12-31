# 环境搭建

## docker环境部署

### 下载docker 

```sh
# 1.阿里云镜像资源（先执行这个下载加速）
yum-config-manager --add-rep https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#2.安装docker
yum install -y docker-ce
```



### 启动Docker服务

安装完成后，使用下面的命令来启动 docker 服务，并将其设置为开机启动：

```sh
service docker start
chkconfig docker on
```

测试Docker是否安装成功

```sh
docker version
```

输入上述命令，返回docker的版本相关信息，证明docker安装成功。

### 设置国内镜像

```sh
vi  /etc/docker/daemon.json

#添加
{
    "registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
    "live-restore": true
}
```

依次执行以下命令，重新启动 Docker 服务。

```sh
systemctl daemon-reload
service docker restart
```

检查是否生效

```sh
docker info
```

查看是否有如下信息

```sh
Registry Mirrors:
    https://mirror.ccs.tencentyun.com/
```



### Docker Compose的安装 

我们一般都是通过docker compose来安装中间件，所以这个必不可少。可手动下载，直接上传到`/usr/local/bin`

[下载地址](https://github.com/docker/compose/releases/download/1.28.6/docker-compose-Linux-x86_64)，

[百度云下载地址](https://pan.baidu.com/s/1xbK9p9Gz_2qVNgZhU2HseA?pwd=8888) 

将可执行权限应用于二进制文件：

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

测试是否安装成功：

```sh
docker-compose --version
```



### 常用命令 

除过以上我们使用的Docker命令外，Docker还有一些其它常用的命令

**拉取docker镜像**

```sh
docker pull image_name
```

**查看宿主机上的镜像，Docker镜像保存在/var/lib/docker目录下:**

```sh
docker images
```

**删除镜像**

```sh
docker rmi image_name:version
#或者 
docker rmi b39c68b7af30
```

**查看当前有哪些容器正在运行**

```sh
docker ps
```

**查看所有容器**

```sh
docker ps -a
```

**启动、停止、重启容器命令**

```sh
docker start container_name/container_id 
docker stop container_name/container_id 
docker restart container_name/container_id
```

**后台启动一个容器后，如果想进入到这个容器，可以使用attach命令**

```sh
docker attach container_name/container_id
```

**删除容器的命令**

```sh
docker rm container_name/container_id
```

**删除所有停止的容器**

```sh
docker rm $(docker ps -a -q)
```

**查看当前系统Docker信息**

```sh
docker info
```

**从Docker hub上下载某个镜像**

```sh
#执行docker pull centos会将Centos这个仓库下面的所有镜像下载到本地repository。
docker pull centos:latest
```

**查找Docker Hub上的nginx镜像**

```sh
docker search nginx
```



## Mysql部署

`记得开防火墙端口3306`

mysql修改密码：http://www.yuyanba.com/default.aspx/did93214

创建挂载目录

```sh
#创建挂载目录
mkdir -p /data/mysql/data;
mkdir -p /data/mysql/conf;
```

创建yml文件

```
vi /data/mysql/docker-compose.yml
```

填入配置

```yml
version: '3'
services:
  mysql:
    image: mysql:5.7 #mysql版本
    container_name: mysql
    volumes:
      - /data/mysql/data:/var/lib/mysql
      - /data/mysql/conf/my.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf
    restart: always
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: 123 #root用户密码
      TZ: Asia/Shanghai
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

创建配置文件

```sh
vi /data/mysql/conf/my.cnf
```

```sh
[mysqld]
default-storage-engine=INNODB  # 创建新表时将使用的默认存储引擎
character-set-server=utf8mb4      # 设置mysql服务端默认字符集
pid-file        = /var/run/mysqld/mysqld.pid  # pid文件所在目录
socket          = /var/run/mysqld/mysqld.sock # 用于本地连接的socket套接字
datadir         = /var/lib/mysql              # 数据文件存放的目录
symbolic-links=0
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION # 定义mysql应该支持的sql语法，数据校验等!

# 允许最大连接数
max_connections=200


# 同一局域网内注意要唯一
server-id=3306
# 开启二进制日志功能 & 日志位置存放位置`/var/lib/mysql`
#log-bin=mysql-bin
log-bin=/var/lib/mysql/mysql-bin
# binlog格式
# 1. STATEMENT：基于SQL语句的模式，binlog 数据量小，但是某些语句和函数在复制过程可能导致数据不一致甚至出错；
# 2. MIXED：混合模式，根据语句来选用是 STATEMENT 还是 ROW 模式；
# 3. ROW：基于行的模式，记录的是行的完整变化。安全，但 binlog 会比其他两种模式大很多；
binlog_format=ROW
# FULL：binlog记录每一行的完整变更 MINIMAL：只记录影响后的行
binlog_row_image=FULL
# 日志文件大小
max_binlog_size=100M
# 定义清除过期日志的时间(这里设置为7天)
expire_logs_days=7

# ================= ↑↑↑ mysql主从同步配置end ↑↑↑ =================

[mysql]
default-character-set=utf8mb4

[client]
default-character-set=utf8mb4  # 设置mysql客户端默认字符集
```

到对应目录下启动容器

```sh
cd /data/mysql
docker-compose up -d
```

可以测试是否连接成功

![image-20231231135228375](assets/image-20231231135228375.png)

![image-20231231135438154](assets/image-20231231135438154.png)

之后迁移数据到服务器，把conf和data目录复制服务器，使用挂载的方式挂载到mysql容器上

![image-20231231135633100](assets/image-20231231135633100.png)

## Redis部署

创建挂载目录

```sh
#创建挂载目录
mkdir -p /data/redis
```

创建yml文件

```sh
vi /data/redis/docker-compose.yml
```

填入配置

```sh
version: '3'
services:
  redis:
    image: redis:6.2.6
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - /data/redis/redis.conf:/etc/redis/redis.conf
      - /data/redis/data:/data
      - /data/redis/logs:/logs
    command: ["redis-server","/etc/redis/redis.conf"]
```

创建挂载的配置文件

```sh
vi /data/redis/redis.conf
```

```sh
protected-mode no
port 6379
timeout 0
#rdb配置
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /data
appendonly yes
appendfsync everysec
#设置你的redis密码
requirepass 123
```

到对应目录下启动容器

```sh
cd /data/redis
docker-compose up -d
#如果需要强制重新构建
docker-compose up --force-recreate -d
```

`记得开防火墙端口6379`



## rocketMQ部署

参考：[链接](https://blog.csdn.net/oschina_41731918/article/details/123115102),[帐号密码](https://www.cnblogs.com/binz/p/15252277.html)，[官方文档](https://github.com/apache/rocketmq/blob/master/docs/cn/acl/user_guide.md)

一般建议设置密码

### 不配置密码 

创建挂载目录

```sh
#创建挂载目录
mkdir -p /data/rocketmq/namesrv ;
mkdir -p /data/rocketmq/broker/conf ;
```

创建yml文件

```sh
vi /data/rocketmq/docker-compose.yml
```

填入配置

```sh
version: '3.5'
services:
  rocketmq-namesrv:
    image: foxiswho/rocketmq:4.8.0
    container_name: rocketmq-namesrv
    restart: always
    ports:
      - 9876:9876
    volumes:
      - ./namesrv/logs:/home/rocketmq/logs
      - ./namesrv/store:/home/rocketmq/store
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128M -Xmx128M -Xmn128m"
    command: ["sh","mqnamesrv"]
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-namesrv


  rocketmq-broker:
    image: foxiswho/rocketmq:4.8.0
    container_name: rocketmq-broker
    restart: always
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./broker/logs:/home/rocketmq/logs
      - ./broker/store:/home/rocketmq/store
      - ./broker/conf/broker.conf:/etc/rocketmq/broker.conf
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128m -Xmx128m -Xmn128m"
    command: ["sh","mqbroker","-c","/etc/rocketmq/broker.conf"]
    depends_on:
      - rocketmq-namesrv
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-broker


  rocketmq-console:
    image: iamverygood/rocketmq-console:4.7.1
    container_name: rocketmq-console
    restart: always
    ports:
      - 8180:8080
    environment:
      JAVA_OPTS: "-Drocketmq.namesrv.addr=rocketmq-namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false"
    depends_on:
      - rocketmq-namesrv
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-console

networks:
  rocketmq_net:
    name: rocketmq_net
    driver: bridge
```

创建挂载的配置文件

```sh
vim /data/rocketmq/broker/conf/broker.conf
```

```sh

#所属集群名字
brokerClusterName=DefaultCluster

#broker名字，注意此处不同的配置文件填写的不一样，如果在broker-a.properties使用:broker-a,
#在broker-b.properties使用:broker-b
brokerName=broker-a

#0 表示Master，>0 表示Slave
brokerId=0

#nameServer地址，分号分割
#namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
namesrvAddr=rocketmq-namesrv:9876

#启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
brokerIP1=192.168.253.128

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭 ！！！这里仔细看是false，false，false
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=10911

#此参数控制是否开启密码,不开启可设置false
aclEnable=false

#删除文件时间点，默认凌晨4点
deleteWhen=04

#文件保留时间，默认48小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
#storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
#commitLog 存储路径
#storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
#消费队列存储
#storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
#消息索引存储路径
#storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
#checkpoint 文件存储路径
#storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
#abort 文件存储路径
#abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
#限制的消息大小
maxMessageSize=65536

#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER

#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

brokerIP1=192.168.253.128填上自己服务器公网ip，客户端发送消息要联这个

授权目录

```sh
#目录权限（不加会有坑，rocketmq没法操作挂载目录）：
chmod -R 777 /data/rocketmq/namesrv/;
chmod -R 777 /data/rocketmq/broker/;
```



### 设置密码 

创建挂载目录,一次性执行

```sh
#创建挂载目录
mkdir -p /data/rocketmq/namesrv;
mkdir -p /data/rocketmq/broker/conf;
mkdir -p /data/rocketmq/broker/lib;
mkdir -p /data/rocketmq/console/data;
```

创建挂载的配置文件

```sh
vi /data/rocketmq/broker/conf/broker.conf
```

```sh

#所属集群名字
brokerClusterName=DefaultCluster

#broker名字，注意此处不同的配置文件填写的不一样，如果在broker-a.properties使用:broker-a,
#在broker-b.properties使用:broker-b
brokerName=broker-a

#0 表示Master，>0 表示Slave
brokerId=0

#nameServer地址，分号分割
#namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
namesrvAddr=rocketmq-namesrv:9876

#启动IP,如果 docker 报 com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
# 解决方式1 加上一句producer.setVipChannelEnabled(false);，解决方式2 brokerIP1 设置宿主机IP，不要使用docker 内部IP
brokerIP1=192.168.253.128

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭 ！！！这里仔细看是false，false，false
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=10911

#此参数控制是否开启密码,不开启可设置false
aclEnable=true

#删除文件时间点，默认凌晨4点
deleteWhen=04

#文件保留时间，默认48小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000

#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
#storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
#commitLog 存储路径
#storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
#消费队列存储
#storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
#消息索引存储路径
#storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
#checkpoint 文件存储路径
#storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
#abort 文件存储路径
#abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
#限制的消息大小
maxMessageSize=65536

#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER

#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

brokerIP1=192.168.253.128填上自己服务器公网ip，客户端发送消息要联这个

**如果有设置密码的需求，先给broker.conf开启acl密码配置 true** 

![image-20231231141013708](assets/image-20231231141013708.png)

**创建acl文件，用于开启用户名密码** 

```sh
vi /data/rocketmq/broker/conf/plain_acl.yml
```

```sh
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
# 全局白名单，如果配置了则不需要走acl校验，慎重配置
globalWhiteRemoteAddresses:
#  - 47.100.93.*
#  - 156.254.120.*
 
accounts:
  - accessKey: RocketMQ
    secretKey: 12345678
    whiteRemoteAddress:
    admin: false
    defaultTopicPerm: DENY
    defaultGroupPerm: SUB
    topicPerms:
      - topicA=DENY
      - topicB=PUB|SUB
      - topicC=SUB
    groupPerms:
      # the group should convert to retry topic
      - groupA=DENY
      - groupB=PUB|SUB
      - groupC=SUB

  - accessKey: luochat #用户名
    secretKey: 12345678 #密码
    whiteRemoteAddress: 
    # if it is admin, it could access all resources 上面的用于教学，我们用超级管理员账号
    admin: true #管理员权限
```

密码不能小于6位数，血的教训

名称也不能小于6位，血的教训！！！

权限的描述可参考[链接](https://github.com/apache/rocketmq/blob/master/docs/cn/acl/user_guide.md)



**给console加上账号密码** 

```sh
vi /data/rocketmq/console/data/users.properties
```

```sh
# This file supports hot change, any change will be auto-reloaded without Console restarting.
# Format: a user per line, username=password[,N] #N is optional, 0 (Normal User); 1 (Admin)

# Define Admin
# =============用户名和密码规则「用户名=密码,权限」，这里的权限为1表示管理员>，为0表示普通用户=============
# 例如：admin=admin123,1
luochat=123456,1


# Define Users
# =============屏蔽下边两个账户=============
#user1=user1
#user2=user2
```

**创建yml文件** 

```sh
vi /data/rocketmq/docker-compose.yml
```

填入配置

```sh
version: '3.5'
services:
  rocketmq-namesrv:
    image: foxiswho/rocketmq:4.8.0
    container_name: rocketmq-namesrv
    restart: always
    ports:
      - 9876:9876
    volumes:
      - ./namesrv/logs:/home/rocketmq/logs
      - ./namesrv/store:/home/rocketmq/store
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128M -Xmx128M -Xmn128m"
    command: ["sh","mqnamesrv"]
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-namesrv


  rocketmq-broker:
    image: foxiswho/rocketmq:4.8.0
    container_name: rocketmq-broker
    restart: always
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - ./broker/logs:/home/rocketmq/logs
      - ./broker/store:/home/rocketmq/store
      - ./broker/conf/plain_acl.yml:/home/rocketmq/rocketmq-4.8.0/conf/plain_acl.yml
      - ./broker/conf/broker.conf:/etc/rocketmq/broker.conf
    environment:
      JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128m -Xmx128m -Xmn128m"
    command: ["sh","mqbroker","-c","/etc/rocketmq/broker.conf"]
    depends_on:
      - rocketmq-namesrv
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-broker


  rocketmq-console:
    image: iamverygood/rocketmq-console:4.7.1
    container_name: rocketmq-console
    restart: always
    ports:
      - 8180:8080
    volumes:
      - ./console/data:/tmp/rocketmq-console/data
    environment:
      JAVA_OPTS: "-Drocketmq.namesrv.addr=rocketmq-namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false -Drocketmq.config.loginRequired=true -Drocketmq.config.aclEnabled=true -Drocketmq.config.accessKey=luochat -Drocketmq.config.secretKey=12345678"
    depends_on:
      - rocketmq-namesrv
    networks:
      rocketmq_net:
        aliases:
          - rocketmq-console

networks:
  rocketmq_net:
    name: rocketmq_net
    driver: bridge
```

如果你的acl密码改了，记得把yml的console帐号密码也一同更改

-Drocketmq.config.accessKey=luochat -Drocketmq.config.secretKey=12345678

授予目录权限

```sh
#目录权限：
chmod -R 777 /data/rocketmq/namesrv/;
chmod -R 777 /data/rocketmq/broker/;
chmod -R 777 /data/rocketmq/console/;
```

`记得防火墙开端口号 9876,10911,8180 !!!`

### 启动容器 

到对应目录下启动容器

```sh
cd /data/rocketmq
docker-compose up -d
```

docker ps，可以看见一直在重启中

![image-20231231143929477](assets/image-20231231143929477.png)

注意，第一次会启动不成功，因为broker需要创建一堆文件，没有权限。再执行一遍权限命令

```sh
#目录权限：
chmod -R 777 /data/rocketmq/namesrv/;
chmod -R 777 /data/rocketmq/broker/;
chmod -R 777 /data/rocketmq/console/;
#然后强制重新构建
docker-compose up --force-recreate -d
```

![image-20231231144028248](assets/image-20231231144028248.png)

安装过程中踩过一个坑，感兴趣可以看看

[RocketMQ部署失败排查](https://www.yuque.com/snab/mallchat/tt9mcnc6t0g125ow)



**docker-compose版本升级** 

可能有的小伙伴启动会报错。是因为我们的docker-compose版本太低了，识别不了networks命令需要更新一下。速度比较慢，等等就好

```sh
#更新版本
curl -L https://github.com/docker/compose/releases/download/1.28.6/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
#将可执行权限应用于二进制文件
sudo chmod +x /usr/local/bin/docker-compose
```

**登录控制台** 

启动后可以登录控制台，记得开防火墙！！！

http://ip:8180/

![image-20231231144509242](assets/image-20231231144509242.png)

输入帐号密码 luochat123456

![image-20231231144526836](assets/image-20231231144526836.png)

修改后端环境配置

![image-20231231145255314](assets/image-20231231145255314.png)

测试生产者

```java
@Autowired
private RocketMQTemplate rocketMQTemplate;
@Test
public void sendMQ() {
    Message<String> build = MessageBuilder.withPayload("123").build();
    rocketMQTemplate.send("test-topic", build);
}
```

![image-20231231150052398](assets/image-20231231150052398.png)

![image-20231231145941573](assets/image-20231231145941573.png)

![image-20231231150121619](assets/image-20231231150121619.png)

测试消费者

```java
import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQListener;
import org.springframework.stereotype.Component;

@RocketMQMessageListener(consumerGroup = "test-group", topic = "test-topic")
@Component
public class TestConsumer implements RocketMQListener<String> {

    @Override
    public void onMessage(String dto) {
        System.out.println("收到消息{}"+dto);
    }
}

```

![image-20231231150416860](assets/image-20231231150416860.png)

消费者也注册上了test-topic![image-20231231150539702](assets/image-20231231150539702.png)

手动发消息

![image-20231231150728254](assets/image-20231231150728254.png)

![image-20231231150811147](assets/image-20231231150811147.png)



## minio部署

学习参考文档https://java.isture.com/arch/minio/minio-concept.html

比较完整https://blog.csdn.net/weixin_39060009/article/details/115520696

创建挂载目录

```sh
#创建挂载目录
mkdir -p /data/minio
```

创建yml文件

```sh
vi /data/minio/docker-compose.yml
```

填入配置

```sh
version: '3'
services:
  minio:
    image: "quay.io/minio/minio:RELEASE.2022-08-02T23-59-16Z"
    container_name: minio
    ports:
      - "9000:9000" # api 端口
      - "9001:9001" # 控制台端口
    environment:
      TZ: Asia/Shanghai # 时区上海
      MINIO_ROOT_USER: admin # 管理后台用户名
      MINIO_ROOT_PASSWORD: 12345678 # 管理后台密码，最小8个字符
      MINIO_SERVER_URL: "http://192.168.253.128:9000" # 指定分享的域名
      MINIO_COMPRESS: "off" # 开启压缩 on 开启 off 关闭
      MINIO_COMPRESS_EXTENSIONS: "" # 扩展名 .pdf,.doc 为空 所有类型均压缩
      MINIO_COMPRESS_MIME_TYPES: "" # mime 类型 application/pdf 为空 所有类型均压缩
    volumes:
      - /data/minio/data:/data/ # 映射当前目录下的data目录至容器内/data目录      
      - /data/minio/config:/root/.minio/ # 映射配置目录
    command: server --address ':9000' --console-address ':9001' /data  # 指定容器中的目录 /data
    privileged: true
```

有些非大陆服务器,，会出现登录不上，密码不对invalid Login的错误提示，改用下面的配置重新构建

```
docker-compose up --force-recreate -d
```

```sh
version: '3'
services:
  minio:
    image: "quay.io/minio/minio:RELEASE.2022-08-02T23-59-16Z"
    container_name: minio
    ports:
      - "9000:9000" # api 端口
      - "9001:9001" # 控制台端口
    environment:
      MINIO_ROOT_USER: admin # 管理后台用户名
      MINIO_ROOT_PASSWORD: 12345678 # 管理后台密码，最小8个字符
      MINIO_COMPRESS: "off" # 开启压缩 on 开启 off 关闭
      MINIO_COMPRESS_EXTENSIONS: "" # 扩展名 .pdf,.doc 为空 所有类型均压缩
      MINIO_COMPRESS_MIME_TYPES: "" # mime 类型 application/pdf 为空 所有类型均压缩
    volumes:
      - /data/minio/data:/data/ # 映射当前目录下的data目录至容器内/data目录      
      - /data/minio/config:/root/.minio/ # 映射配置目录
    command: server --address ':9000' --console-address ':9001' /data  # 指定容器中的目录 /data
    privileged: true
```

到对应目录下启动容器

```sh
cd /data/minio
docker-compose up -d
#如果需要强制重新构建
docker-compose up --force-recreate -d
```

打开对应的控制台

[http://ip:9001/](http://192.168.253.128:9001/)

![image-20231231152028901](assets/image-20231231152028901.png)

`记得防火墙开端口号 9000,9001  !!!，输入帐号密码admin，12345678`



创建一个测试捅，尝试上传一张图片。

![image-20231231152255341](assets/image-20231231152255341.png)

![image-20231231152358487](assets/image-20231231152358487.png)

![image-20231231152432021](assets/image-20231231152432021.png)

![image-20231231152547568](assets/image-20231231152547568.png)

![image-20231231152559911](assets/image-20231231152559911.png)

![image-20231231152843367](assets/image-20231231152843367.png)

![image-20231231152816710](assets/image-20231231152816710.png)

设置桶为公开桶

![image-20231231153132515](assets/image-20231231153132515.png)

![image-20231231153152467](assets/image-20231231153152467.png)

![image-20231231153225078](assets/image-20231231153225078.png)



url替换成自己的服务器ip，删除后面的权限信息。尝试访问（不成功也没关系）

创建一个权限用户，获取密钥



会随机生成accessKey secretKey，点击create按钮后. 会将这两个key进行展示，此时可以复制粘贴到一个文本文件上，后续使用代码上传文件时需要用到这两个key

在配置文件中配置如下信息



endpoint就是你的minio部署的ip地址。



配置nginx的转发，一次性配置规范点，为对象存储搞个二级域名和https。

端口重定向增加minio.luochat.cn

```sh
server {
    listen       80;
    server_name  api.luochat.cn minio.luochat.cn;
    #将请求转成https
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

增加一个二级域名的监听

```sh
server {
    listen       443 ssl;
    server_name  minio.luochat.cn;
    #频率控制
    limit_req zone=one burst=10 nodelay;

    ssl_certificate      ../cert/minio.mallchat.cn.pem;
    ssl_certificate_key  ../cert/minio.mallchat.cn.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    location / {
        client_body_buffer_size 10M;
        client_max_body_size 10G;
        proxy_buffers 1024 4k;
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

证书放在指定位置