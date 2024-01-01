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





# WebSocket模块

不仅仅是IM通讯系统，在很多业务中都会有服务端需要主动推送web的场景。比如小红点提醒，新消息提醒，审批流提醒等。那么常见的推送方案有哪些？

## 服务端推送web方案 

### 短轮询 

短轮询，就是web端不停地间隔一段时间向服务端发一个 HTTP 请求，如果有新消息，就会在某次请求返回。

![image-20231231200014476](assets/image-20231231200014476.png)



比如OA系统，用户需要收到小红点，审批流提醒等信息，为了方便，就直接采用每秒1次的请求，等待后端返回数据。

**适用场景：**

- 扫码登录：短时间内频繁查询二维码状态

- 小OA系统：客户端使用量不大的情况下可以使用

**缺点：**

- 大量无效请求：大量的无效请求，浪费服务器资源

- 服务端请求压力大：万人群聊频繁访问，上万并发服务扛不住。



### 长轮询 

长轮询和短轮询相比，一个最大的改进之处在于：

- 短轮询模式下，服务端不管本轮有没有新消息产生，都会马上响应并返回。而长轮询模式当本次请求没有获取到新消息时，并不会马上结束返回，而是会在服务端“悬挂（hang）”，等待一段时间；

- 如果在等待的这段时间内有新消息产生，就能马上响应返回。

这也意味着web端的请求超时时长得设置长一些。

![image-20231231200711505](assets/image-20231231200711505.png)



**优点：相比短轮询模式**

- 大幅降低短轮询模式中客户端高频无用的轮询导致的网络开销和功耗开销

- 降低了服务端处理请求的 QPS

**缺点：**

- 无效请求：长轮询在超时时间内没有获取到消息时，会结束返回，因此仍然没有完全解决客户端“无效”请求的问题。

- 服务端压力大：服务端悬挂（hang）住请求，只是降低了入口请求的 QPS，并没有减少对后端资源轮询的压力。假如有 1000 个请求在等待消息，可能意味着有 1000 个线程在不断轮询消息存储资源。（轮询转移到了后端）



### Websocket长连接 

长轮询和短轮询是通过服务端被动地接收客户端发起的询问请求，从而达到服务端向客户端推送的一种曲线救国的方式，那最好的方案就是服务端直接向客户端推送，因此诞生了websocket。

实现原理：客户端和服务器之间维持一个 TCP/IP 长连接，全双工通道。

![image-20231231201235249](assets/image-20231231201235249.png)

基本弥补了上面的缺点，唯一的缺点就是实现起来可能会有些复杂，我们需要去管理链接。



### websocket代码实现方案 

支持websocket的容器很多，我们实现一般用两种常见方案。



#### tomcat实现websocket 

原理和使用细节可以查看https://blog.csdn.net/devcloud/article/details/124681914



#### netty实现websocket 

用netty实现websocket可以看项目代码，或者参考文章：https://blog.csdn.net/mahao25/article/details/127418543



#### 为什么选netty不用tomcat？ 

- netty是nio基于事件驱动的多路复用框架，使用单线程或少量线程处理大量的并发连接。相比之下，Tomcat 是基于多线程的架构，每个连接都会分配一个线程，适用于处理相对较少的并发连接。最近的 Tomcat 版本（如 Tomcat 8、9）引入了 NIO（New I/O）模型。所以这个点并不是重点。

- Netty 提供了丰富的功能和组件，可以灵活地构建自定义的网络应用。它具有强大的编解码器和处理器，可以轻松处理复杂的协议和数据格式。Netty 的扩展性也非常好，可以根据需要添加自定义的组件。比如我们可以用netty的pipeline方便的进行前置后置的处理，可以用netty的心跳处理器来检查连接的状态。这些都是netty的优势。



### websocket的连接过程 

客户端依靠发起HTTP握手，告诉服务端进行WebSocket协议通讯，并告知WebSocket协议版本。服务端确认协议版本，升级为WebSocket协议。之后如果有数据需要推送，会主动推送给客户端。 

![image-20231231202611491](assets/image-20231231202611491.png)



连接开始时，客户端使用HTTP协议和服务端升级协议，升级完成后，后续数据交换遵循WebSocket协议。我们看看Request Headers

![image-20231231202926986](assets/image-20231231202926986.png)



其中关键的字段Upgrade,Connection就是告诉 Apache 、 Nginx 等服务器：注意啦，我要升级成Websocket协议，不再使用原先的HTTP。

