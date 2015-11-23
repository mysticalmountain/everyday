[TOC]
# 创建zk连接
```java
public class ConnectingExample {

    private static final int SESSION_TIMEOUT = 5000;

    public ZooKeeper connect(String hosts) throws IOException, InterruptedException {
        final CountDownLatch signal = new CountDownLatch(1);
        ZooKeeper zk = new ZooKeeper(hosts, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getState() == Event.KeeperState.SyncConnected) {              //连接创建成功后客户端收单会受到服务店的通知
                    signal.countDown();
                }
            }
        });
        signal.await();
        return zk;
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        ConnectingExample example = new ConnectingExample();
        ZooKeeper zk = example.connect("localhost:2181");
        System.out.printf("ZK state: %s\n", zk.getState());
        zk.close();
    }
}
```
# zk事务处理
:warning:zk3.4.0版本及以上支持处理处理
```java
public class ConnectionHelper {

    public static final int DEFAULT_SESSION_TIMEOUT = 5000;

    public ZooKeeper connect(String hosts, int sessionTimeout) throws IOException, InterruptedException {
        final CountDownLatch connectedSignal = new CountDownLatch(1);
        ZooKeeper zk = new ZooKeeper(hosts, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getState() == Watcher.Event.KeeperState.SyncConnected) {
                    connectedSignal.countDown();
                }
            }
        });
        connectedSignal.await();
        return zk;
    }

    public ZooKeeper connect(String hosts) throws IOException, InterruptedException {
        return connect(hosts, DEFAULT_SESSION_TIMEOUT);
    }
}

public class TransactionExample {

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        if (args.length < 4) {
            System.out.printf("Usage: %s <zk-connection-string> <parent-znode> <child-znode-1> <child-znode-2> [<child-znode-n> ...]\n",
                    TransactionExample.class.getSimpleName());
            System.exit(1);
        }

        ZooKeeper zooKeeper = new ConnectionHelper().connect(args[0]);

        String topZnodePath = "/txn-examples";
        if (zooKeeper.exists(topZnodePath, false) == null) {
            System.out.printf("Creating top level znode %s for transaction examples\n", topZnodePath);
            zooKeeper.create(topZnodePath, null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }

        String baseParentPath = topZnodePath + "/" + args[1] + "-";
        String parentPath = zooKeeper.create(baseParentPath, null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
        System.out.printf("Created parent znode %s\n", parentPath);

        List<String> childPaths = new ArrayList<String>(args.length - 2);
        Transaction txn = zooKeeper.transaction();
        for (int i = 2; i < args.length; i++) {
            String childPath = parentPath + "/" + args[i];
            childPaths.add(childPath);
            System.out.printf("Adding create op with child path %s\n", childPath);
            txn.create(childPath, null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        System.out.println("Committing transaction");
        List<OpResult> opResults = txn.commit();

        System.out.println("Transactions results:");
        for (int i = 0; i < opResults.size(); i++) {
            OpResult opResult = opResults.get(i);
            int type = opResult.getType();
            String childPath = childPaths.get(i);
            switch (type) {
                case ZooDefs.OpCode.create:
                    System.out.printf("Child node %s created successfully\n", childPath);
                    break;
                case ZooDefs.OpCode.error:
                    System.out.printf("Child node %s was not created. There was an error.\n", childPath);
                    break;
                default:
                    System.out.printf("Don't know what happened with child node %s! OpResult type: %d", childPath, type);
            }
        }
    }
}
```

# 创建node
```java
public class CreateNodeExample implements Watcher {

    private static final int SESSION_TIMEOUT = 5000;
    private ZooKeeper _zk;
    private CountDownLatch _connectedSignal = new CountDownLatch(1);

    public void connect(String hosts) throws IOException, InterruptedException {
        _zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
        _connectedSignal.await();
    }

    @Override
    public void process(WatchedEvent event) { // Watcher interface
        if (event.getState() == Watcher.Event.KeeperState.SyncConnected) {
            System.out.println("Connected...");
            _connectedSignal.countDown();
        }
    }

    public void create(String groupName) throws KeeperException, InterruptedException {
        String path = "/" + groupName;
        String createdPath = _zk.create(path,
                null /*data*/,
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT);
        System.out.println("Created " + createdPath);
    }

    public void close() throws InterruptedException {
        _zk.close();
    }

    public static void main(String[] args) throws Exception {
        CreateNodeExample createNode = new CreateNodeExample();
        createNode.connect(args[0]);
        createNode.create(args[1]);
        createNode.close();
    }
}


输出
Created parent znode /txn-examples/node1-0000000002
Adding create op with child path /txn-examples/node1-0000000002/node11
Adding create op with child path /txn-examples/node1-0000000002/node12
Committing transaction
Transactions results:
Child node /txn-examples/node1-0000000002/node11 created successfully
Child node /txn-examples/node1-0000000002/node12 created successfully
```

