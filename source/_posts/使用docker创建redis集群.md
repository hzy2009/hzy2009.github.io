---
title: 使用docker创建redis集群
date: 2019-10-23 16:51:05
tags: 技术分享
top: true
---

近期研究docker，同时也有同事在问，如何使用docker搭建redis集群，我自己也实践下，本机环境是：
* 操作系统:macOS 10.15
* docker:18.09.2
* redis:5.0.5

为了能够顺利地使用docker来部署redis集群，我首先本地做redis集群，确保集群的方式是对的，再来到docker上面做redis的集群。

# 本地做Redis集群
本地做redis集群，也是分几个小的步骤，首先配置6个不同redis文件，用于做3个master 3个slaver的集群，然后启动redis server分别启动这个6个redis，最后使用创建集群。
## 配置redis文件
* 首先在任意目录下，创建一个redis配置文件的模板 redis.conf，用于配置redis开启集群模式，代码如下：

```
# 端口
port 7000

后台运行
daemonize yes

# 是否启用redis集群模式
cluster-enabled yes

# 请注意，尽管这个选项的名称，但它不是一个用户可编辑的配置文件，
# 但是Redis集群节点所在的文件在每次发生更改时自动保持集群配
# 置(基本上是状态)，以便能够在启动时重新读取它。该文件列出了
# 集群中的其他节点、它们的状态、持久变量等等。由于接收到一些
# 消息，这个文件常常被重写并刷新到磁盘上。
cluster-config-file nodes.conf

# Redis集群节点不可用的最长时间，而不会被认为是失败的。如果
# 一个主节点的访问时间超过了指定的时间量，则它的从节点将进行
# 故障转移。该参数控制Redis集群中的其他重要内容。值得注意的
# 是，在指定的时间内不能到达大多数主节点的节点将停止接受查询。
cluster-node-timeout 5000

# 开启AOF
appendonly yes
```

* 建立6个redis配置文件夹，分别用6个不同端口号命名，7000-7005，操作如下：
  
```
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
```

* 将模板文件，复制到创建的6个目录下，并且修改模板文件把端口号，改成跟目录一样，把6379，改成7000(获取其他端口)

## 启动redis server
配置文件准备好之后，我们就可以逐一的启动这个redis，类似如下：
```
cd 7000
../redis-server ./redis.conf
```

从每个实例的日志中可以看到，由于不存在nodes.conf文件，每个节点都为自己分配一个新ID，如下。同时会新增文件，appendonly.aof	nodes.conf

```
[82462] 26 Nov 11:56:55.329 * No cluster configuration found, I'm 97a3a64667477371c4479320d683e4c8db5858b1
```

## 执行redis-cli 创建集群
```
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1
```

执行结束后，如果一切配置，在日志的最后，会出现类似日志：
```
[OK] All 16384 slots covered
```

## 执行 redis-cli -c -p 来校验集群是否成功
```
$ redis-cli -c -p 7000
redis 127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
redis 127.0.0.1:7002> set hello world
-> Redirected to slot [866] located at 127.0.0.1:7000
OK
redis 127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
redis 127.0.0.1:7000> get hello
-> Redirected to slot [866] located at 127.0.0.1:7000
"world"
```

# 使用docker做Redis集群
如果上述在本地上创建集群都能成功执行。那么把集群的操作放到docker上面而已。

## 使用docker拉取最新的redis镜像
* 在Docker Hub上查找redis镜像
  
```
hzy@localhost ~ % docker search redis
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that…   7430                [OK]
bitnami/redis                    Bitnami Redis Docker Image                      129                                     [OK]
sameersbn/redis                                                                  77                                      [OK]
grokzen/redis-cluster            Redis cluster 3.0, 3.2, 4.0 & 5.0               61
rediscommander/redis-commander   Alpine image for redis-commander - Redis man…   31                                      [OK]
kubeguide/redis-master           redis-master with "Hello World!"                30
redislabs/redis                  Clustered in-memory database engine compatib…   23
oliver006/redis_exporter          Prometheus Exporter for Redis Metrics. Supp…   18
arm32v7/redis                    Redis is an open source key-value store that…   17
redislabs/redisearch             Redis With the RedisSearch module pre-loaded…   17
webhippie/redis                  Docker images for Redis                         10                                      [OK]
s7anley/redis-sentinel-docker    Redis Sentinel                                  9                                       [OK]
redislabs/redisgraph             A graph database module for Redis               8                                       [OK]
insready/redis-stat              Docker image for the real-time Redis monitor…   8                                       [OK]
bitnami/redis-sentinel           Bitnami Docker Image for Redis Sentinel         8                                       [OK]
arm64v8/redis                    Redis is an open source key-value store that…   6
redislabs/redismod               An automated build of redismod - latest Redi…   5                                       [OK]
centos/redis-32-centos7          Redis in-memory data structure store, used a…   4
circleci/redis                   CircleCI images for Redis                       2                                       [OK]
frodenas/redis                   A Docker Image for Redis                        2                                       [OK]
runnable/redis-stunnel           stunnel to redis provided by linking contain…   1                                       [OK]
tiredofit/redis                  Redis Server w/ Zabbix monitoring and S6 Ove…   1                                       [OK]
wodby/redis                      Redis container image with orchestration        1                                       [OK]
cflondonservices/redis           Docker image for running redis                  0
xetamus/redis-resource           forked redis-resource                           0                                       [OK]
hzy@localhost ~ %
```

