# zk实现原理



## 基本实现

### 节点上的数据

zookeeper上的数据是树型结构，保存在数据库中。另外为了防止数据丢失，客户端的写请求会被封装成日志，被保存在日志文件中。当节点故障恢复后，可以通过日志文件恢复数据。同时为了快速恢复数据，会将内存中的数据制作快照，保存在快照文件中。因为有快照可以直接恢复数据，而不用从头开始回放日志。



数据是通过`org.apache.zookeeper.server.ZKDatabase`维护的

| 方法                                                    | 说明                               |
| ------------------------------------------------------- | ---------------------------------- |
| long loadDataBase()                                     | 从日志和快照文件中恢复数据         |
| boolean append(Request si)                              | 将请求写入到日志文件中还需要提交   |
| void commit()                                           | 将之前提交的请求的日志，写入磁盘中 |
| processTxn(TxnHeader hdr, Record txn, TxnDigest digest) | 将proposal应用到内存数据库中       |



#### 树形数据DataTree

内存中的树形数据对应类`org.apache.zookeeper.server.DataTree`

所有对节点的增删改查、监听和ACL的操作，都是通过该类完成





#### 日志和快照文件FileTxnSnapLog

日志文件对应的类为`org.apache.zookeeper.server.persistence.FileTxnSnapLog`



| 方法                                                         | 说明                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| boolean append(Request si)                                   | 添加请求到日志文件，不一定保存到磁盘 |
| void commit()                                                | 将日志数据flush到磁盘中              |
| void rollLog()                                               | 日志文件归档                         |
| void save(DataTree dataTree,  sessionsWithTimeouts,    boolean syncSnap) | 保存快照到文件中                     |
| long restore(DataTree dt,  sessions, PlayBackListener listener) | 从快照文件和日志文件中恢复数据       |



#### 恢复数据的流程

`org.apache.zookeeper.server.persistence.FileTxnSnapLog#restore`

1. 从最新的快照中先恢复数据
2. 然后从日志文件中回放快照zxid之后的日志



### 选举

zookeeper包含了三种角色Leader，Follower和Observer。对应节点有4中状态

1. Looking：未确定自己的状态，需要通过选举确定
2. Leading：Leader节点的状态
3. Following：follower节点的状态
4. Observing：Observer节点的状态

#### 状态转移矩阵

**待补充Observing**

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220327234031905.png" alt="image-20220327234031905" style="zoom:50%;" />



#### 投票规则

当节点处于Looking状态时，会发起投票，从而确定当前的Leader。

1. 每一轮投票会将自己的逻辑时钟+1，并且最开始的投票是自己
2. 如果收到逻辑时钟大于自己的投票，将会采用该投票，并把自己的逻辑时钟设置成该逻辑时钟，并重新通知其他人自己的投票。如果收到的逻辑时钟小于自己的投票，将会直接丢弃该投票。如果逻辑时钟相等，将会比较本地投票与接收到的投票，如果收到的投票更大，将会更新自己的投票，并重新通知其他人自己的投票，否者，将会丢弃该投票。
3. 当某个投票超过半数节点认同时，将会结束投票，从投票中获取当前的Leader。



##### 投票比较方法

依次比较epoch（节点纪元），zxid（节点最大的zxid）和sid（节点id），值越大的越胜出



##### 投票状态转移

待补充



### 同步

在选举结束之后，节点的角色也就确认了，此时Leader和Follower需要同步，同步之后才会对外提供服务。同步是指：Leader和Follower上的数据是一致的。



#### 同步的方法

为了更快与Follower同步日志，Leader会将最新的部分已提交的请求保存在内存中，默认是500个请求。为了方便描述同步的方法，假设这部分请求的最大zxid为maxCommittedLog，最小zxid为minCommittedLog。Follower的zxid为peerLastZxid

| 情况                                           | 方法      | 说明                                                         |
| ---------------------------------------------- | --------- | ------------------------------------------------------------ |
| peerLastZxid>maxCommittedLog                   | TUNC      | 说明Follower的日志已经超前Leader，需要将日志截取到maxCommittedLog位置 |
| minCommittedLog<=peerLastZxid<=maxCommittedLog | DIFF      | 表示Follower和Leader基本同步，直接通过CommitedLog恢复        |
| peerLastZxid<minCommitedLog                    | DIFF/SNAP | 从日志文件提取内存中没有的日志发送给Follower（DIFF），如果日志文件中缺少日志，将会使用快照的方式（SNAP） |





### 传播数据



#### 客户端连接到Leader

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220329084829684.png" alt="image-20220329084829684" style="zoom:50%;" />



#### 客户端连接到Follower

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220329085205782.png" alt="image-20220329085205782" style="zoom:50%;" />



## 实现

### 会话

1. 客户端内部会维护两个队列，里面分别保存发送和返回的消息。
2. 客户端里面会包含ClientCnxn，它负责维护与服务端的session和处理接发消息
3. SendThread用于循环监听socket事件。处理connect，read和write事件。其中对于read事件，SendThread只是把接收到消息放到队列中，EventThread会从队列中获取消息，然后处理。

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220330083207830.png" alt="image-20220330083207830" style="zoom:50%;" />

