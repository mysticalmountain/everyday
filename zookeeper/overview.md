[TOC]
# 概述
为分布式程序提供分布式协调服务，用于保持配置信息，命名服务，提供分布式锁，提供集群服务
## 设计目标
### 简单的
zookeeper允许分布式程序访问namespace,namespace和文件系统类似。namespace有znode构成。zookeeper数据保存在内存中，所以可以实现高吞吐和低延迟。
高性能，高可用，严格的顺序性。
### 可复制的
![zkservice](./images/zkservice.jpg)
客户端只连接到一个服务器，客户端通过这个TCP连接，发送请求、接收响应、获取观察时间、发送心跳。如果TCP连接与服务器断开，客户端将连接到另一台服务器。
### 有序的
zookeeper的每个更新都有一个戳记，它反映了所有zookeepr所有事物的顺序。
### 快速的
## 数据模型和分层次的命名空间
![zknamespace](./images/zknamespace.jpg)
## 节点和短暂的节点
znode有子znode，每个znode可以存储数据，例如：状态信息、配置、本地信息、环境信息。存储在节点的数据通常比较小，大小为1M。
znode的结构包含版本的改变。每个节点有访问控制列表（ACL）,列出谁能做什么。
zookeeper有瞬态znode，节点将一直存在如果session一直存在。session结束节点自动删除。
## 更新和监控的条件
客户端可以监控这个znode，当该节点改变监控事件将被触发。
## API
create
delete
exists
get data
set data
sync
## 实现
![zkcomponents](./images/zkcomponents.jpg)
复制数据库内存中的数据库，包含数据树的入口。更新操作记录日志到硬盘，写操作在修改复制数据库前更新到硬盘。
客户端提交所有的写请求给leader,followers从leader接收请求
zookeeper消息协议为原子的，所以本地的副本不存在分歧。
## 性能
![zkperfRW-3.2](./images/zkperfRW-3.2.jpg)
## 可靠性
# 开始
## conf/zoo.cfg
```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
``` 
tickTime:心跳时间，单位毫秒
dataDir:内存数据库
clientPort:链接端口

# 管理zookeeper存储
长时间运行的生产系统必须管理dataDir 和log