验证我们的节点是否已经创建
```shell
连接到zk
zkCli.sh -server 192.168.255.133:2181

查看跟目录
ls /
[txn-examples, zookeeper]
查看新建的目录
ls /txn-examples/node1-0000000002
[node11, node12]
获取节点信息
get /txn-examples/node1-0000000002
null
cZxid = 0x20
ctime = Fri Nov 20 14:27:16 CST 2015
mZxid = 0x20
mtime = Fri Nov 20 14:27:16 CST 2015
pZxid = 0x21
cversion = 2
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
```

# 删除node
```java
public class ConnectionWatcher implements Watcher {

    private static final int SESSION_TIMEOUT = 5000;
    protected ZooKeeper zk;
    private CountDownLatch _connectedSignal = new CountDownLatch(1);

    public void connect(String hosts) throws IOException, InterruptedException {
        zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
        _connectedSignal.await();
    }

    @Override
    public void process(WatchedEvent event) {
        if (event.getState() == Event.KeeperState.SyncConnected) {
            _connectedSignal.countDown();
        }
    }

    public void close() throws InterruptedException {
        zk.close();
    }
}

public class DeleteNodeExample extends ConnectionWatcher{
    public void delete(String groupName) throws KeeperException, InterruptedException {
        String path = "/" + groupName;

        try {
            List<String> children = zk.getChildren(path, false);
            for (String child : children) {
                zk.delete(path + "/" + child, -1);
            }
            zk.delete(path, -1);
            System.out.printf("Deleted group %s at path %s\n", groupName, path);
        }
        catch (KeeperException.NoNodeException e) {
            System.out.printf("Group %s does not exist\n", groupName);
        }
    }

    public static void main(String[] args) throws Exception {
        DeleteNodeExample deleteNode = new DeleteNodeExample();
        deleteNode.connect(args[0]);
        deleteNode.delete(args[1]);
        deleteNode.close();
    }
}
```

# 同步获取子节点
```java
public class AsyncNodeList extends ConnectionWatcher {
    public static void main(String[] args) throws Exception {
        AsyncNodeList asyncNodeList = new AsyncNodeList();
        asyncNodeList.connect(args[0]);
        asyncNodeList.list(args[1]);
        asyncNodeList.close();
    }

    public void list(final String groupName) throws KeeperException, InterruptedException {
        String path = "/" + groupName;

        // In real code, you would not use the async API the way it's being used here. You would
        // go off and do other things without blocking like this example does.
        final CountDownLatch latch = new CountDownLatch(1);
        zk.getChildren(path, false,
                new AsyncCallback.ChildrenCallback() {
                    @Override
                    public void processResult(int rc, String path, Object ctx, List<String> children) {
                        System.out.printf("Called back for path %s with return code %d\n", path, rc);
                        if (children == null) {
                            System.out.printf("Group %s does not exist\n", groupName);
                        } else {
                            if (children.isEmpty()) {
                                System.out.printf("No members in group %s\n", groupName);
                                return;
                            }
                            for (String child : children) {
                                System.out.println(child);
                            }
                        }
                        latch.countDown();
                    }
                }, null /* optional context object */);
        System.out.println("Awaiting latch countdown...");
        latch.await();
    }
}
```

