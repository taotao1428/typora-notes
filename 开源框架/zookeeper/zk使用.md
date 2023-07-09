# zk使用

https://juejin.cn/post/6844903677367418893

zookeeper是一个分布式协调系统，由雅虎公司开发。

zookeeper是一个用于存放数据的产品，它支持单机部署和集群部署，集群部署具有高可用的特性。



## zookeeper的集群架构

![ZooKeeper官方架构图](/开源框架/zookeeper/.assert/zk使用/watermark,typZe_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5ODI3MTMx,size_16,color_FFFFFF,t_70.png)

| 节点类型 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Leader   | 处理客户端**读写**请求，并将数据同步给其他节点               |
| Follower | 从节点，处理客户端读请求。从Leader节点接收数据，当leader故障后，会从Follower节点中选择一个作为新Leader节点 |
| Observer | 观察节点，处理客户端读请求。从Leader节点接收数据，不参数选举和半数提交 |



## 读写请求处理

### 写请求

zookeeper的写请求都是由Leader节点处理。如果从节点接收到客户端的写请求，将会转发给Leader处理。



### 读请求





### ZK概念

### 节点类型

| 节点类型                       | 特性                                                         |      |
| ------------------------------ | ------------------------------------------------------------ | ---- |
| PERSISTENT                     | 永久节点                                                     |      |
| PERSISTENT_SEQUENTIAL          | 自动增加序号的永久节点                                       |      |
| EPHEMERAL                      | 临时节点，当连接断开后，将会被删除。不能创建子节点           |      |
| EPHEMERAL_SEQUENTIAL           | 自动增加需要的临时节点，当临时断开后，将会被删除。不能创建子节点 |      |
| CONTAINER                      | 容器节点，当子节点全部删除后，服务器将会在某个时间点删除容器节点 |      |
| PERSISTENT_WITH_TTL            | 有存活时间的永久节点，当节点没有子节点，并且在TTL时间内没有被修改，将会被删除 |      |
| PERSISTENT_SEQUENTIAL_WITH_TTL | 有存活时间并且自动增加序号的永久节点，当节点没有子节点，并且在TTL时间内没有被修改，将会被删除 |      |



### 节点属性

#### 1. 路径path

代表每个节点的位置

#### data

每个节点的数据

#### acl

权限控制。分为两类，权限和对象

权限perm

| 权限       | 说明                         |
| ---------- | ---------------------------- |
| READ       | 读取节点数据和**子节点列表** |
| WRITE      | 修改节点数据                 |
| **CREATE** | 创建子节点                   |
| **DELETE** | 删除子节点                   |
| ADMIN      | 设置节点访问控制权限         |



对象id

| schema         | id                                             | 说明                                     |
| -------------- | ---------------------------------------------- | ---------------------------------------- |
| world          | 只能为anyone                                   | 表示任何人                               |
| ip             | ip或者网段，例如：192.168.0.10，192.168.1.0/24 | 表示允许指定ip或者网段内的客户端可以访问 |
| auth           | 空字符串                                       | 表示通过认证的客户端                     |
| digest（常用） | username:BASE64(SHA1(password))                | 表示通过usename:password认证的客户端     |



### 节点信息

| **状态属性**   | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 数据节点创建时的事务ID                                       |
| ctime          | 数据节点创建时的时间                                         |
| mZxid          | 数据节点最后一次更新时的事务ID                               |
| mtime          | 数据节点最后一次更新时的时间                                 |
| pZxid          | 数据节点的子节点列表最后一次被修改（是子节点列表变更，而不是子节点内容变更）时的事务ID |
| cversion       | 子节点的版本号                                               |
| dataVersion    | 数据节点的版本号                                             |
| aclVersion     | 数据节点的ACL版本号                                          |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的SessionID；如果节点是持久节点，则该属性值为0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 数据节点当前的子节点个数                                     |

### 安全认证

待了解





## 监听功能

### 监听一次

当查询查询节点或者判断节点是否存在时，可以注册一个watcher。这种watch只能被触发一次。



### 持久监控

使用addWatch添加，这种watch在触发时，不会自动移除，除非手动移除



| watcher模式          | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| STANDARD             | 仅触发一次，仅监听本节点。在existed和getData方法注册的watcher属于这种类型。不能通过addWatcher添加这种模式的watcher |
| PERSISTENT           | 持久监控节点                                                 |
| PERSISTENT_RECURSIVE | 持久监控节点已经其子孙节点，递归监听所有节点                 |



