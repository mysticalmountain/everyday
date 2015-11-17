# 数据模型
zookeeper有分层次的命名空间，类似于文件系统的目录；区别为每个节点都有自己的子节点。
任何的unicode字符有可用于zookeeper路径，以下字符除外：
```
null字符 (\u0000) 不允许作为path的一部分
\u0001 - \u001F 和 \u007F - \u009F不能使用，因为不能正常显示
\ud800 - uF8FF, \uFFF0 - uFFFF 不允许
"." 或 ".." 不允许，zookeeper也不允许使用相对路径
"zookeeper" 是保留字，不允许使用
```

## znode
znode结构包含
+ 数据改变的版本号
+ 访问控制列表的版本号
+ 时间戳
当znode数据改变版本号增长，客户端检索回数据包含版本信息，当客户端更新或删除节点会将版本信息上送，服务店检查版本号如果不匹配更新失败。
znode是主要的实体，有一下几点提出：

- [x] Watches
客户端可以在znode上设置watches，改变节点将触发watche事件，当watche事件被触发客户端会受到服务端的通知。
- [x] Data Access
znode的命名空间自动存储了是可读或可写，读获取znode的所有信息，写替换znode的所有数据。每个节点有访问控制列表（ACL）约束谁可以做什么
zookeeper没设计成为像一般的数据或者大对象存储。它用于管理协调数据。这些数据指：配置信息，状态信息。每个znode的数据大小应该小于1M。如果需要大数据存储，一般存储在存储系统，NFS或HDFS,zookeeper中存储指针指向存储位置。
- [x] 短暂的节点
zookeeper有短暂的节点概念，节点的生存时间与session时间一样长。session超时节点删除，短暂节点不允许有子节点。
- [x] 顺序节点
当创建znode，能请求zookeeper添加一个自动增长的计数器，这个计数器是唯一的在父znode。counter的格式为10个数字不够前面加0。父节点维护了下一个计数器的值，如果计数器值大于2147483647结果为-2147483647 

## Time in zookeeper
- Zxid
任何zookeeper状态的改变都会有一个唯一的事务戳记，它暴漏了所有改变的顺序。if zxid1 < zxid2 zxid1将会在zxid2前发生。
- Version numbers
每次znode改变都会造成版本号改变。版本号分为3个
version:znode数据改变版本号
cversion:子节点改变版本号
aversion:znode ACL改变版本号
- Ticks
用于检测session超时，链接超时等
- Real time
zookeeper不使用真正的时间，除节点的创建时间和修改时间

## 状态结构
每个znode的状态结构有如下属性组成

- czxid
znode创建时的zxid
- mzxid
znode最后一次修改的zxid
- ctime
znode创建时的时间戳
- mtime
znode最后一次修改的时间戳
- version
znode数据修改的版本号
- cversion
znode子节点修改的版本号
- aversion
znode ACL(访问控制列表)改变的版本号
- ephemeralOwner
如果是短暂的节点：znode所属session的session id
如果非短暂的节点：0
- dataLength
znode数据的长度
- numChildren
znode子节点的数量

## Sessions
![state_dia](./images/state_dia.jpg)
创建session需要连接字符串，字符串包含主机列表。（"127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"）客户端将选择任意一个服务器连接。如果连接失败了或者因为任何原因连接关闭了，客户端将自动的与服务器列表的其他机器建立连接。
连接字符串还支持写法：127.0.0.1:4545/app/a  session创建成功后目录更目录为“app/a”

客户端获取zookeeper服务的句柄，zookeeper将创建一个session，一个64bit数字标志一个客户端。如果客户端与另一台服务器建立连接，session id 做作为握手通讯内容的一部分，服务器为session id 穿件一个密码，任何服务器都可以验证该密码。当客户端创建连接时发送session id和密码