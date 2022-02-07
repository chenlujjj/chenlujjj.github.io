---
layout: single
title:  "ZooKeeper部署和初步使用"
date:   2022-02-07 16:15:00 +0800
categories: zookeeper
tags: [zookeeper]
---

## 部署

这里仅出于学习需要，所以部署一个单机standalone版本即可，不涉及集群。

ZooKeeper官方提供了下载包，不过在MacOS上可以使用homebrew来快速部署。
```
$ brew install zookeeper
...
To start zookeeper now and restart at login:
  brew services start zookeeper
Or, if you don't want/need a background service you can just run:
  zkServer start
...
```
然后执行`zkServer start`来启动服务。
```
$ zkServer start
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Starting zookeeper ... STARTED
```
当然也可能不那么顺利，遇到如下报错：
```
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Starting zookeeper ... FAILED TO START
```
这个时候尝试在前台启动ZooKeeper，可以看到更多的信息用于排查。
```
$ zkServer start-foreground
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Unable to start AdminServer, exiting abnormally
```
噢，原来是AdminServer的问题，搜索后很快可知是端口占用的问题。默认地，AdminServer的端口是8080，在本机上被其他服务使用了。因此修改 `/usr/local/etc/zookeeper/zoo.cfg`，加上这一行：
```
admin.serverPort=8081
```
然后重新启动就好了。

## 使用客户端连接ZooKeeper

### zkCli

使用homebrew安装ZooKeeper后，也包含了其客户端命令行工具 zkCli。
```shell
$ zkCli -server 127.0.0.1:2181
Connecting to 127.0.0.1:2181
Welcome to ZooKeeper!
JLine support is enabled

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[zookeeper]
```
顺利连上了，也初步验证了部署是OK的。

尝试操作ZNode，包括create，set，get和delete。
```sh
[zk: 127.0.0.1:2181(CONNECTED) 2] create /MyFirstZNode ZNodeVal
Created /MyFirstZNode
[zk: 127.0.0.1:2181(CONNECTED) 3] ls /
[MyFirstZNode, zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 4] get /

[zk: 127.0.0.1:2181(CONNECTED) 5] get /zookeeper

[zk: 127.0.0.1:2181(CONNECTED) 6] get  /MyFirstZNode
ZNodeVal
[zk: 127.0.0.1:2181(CONNECTED) 7] get /FirstZnode
org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /FirstZnode
[zk: 127.0.0.1:2181(CONNECTED) 8] set /MyFirstZNode ZNodeValUpdated
[zk: 127.0.0.1:2181(CONNECTED) 9] get /MyFirstZNode
ZNodeValUpdated
[zk: 127.0.0.1:2181(CONNECTED) 14] delete /MyFirstZNode
[zk: 127.0.0.1:2181(CONNECTED) 15] get /MyFirstZNode
org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /MyFirstZNode
```
> Tip：在zkCli的shell里输入help可列出所有命令和用法。

关于zkCli的更多使用方法可参看官方文档[ZooKeeper-cli: the ZooKeeper command line interface](https://zookeeper.apache.org/doc/current/zookeeperCLI.html)

### Java SDK

对着教程[Getting Started with Java and Zookeeper](https://www.baeldung.com/java-zookeeper)抄写了几个使用Java SDK对ZNode做CRUD操作的例子，放在代码库 https://github.com/chenlujjj/zookeeper 了，也可算是重新熟悉了一下Java代码和IDE的使用方式。

根据官方文档[Useful Tools](https://cwiki.apache.org/confluence/display/ZOOKEEPER/UsefulTools)中的指示，[curator](https://github.com/Netflix/curator)和[zkclient](https://github.com/sgroschupf/zkclient)也是值得一看的Java ZooKeeper Client库。