### 监听的事件

监控的连接情况

| 连接状态          | 说明                              | 触发时间               |
| ----------------- | --------------------------------- | ---------------------- |
| Disconnected      | 连接断开，由客户端触发            | 因为异常连接断开时触发 |
| SyncConnected     | 连接正常                          |                        |
| AuthFailed        | 认证失败                          |                        |
| ConnectedReadOnly | 连接到只读节点                    |                        |
| SaslAuthenticated | sasl认证成功                      |                        |
| Expired           | 连接已经过期，需要重新创建session |                        |
| Closed            | 客户端关闭，由客户端自己触发      | 主动连接断开时触发     |



通知的事件

| 事件                   | 说明               | 触发时间                            |
| ---------------------- | ------------------ | ----------------------------------- |
| None                   | 代表非节点变更事件 | 非正常事件                          |
| NodeCreated            | 节点创建           | 节点创建                            |
| NodeDeleted            | 节点删除           | 节点删除                            |
| NodeDataChanged        | 节点数据变更       | 节点数据变更                        |
| NodeChildrenChanged    | 节点子节点列表变更 | 创建子节点和新增子节点              |
| DataWatchRemoved       | 节点监控移除       | DataWatcher被移除。客户端触发       |
| ChildWatchRemoved      | 子节点监控移除     | ChildWatcher被移除。客户端触发      |
| PersistentWatchRemoved | 永久监控移除       | PersistentWatcher被移除。客户端触发 |





#### 事件通知实现

watcher类型

| 类型     | 说明             |
| -------- | ---------------- |
| Data     | 仅监听数据变化   |
| Children | 仅监听子节点变化 |
| Any      | 任何变化         |



## 配置文件

1. tickTime client-server 通信心跳时间
   1. zk 服务器之间或client 与服务器之间维持心跳的时间间隔、也就是每个tickTime 就会发送一个心跳、tickTime 以毫秒为单位
2. initLimit leader-follower 初始通信时限
   1. 集群中 follower 与leader 之间初始连接时最多能容忍的最多心跳数 。
3. syncLimit leader follower 同步通信时限
   1. follower与 leader 服务器请求与应答之间能容忍的最多心跳数
4. dataDir 数据目录



```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
```



### 客户端对ping的机制

```
readTimeout = negotiatedSessionTimeout * 2 / 3;

```



```java
                        int timeToNextPing = readTimeout / 2
                                             - clientCnxnSocket.getIdleSend()
                                             - ((clientCnxnSocket.getIdleSend() > 1000) ? 1000 : 0);
                        //send a ping request either time is due or no packet sent out within MAX_SEND_PING_INTERVAL
                        if (timeToNextPing <= 0 || clientCnxnSocket.getIdleSend() > MAX_SEND_PING_INTERVAL) {
                            sendPing();
                            clientCnxnSocket.updateLastSend();
                        }
```



1. 如果服务端超过readTimeout时间没有从服务端接口消息，将会断开当前连接。因此服务端需要定期给客户端发送ping
2. 当超过readTimeout/2时间内没有发送消息时，客户端将会主动发送ping给服务端



### follower对ping的机制

follower接收到ping之后，会将所有的touch过的session发送给leader

follower与Leader建立通信的connectTimeout为tickTime * initLimit



Leader每1/2 * tickTime就会发送一个ping请求给follower



### 节点配置

配置文件为server配置为server_config或者server_config;client_config。server_config可以配置多个地址，每个地址使用`|`分割，每个地址的格式为host:port:port或者host:port:port:type。client_config为port或者host:port

```java
        private static final String wrongFormat =
            " does not have the form server_config or server_config;client_config"
            + " where server_config is the pipe separated list of host:port:port or host:port:port:type"
            + " and client_config is port or ";

```









## zookeeper功能



### 特点

1. zookeeper是一个分布式系统，当有半数以上的节点正常时，系统可以正常工作。
2. 数据一致性。主节点故障后，新主节点上会有全部已成功提交的数据，不会丢失数据
3. 会话功能。在客户端与服务端网络断开后，如果客户端在指定时间内连接到服务端，此时会话将继续存在。该特性可以使客户端与服务端通信稳定。
4. 订阅和通知功能。客户端可以订阅事件，当时间发生时，服务端会通知客户端



