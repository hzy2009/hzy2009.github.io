---
title: docker 安装
date: 2019-11-14 15:01:53
tags:
---

### 安装环境:
* 操作系统 :CentOS Linux release 7.6.1810 (Core)
* CPU 2核
* 内存 8G
* 硬盘 60G

### 安装步骤
本次安装参考docker的官方文档，进行yum安装。文档地址如下：  
https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

1. 卸载旧版本的docker安装，然而机子上本来没有。
2. 安装依赖包
3. 配置安装的仓库
4. 安装docker最新版本
5. 启动docker
6. 检查docker是否启动
7. 执行demo镜像


### 卸载旧版本的docker
```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# sudo yum remove docker \
>                   docker-client \
>                   docker-client-latest \
>                   docker-common \
>                   docker-latest \
>                   docker-latest-logrotate \
>                   docker-logrotate \
>                   docker-engine
已加载插件：fastestmirror
参数 docker 没有匹配

参数 docker-client 没有匹配
参数 docker-client-latest 没有匹配
参数 docker-common 没有匹配
参数 docker-latest 没有匹配
参数 docker-latest-logrotate 没有匹配
参数 docker-logrotate 没有匹配
参数 docker-engine 没有匹配
不删除任何软件包
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 安装依赖包

```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# yum install -y yum-utils \
>   device-mapper-persistent-data \
>   lvm2
已加载插件：fastestmirror
Determining fastest mirrors

。。。。(省略)

已安装:
  device-mapper-persistent-data.x86_64 0:0.8.5-1.el7                             lvm2.x86_64 7:2.02.185-2.el7_7.2                             yum-utils.noarch 0:1.1.31-52.el7

作为依赖被安装:
  device-mapper-event.x86_64 7:1.02.158-2.el7_7.2       device-mapper-event-libs.x86_64 7:1.02.158-2.el7_7.2       libaio.x86_64 0:0.3.109-13.el7            libxml2-python.x86_64 0:2.9.1-6.el7_2.3
  lvm2-libs.x86_64 7:2.02.185-2.el7_7.2                 python-chardet.noarch 0:2.2.1-3.el7                        python-kitchen.noarch 0:1.1.1-5.el7

作为依赖被升级:
  device-mapper.x86_64 7:1.02.158-2.el7_7.2                                                          device-mapper-libs.x86_64 7:1.02.158-2.el7_7.2

完毕！
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 配置安装的仓库

```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
已加载插件：fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 安装docker最新版本

```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# yum install docker-ce docker-ce-cli containerd.io
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
docker-ce-stable                                                                                                                                                                     | 3.5 kB  00:00:00
(1/2): docker-ce-stable/x86_64/updateinfo                                                                                                                                            |   55 B  00:00:00
(2/2): docker-ce-stable/x86_64/primary_db

。。。。(省略)

已安装:
  containerd.io.x86_64 0:1.2.10-3.2.el7                                docker-ce.x86_64 3:19.03.4-3.el7                                docker-ce-cli.x86_64 1:19.03.4-3.el7

作为依赖被安装:
  audit-libs-python.x86_64 0:2.8.5-4.el7         checkpolicy.x86_64 0:2.5-8.el7     container-selinux.noarch 2:2.107-3.el7     libcgroup.x86_64 0:0.41-21.el7     libsemanage-python.x86_64 0:2.5-14.el7
  policycoreutils-python.x86_64 0:2.5-33.el7     python-IPy.noarch 0:0.75-6.el7     setools-libs.x86_64 0:3.3.8-4.el7

作为依赖被升级:
  audit.x86_64 0:2.8.5-4.el7                                     audit-libs.x86_64 0:2.8.5-4.el7                                     policycoreutils.x86_64 0:2.5-33.el7

完毕！
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 启动docker

```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# systemctl start docker
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 检查docker是否启动
```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.4
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
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
 Kernel Version: 3.10.0-957.21.3.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 7.638GiB
 Name: iZ2ze1mqd75ujzoxsvk29lZ
 ID: HU4T:UUU3:J5FR:D7SJ:7GO5:IKVG:X4A7:FJXP:R2OL:ZDZY:IADU:7DK5
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 执行demo镜像
```
[root@iZ2ze1mqd75ujzoxsvk29lZ ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[root@iZ2ze1mqd75ujzoxsvk29lZ ~]#
```

### 结束
