---
title: 'FTP 异常“Entering Passive Mode” '
date: 2019-11-19 18:06:15
tags:
---


FTP连接时出现“227 Entering Passive Mode”解决办法也很简单，只需要关闭客户端的PASV方式，强制其用PORT方式访问服务器即可

## FTP服务的两种工作模式：

* port方式(主动模式)， 连接过程：客户端向服务器的FTP端口(默认是21)发送连接请求，服务器接受连接，建立一条命令链路。当需要传送数据时， 客户端在命令链路上用PORT命令告诉服务器：“我打开了***X端口，你过来连接我”。于是服务器从20端口向客户端的***X端口发送连接请求，建立一条数据链路来传送数据。

* pasv方式 ( 被动模式 )， 连接过程：客户端向服务器的FTP端口(默认是21)发送连接请求，服务器接受连接，建立一条命令链路。当需要传送数据时， 服务器在命令链路上用PASV命令告诉客户端：“我打开了***X端口，你过来连接我”。于是客户端向服务器的***X端口发送连接请求，建立一条数据链 路来传送数据。

由于服务器上的FTP进行TCP/IP筛选，仅允许特定的端口可以被客户端连接，所以无法使用PASV方式。找到了原因，解决办法也很简单，只需要关闭客户端的PASV方式，强制其用PORT方式访问服务器即可。

客户端登录FTP服务器后，用passive命令关闭客户端的PASV方式，如下：

```
ftp> passive 
Passive mode off. 
```
再次执行该命令就可以启用PASV模式。

### 修改配置文件解决问题
1. 修改vsftpd.conf文件，添加: <以被动模式启动，开放被动的访问端口>

```
pasv_min_port=30000
pasv_max_port=30999
```

2. 然后定义iptables规则：
  
```
iptables –A INPUT –p tcp ——dport 30000:30999 –j ACCEPT
iptables –A INPUT –p tcp ——dport 30000:30999 –j ACCEPT
iptables –A INPUT –p tcp ——dport 21 –j ACCEPT
```

3. 将vsftpd的模式修改为主动模式：<以主动模式起动>  
  
```
pasv_enable=no 
```