* 从Docker Hub 拉取镜像

```
hzy@localhost ~ % docker pull redis
```
* 下载完成后，我们就可以在本地镜像列表里查到redis

```
hzy@localhost ~ % docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              de25a81a5a0b        6 days ago          98.2MB
```

## 创建redis集群的容器网络
我们为redis集群在创建一个容器网络由于集群用。这个网路起名为net2，子网是 172.19.0.0/16。
```
docker network create --driver=bridge --subnet=172.19.0.0/16 net2
```

创建成功后，执行docker network ls
```
hzy@localhost 7000 % docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ea330e03f971        bridge              bridge              local
3ceb6ce41b7d        host                host                local
b56e34603465        net2                bridge              local
e403175c1fa1        none                null                local
```

## 启动6个redis服务器并创建redis集群
* 回到redis.conf的模板文件的所在目录
```
hzy@localhost 7000 % ls
redis.conf
```

* 运行容器，绑定模板文件到容器的/etc/redis/redis.conf，绑定本地的5002端口到7000，分配IP 172.19.0.2 

```
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5002:7000 --net net2 --ip 172.19.0.2 redis:latest bash
9119fb0bc6c64f4b435e0d642a916c4b52d8d23bd0a50d9e22942b1e86968813
hzy@localhost 7000 % docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
9119fb0bc6c6        redis:latest        "docker-entrypoint.s…"   4 seconds ago       Up 3 seconds        6379/tcp, 0.0.0.0:5002->7000/tcp   boring_tharp
```

* 进入到容器，执行命令 docker exec ，在容器内命令行的方式，把redis启动起来
```
hzy@localhost 7000 % docker exec -it  9119fb0bc6c6  bash
root@9119fb0bc6c6:/data#
root@9119fb0bc6c6:/data#
root@9119fb0bc6c6:/data#
root@9119fb0bc6c6:/data# redis-server /etc/redis/redis.conf
14:C 23 Oct 2019 14:54:20.626 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
14:C 23 Oct 2019 14:54:20.626 # Redis version=5.0.6, bits=64, commit=00000000, modified=0, pid=14, just started
14:C 23 Oct 2019 14:54:20.626 # Configuration loaded
root@9119fb0bc6c6:/data#
```

* 同样的方式，启动剩余5个 docker容器
```
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5003:7000 --net net2 --ip 172.19.0.3 redis:latest bash
604f1a0dd0c3ffc51c4e281121aff8c31942677b14ab81c32dff8b541e0691df
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5004:7000 --net net2 --ip 172.19.0.4 redis:latest bash
b082f4dc583ff30373e1b46c0244f64795968d480b55cfcfa961647add794aea
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5005:7000 --net net2 --ip 172.19.0.5 redis:latest bash
0c77f80821cfacd2af485e13fee7ab46174d9252546c9328b378448f3a2ad05c
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5006:7000 --net net2 --ip 172.19.0.6 redis:latest bash
56f1c32d6f938280e6bd9c1179145c1da89881206ba262472222074b45e73b52
hzy@localhost 7000 % docker run -it -d  -v $PWD/redis.conf:/etc/redis/redis.conf  -p 5007:7000 --net net2 --ip 172.19.0.7 redis:latest bash
5b3996c528b7c09dd2ed57e93cbc5eef20fb9157a80a1530799182943e52116a
hzy@localhost 7000 %
hzy@localhost 7000 %
hzy@localhost 7000 %
hzy@localhost 7000 %
hzy@localhost 7000 % docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                              NAMES
5b3996c528b7        redis:latest        "docker-entrypoint.s…"   54 seconds ago       Up 52 seconds       6379/tcp, 0.0.0.0:5007->7000/tcp   dreamy_aryabhata
56f1c32d6f93        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp, 0.0.0.0:5006->7000/tcp   silly_carson
0c77f80821cf        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp, 0.0.0.0:5005->7000/tcp   xenodochial_hawking
b082f4dc583f        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp, 0.0.0.0:5004->7000/tcp   naughty_goldwasser
604f1a0dd0c3        redis:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp, 0.0.0.0:5003->7000/tcp   gracious_nash
9119fb0bc6c6        redis:latest        "docker-entrypoint.s…"   3 minutes ago        Up 3 minutes        6379/tcp, 0.0.0.0:5002->7000/tcp   boring_tharp
```

* 同样的方式，启动redis-server

* 进入到某一个容器，让创建建群,执行以下命令，如果看到输入日志，最终是 “[OK] All 16384 slots covered.” 就是成功了。
```
redis-cli --cluster create \
172.19.0.2:7000 172.19.0.3:7000 \
172.19.0.4:7000 172.19.0.5:7000 \
172.19.0.6:7000 172.19.0.7:7000 \
--cluster-replicas 1
```

## 校验集群是否成功
```
hzy@localhost 7000 % docker exec -it 604f1a0dd0c3 bash
root@604f1a0dd0c3:/data#
root@604f1a0dd0c3:/data#
root@604f1a0dd0c3:/data# redis-cli -c -p 7000
127.0.0.1:7000>
127.0.0.1:7000>
127.0.0.1:7000> get hello
-> Redirected to slot [866] located at 172.19.0.2:7000
"workd"
172.19.0.2:7000>
172.19.0.2:7000>
172.19.0.2:7000>
```

# 总结
根据本次实践，熟悉了redis的集群配置，同时熟悉的docker的简单操作。