#### 会话创建

客户端与服务端的tcp连接刚刚建立时，客户端会发送ConnectRequest请求给服务端，里面会包含客户端sessionId、paaswd和sessionTimeout给服务端。服务端会返回一个ConnectResponse给客户端，里面会包含最终session的sessionId、paaswd和sessionTimeout



下图是客户端连接到Follower时，ConnectRequest的处理流程。

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220330085038581.png" alt="image-20220330085038581" style="zoom:50%;" />

1. 客户端会发送以前的sessionId和paaswd给服务端，用于在重新建立tcp连接时仍保持session
2. 客户端会发送之前接收到的最大zxid，当服务端的zxid小于该值时，将会拒绝连接。用于保证服务端时线性一致性
3. Follower会将创建session的请求转发给leader，由leader统一管理session。创建session的请求也会被Leader传播给Follower，保证Leader故障后，新Leader上会有session数据。
4. 客户端发送的timeout可能不是最终使用的timeout，如果客户端的timeout超出一定范围，服务端将会重新设置timeout。最终协商后的timeout为ConnectResponse中的timeout。



#### 会话保活

session的管理是统一由Leader管理，节点通过以下方法实现节点探活

1. Follower和Leader是根据能否接收到请求判断连接是否存活。
2. 客户端在readTimeout时间没有接收到响应，会主动断开连接。`readTimeout = negotiatedSessionTimeout * 2 / 3`; negotiatedSessionTimeout是最终服务端和客户端协商的session超时时间。
3. 客户端如果readTimeout/2没有发送消息给服务端，将会主动发送一个ping消息，用于保活。
4. 每tickTime/2，Leader会发送ping消息给Follower，Follower会返回本节点还存活的session（仍在发送消息的session）。

<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220401085556699.png" alt="image-20220401085556699" style="zoom:50%;" />





#### 会话清理

Leader上会每隔tickTime清理一次会话



### watcher

watcher是由客户端和服务端共同实现。

1. 注册watcher。客户端会向服务端注册watcher，服务端的watcher是注册到DataTree上，同时客户端也会注册本地的watcher。如果操作触发了监听的事件，服务端会向客户端发送一个event事件。客户端接收后，将会触发本地的watcher
2. 移除watcher。客户端会向服务端删除watcher，同时也会删除本地的watcher。
3. **watcher是连接级别的，不是session级别。如果连接断开，watcher需要重新注册**。





<img src="/开源框架/zookeeper/.assert/zk实现原理2/image-20220403113059739.png" alt="image-20220403113059739" style="zoom:50%;" />

#### 注册Watcher模式

`org.apache.zookeeper.AddWatchMode`

| watcher模式          | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| STANDARD             | 仅触发一次，仅监听本节点。在existed和getData方法注册的watcher属于这种类型。不能通过addWatcher添加这种模式的watcher |
| PERSISTENT           | 持久监控节点，触发后不会手动移除，需手动移除，可以通过addWatch注册该模式的watcher |
| PERSISTENT_RECURSIVE | 持久监控节点已经其子孙节点，递归监听所有节点，触发后不会手动移除，需手动移除，可以通过addWatch注册该模式的watcher |



#### 通知的事件

除了服务端触发事件之外，客户端会主动根据当前与服务端的连接状态变化，主动触发watcher。事件对象中包含keeperState和eventType。

```java
public class WatchedEvent {
    private final KeeperState keeperState;
    private final EventType eventType;
    private String path;
}
```



##### 客户端和服务端连接状态

监控的连接情况KeeperState

| 连接状态          | 说明                              | 触发时间               |
| ----------------- | --------------------------------- | ---------------------- |
| Disconnected      | 连接断开，由客户端触发            | 因为异常连接断开时触发 |
| SyncConnected     | 连接正常                          | 服务端触发事件         |
| AuthFailed        | 认证失败                          |                        |
| ConnectedReadOnly | 连接到只读节点                    |                        |
| SaslAuthenticated | sasl认证成功                      |                        |
| Expired           | 连接已经过期，需要重新创建session |                        |
| Closed            | 客户端关闭，由客户端自己触发      | 主动连接断开时触发     |



##### 服务端通知的事件类型

EventType

| 事件                   | 说明               | 触发时间                                               |
| ---------------------- | ------------------ | ------------------------------------------------------ |
| **None**               | 代表非节点变更事件 | 非正常事件。**如果不是服务端触发事件，事件类型为None** |
| NodeCreated            | 节点创建           | 节点创建                                               |
| NodeDeleted            | 节点删除           | 节点删除                                               |
| NodeDataChanged        | 节点数据变更       | 节点数据变更                                           |
| NodeChildrenChanged    | 节点子节点列表变更 | 创建子节点和新增子节点                                 |
| DataWatchRemoved       | 节点监控移除       | DataWatcher被移除。客户端触发                          |
| ChildWatchRemoved      | 子节点监控移除     | ChildWatcher被移除。客户端触发                         |
| PersistentWatchRemoved | 永久监控移除       | PersistentWatcher被移除。客户端触发                    |