# 获取子节点
```java
public class NodeList extends ConnectionWatcher {
    public void list(String groupName) throws KeeperException, InterruptedException {
        String path = "/" + groupName;
        try {
            List<String> children = zk.getChildren(path, false);
            if (children.isEmpty()) {
                System.out.printf("No members in group %s\n", groupName);
                return;
            }
            for (String child : children) {
                System.out.println(child);
            }
        } catch (KeeperException.NoNodeException e) {
            System.out.printf("Group %s does not exist\n", groupName);
        }
    }

    public static void main(String[] args) throws Exception {
        NodeList nodeList = new NodeList();
        nodeList.connect(args[0]);
        nodeList.list(args[1]);
        nodeList.close();
    }
}
```

# 获取znode数据
```java
public class PrintNodeInfo {
    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        ZooKeeper zk = new ConnectionHelper().connect("127.0.0.1:2181");
        String node = "/sample-node";
        if (zk.exists(node, false) == null) {
            System.out.printf("Creating znode %s for transaction examples\n", node);
            zk.create(node, "node info for check print znode info".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        byte[] resByte = zk.getData("/sample-node", null, null);
        System.out.printf("znode info:%s \n", new String(resByte));
    }
}
输出
Creating znode /sample-node for transaction examples
znode info:node info for check print znode info 

public class ZnodeWatcherExample {
    public static void main(String[] args) throws Exception {
        ZnodeWatcherExample zwe = new ZnodeWatcherExample();
        zwe.watcher();
    }

    public void watcher() throws Exception {
        ZooKeeper zk = new ConnectionHelper().connect("127.0.0.1:2181");
        String node = "/sample-node";
        if (zk.exists(node, false) == null) {
            System.out.printf("Creating znode %s for transaction examples\n", node);
            zk.create(node, "node info for check print znode info".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        System.out.printf("query and watcher znode %s\n", node);
        byte[] resByte = zk.getData(node, getDataWatcher, null);
        System.out.printf("znode data:%s\n", new String(resByte));
        System.out.printf("update znode data\n");
        zk.setData(node, "node had update5".getBytes(), 2);
        resByte = zk.getData(node, null, null);
        System.out.printf("znode data %s\n", new String(resByte));
    }

    Watcher getDataWatcher = new Watcher() {
        public void process(WatchedEvent e) {
            System.out.println("watch event:" + e);
        }
    };
}

输出
query and watcher znode /sample-node
znode data:node had update1
update znode data
watch event:WatchedEvent state:SyncConnected type:NodeDataChanged path:/sample-node
znode data node had update5
```

# 节点状态
```java
public class NodeStatExample {
    public static void main(String[] args) throws Exception {
        NodeStatExample zwe = new NodeStatExample();
        zwe.stat();
    }

    public void stat() throws Exception {
        ZooKeeper zk = new ConnectionHelper().connect("127.0.0.1:2181");
        String node = "/sample-node";
        if (zk.exists(node, false) == null) {
            System.out.printf("Creating znode %s for transaction examples\n", node);
            zk.create(node, "node info for check print znode info".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
        Stat stat = zk.exists(node, false);
        System.out.println("stat:" + stat);
        System.out.println("最后一次修改的时间戳:" + stat.getMtime());
        System.out.println("创建时的时间戳:" + stat.getCtime());
        System.out.println("最后一次修改的zxid:" + stat.getMzxid());
        System.out.println("创建时的zxid:" + stat.getCzxid());
        System.out.println("数据修改的版本号:" + stat.getVersion());
        System.out.println("子节点修改的版本号:" + stat.getCversion());
        System.out.println("(访问控制列表)改变的版本号:" + stat.getAversion());
        System.out.println("节点所有者:" + stat.getEphemeralOwner());
        System.out.println("数据的长度:" + stat.getDataLength());
        System.out.println("子节点的数量:" + stat.getNumChildren());
    }

    Watcher getDataWatcher = new Watcher() {
        public void process(WatchedEvent e) {
            System.out.println("watch event:" + e);
        }
    };
}

输出
stat:38,61,1448003006922,1448004244505,3,0,0,0,16,0,38

最后一次修改的时间戳:1448004244505
创建时的时间戳:1448003006922
最后一次修改的zxid:61
创建时的zxid:38
数据修改的版本号:3
子节点修改的版本号:0
(访问控制列表)改变的版本号:0
节点所有者:0
数据的长度:16
子节点的数量:0
```