其中，Sec-WebSocket-Key当成是请求id就好了。



![image-20231231203120912](assets/image-20231231203120912.png)

Sec-WebSocket-Accept: 用来告知服务器愿意发起一个websocket连接， 值是根据客户端请求头的Sec-WebSocket-Key计算出来。

## 项目搭建和多环境配置

### 创建父模块

![image-20231231204336100](assets/image-20231231204336100.png)

### 创建子模块

![image-20231231204818582](assets/image-20231231204818582.png)

![image-20231231205043330](assets/image-20231231205043330.png)

![image-20231231205306504](assets/image-20231231205306504.png)

对子模块进行版本管理

![image-20231231205956974](assets/image-20231231205956974.png)

luochat-chat-server引入common包

![image-20231231210049693](assets/image-20231231210049693.png)

### 编写配置

`luochat-backend的pom`

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.7</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncodi
    <java.version>1.8</java.version>
    <skipTests>true</skipTests>
    <docker.host>http://192.168.3.101:2375</docker.host>
    <hutool.version>5.8.18</hutool.version>
    <springfox-swagger.version>3.0.0</springfox-swagger.version>
    <swagger-models.version>1.6.0</swagger-models.version>
    <mybatis-plus-generator.version>3.4.1</mybatis-plus-generator.version>
    <mybatis.version>3.5.10</mybatis.version>
    <mysql-connector.version>8.0.29</mysql-connector.version>
    <spring-data-commons.version>2.7.5</spring-data-commons.version>
    <jjwt.version>0.9.1</jjwt.version>
    <logstash-logback.version>7.2</logstash-logback.version>
    <minio.version>8.4.5</minio.version>
    <jaxb-api.version>2.3.1</jaxb-api.version>
    <lombok.version>1.18.10</lombok.version>
    <netty-all.version>4.1.76.Final</netty-all.version>
    <weixin-java-mp.version>4.4.0</weixin-java-mp.version>
    <mybatis-plus-boot-starter.version>3.4.0</mybatis-plus-boot-starter.ver
    <jsoup.version>1.15.3</jsoup.version>
    <okhttp.version>4.8.1</okhttp.version>
    <redisson-spring-boot-starter.version>3.17.1</redisson-spring-boot-star
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.luoying</groupId>
            <artifactId>luochat-common-starter</artifactId>
            <version>${version}</version>
        </dependency>
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>${okhttp.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>${jsoup.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus-boot-starter.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <dependency>
            <groupId>com.github.binarywang</groupId>
            <artifactId>weixin-java-mp</artifactId>
            <version>${weixin-java-mp.version}</version>
        </dependency>
        <!-- netty -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>${netty-all.version}</version>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>${hutool.version}</version>
        </dependency>
        <!-- MyBatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>${mybatis-plus-generator.version}</version>
        </dependency>
        <!--Mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector.version}</version>
        </dependency>
        <!--JWT(Json Web Token)登录支持-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <!-- 阿里云OSS -->
        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>${minio.version}</version>
        </dependency>
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson-spring-boot-starter</artifactId>
            <version>${redisson-spring-boot-starter.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

`luochat-common-starter的pom`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
    </dependency>
    <!-- MyBatis-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.velocity</groupId>
        <artifactId>velocity-engine-core</artifactId>
        <version>2.0</version>
    </dependency>
    <!--Mysql数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!-- netty -->
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <!--使用Swagger2-->
        <artifactId>knife4j-spring-boot-starter</artifactId>
        <version>2.0.9</version>
    </dependency>    
</dependencies>
```

`luochat-chat-server的application.yml`

```yml
spring:
  profiles:
    #运行的环境
    active: test
  application:
    name: luochat
  datasource:
    url: jdbc:mysql://${luochat.mysql.ip}:${luochat.mysql.port}/${luochat.mysql.db}?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: ${luochat.mysql.username}
    password: ${luochat.mysql.password}
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    # Redis服务器地址
    host: ${luochat.redis.host}
    # Redis服务器端口号
    port: ${luochat.redis.port}
    # 使用的数据库索引，默认是0
    database: 0
    # 连接超时时间
    timeout: 1800000
    # 设置密码
    password: ${luochat.redis.password}
  jackson:
    serialization:
      write-dates-as-timestamps: true
```

`luochat-chat-server的application-test.properties`

```properties
##################mysql配置##################
luochat.mysql.ip=192.168.253.128
luochat.mysql.port=3306
luochat.mysql.db=luochat
luochat.mysql.username=root
luochat.mysql.password=123
##################redis配置##################
luochat.redis.host=192.168.253.128
luochat.redis.port=6379
luochat.redis.password=123
```



## netty实现websocket 

### 编码

![image-20240101193214983](assets/image-20240101193214983.png)

**NettyWebSocketServer**

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.handler.stream.ChunkedWriteHandler;
import io.netty.handler.timeout.IdleStateHandler;
import io.netty.util.NettyRuntime;
import io.netty.util.concurrent.Future;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @Author 落樱的悔恨
 * @Date 2024/1/1 12:37
 */
@Slf4j
@Configuration
public class NettyWebSocketServer {
    public static final int WEB_SOCKET_PORT = 8090;
    public static final NettyWebSocketServerHandler NETTY_WEB_SOCKET_SERVER_HANDLER = new NettyWebSocketServerHandler();
    // 创建线程池执行器
    private EventLoopGroup bossGroup = new NioEventLoopGroup(1);
    private EventLoopGroup workerGroup = new NioEventLoopGroup(NettyRuntime.availableProcessors());

    /**
     * 启动 ws server
     *
     * @return
     * @throws InterruptedException
     */
    @PostConstruct
    public void start() throws InterruptedException {
        run();
    }

    /**
     * 销毁
     */
    @PreDestroy
    public void destroy() {
        Future<?> future = bossGroup.shutdownGracefully();
        Future<?> future1 = workerGroup.shutdownGracefully();
        future.syncUninterruptibly();
        future1.syncUninterruptibly();
        log.info("关闭 ws server 成功");
    }

    public void run() throws InterruptedException {
        // 服务器启动引导对象
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 128)
                .option(ChannelOption.SO_KEEPALIVE, true)
                .handler(new LoggingHandler(LogLevel.INFO)) // 为 bossGroup 添加 日志处理器
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        ChannelPipeline pipeline = socketChannel.pipeline();
                        //30秒客户端没有向服务器发送心跳则关闭连接
                        // pipeline.addLast(new IdleStateHandler(30, 0, 0));
                        // 因为使用http协议，所以需要使用http的编码器，解码器
                        pipeline.addLast(new HttpServerCodec());
                        // 以块方式写，添加 chunkedWriter 处理器
                        pipeline.addLast(new ChunkedWriteHandler());
                        /**
                         * 说明：
                         *  1. http数据在传输过程中是分段的，HttpObjectAggregator可以把多个段聚合起来；
                         *  2. 这就是为什么当浏览器发送大量数据时，就会发出多次 http请求的原因
                         */
                        pipeline.addLast(new HttpObjectAggregator(8192));
                        //保存用户ip
                        // pipeline.addLast(new HttpHeadersHandler());
                        /**
                         * 说明：
                         *  1. 对于 WebSocket，它的数据是以帧frame 的形式传递的；
                         *  2. 可以看到 WebSocketFrame 下面有6个子类
                         *  3. 浏览器发送请求时： ws://localhost:7000/hello 表示请求的uri
                         *  4. WebSocketServerProtocolHandler 核心功能是把 http协议升级为 ws 协议，保持长连接；
                         *      是通过一个状态码 101 来切换的
                         */
                        pipeline.addLast(new WebSocketServerProtocolHandler("/"));
                        // 自定义handler ，处理业务逻辑
                        pipeline.addLast(NETTY_WEB_SOCKET_SERVER_HANDLER);
                    }
                });
        // 启动服务器，监听端口，阻塞直到启动成功
        serverBootstrap.bind(WEB_SOCKET_PORT).sync();
    }

}
```

明白了websocket的升级过程，对netty的处理的就比较简单了。websocket初期是通过http请求，进行升级，建立双方的连接。

1.所以编解码器需要用到`HttpServerCodec`

2.`WebSocketServerProtocolHandler`是netty进行websocket升级的处理器。在这期间会抹除http相关的信息，比如请求头啥的。如果想获取相关信息，需要在这之前获取。

3.`HttpHeadersHandler`是我们自己的处理器。赶在websocket升级之前，获取用户的ip地址，然后保存到channel的附件里。

4.`NettyWebSocketServerHandler`是我们的业务处理器，里面处理客户端的事件。

5.`IdleStateHandler`实现心跳检测。

**NettyWebSocketServerHandler**

```java
import io.netty.channel.ChannelHandler.Sharable;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import lombok.extern.slf4j.Slf4j;

/**
 * @Author 落樱的悔恨
 * @Date 2024/1/1 12:37
 */
@Slf4j
@Sharable
public class NettyWebSocketServerHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        String text = msg.text();
        System.out.println(text);
    }
}
```

除了`NettyWebSocketServerHandler`是无状态的（必须要加上`@Sharable`，这是netty提供的一个标识，代表所有的pipeline可以共用它，不然后台没有检测到会报错），其他处理器都是有状态的（也就是这些处理器是不能共用的）

**LuoChatCustomApplication**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

/**
 * @Author 落樱的悔恨
 * @Date 2024/1/1 12:37
 */
@SpringBootApplication(scanBasePackages = {"com.luoying.luochat"})
@ServletComponentScan
public class LuoChatCustomApplication {

    public static void main(String[] args) {
        SpringApplication.run(LuoChatCustomApplication.class,args);
    }

}
```

启动主类后，使用apipost来测试

![image-20240101195332817](assets/image-20240101195332817.png)

![image-20240101200150766](assets/image-20240101200150766.png)

**流程：**

发起连接时，会向netty服务器发起http请求，服务器的initChannel方法会对对http请求进行一个协议的转换，转换成WebSocket协议，然后本地的业务处理器就可以处理请求



### 原理

点进`WebSocketServerProtocolHandler`的源码

![image-20240101201517463](assets/image-20240101201517463.png)

点进`WebSocketServerProtocolHandshakeHandler`

![image-20240101201834457](assets/image-20240101201834457.png)

![image-20240101204003449](assets/image-20240101204003449.png)

![image-20240101204053952](assets/image-20240101204053952.png)



## websocket前后端交互协议

### 协议

我们用websocket的目的，主要是用于后端推送前端，前端能用http的就尽量用http。这样的好处是，http丰富的拦截器，注解，请求头等功能，可以更好地实现或者是收口我们想要的功能。尽量对websocket的依赖降到最低。

前后端的交互用的是json串，里面通过type标识次此次的事件类型。

**前端请求示例**

```json
{
type:1,//1.请求登录二维码，2.心跳检测 3.用户认证
data:{}
}
```

- 请求登录二维码

发送type=1从后端请求一个登录二维码

- 心跳包

前端连接websocket后，需要**10**s发送一次心跳包消息。

- 用户认证

用户在刷新后，连接会断开。前端拿着本地存储的token来对连接进行认证，证明这个连接的所有者是个已登录用户

```json
{
type:3,
data:{
    token:asdajsfhjda//用户的登录凭证，每次请求携带，不需要加Bearer前缀
    }
}
```

**后端返回示例**（既有主动返回，也有被动返回）

```json
{
  type:1//1.登录返回二维码 2.用户扫描成功等待授权 3.用户登录成功返回用户信息 4.收到消息 5.上下线推送6.前端token失效 7.拉黑用户（隐藏它的所有消息）
  data:jsondata//根据不同的类型有不同的返回对象
}

```

### 编码

![image-20240101215901550](assets/image-20240101215901550.png)

**编写基本的websocket请求，响应类**

```java
import lombok.Data;

/**
 * @Author 落樱的悔恨
 * @Date 2024/1/1 21:24
 */
@Data
public class WSBaseReq {
    /**
     * @see com.luoying.luochat.common.websocket.domain.enums.WSReqTypeEnum
     */
    private Integer type;

    private String data;
}
```

```java
import lombok.Data;

/**
 * @Author 落樱的悔恨
 * @Date 2024/1/1 21:30
 */
@Data
public class WSBaseResp<T> {
    /**
     * @see com.luoying.luochat.common.websocket.domain.enums.WSRespTypeEnum
     */
    private Integer type;

    private T data;
}
```

![image-20240101220343687](assets/image-20240101220343687.png)

**复制原项目的`WSReqTypeEnum`，`WSRespTypeEnum`**

![image-20240101220427861](assets/image-20240101220427861.png)

**复制原项目提供的后端返回类**

![image-20240101220710452](assets/image-20240101220710452.png)

**业务处理器中添加业务逻辑**

![image-20240101221344459](assets/image-20240101221344459.png)

![image-20240101221434141](assets/image-20240101221434141.png)

![image-20240101221458226](assets/image-20240101221458226.png)

![image-20240101221512771](assets/image-20240101221512771.png)