# Zab



## 整体架构

### 单个节点的状态转移

<img src="/开源框架/zookeeper/.assert/zab/image-20220326224651881.png" alt="image-20220326224651881" style="zoom:50%;" />



### 投票

投票是由`org.apache.zookeeper.server.quorum.flexible.QuorumVerifier`判断投票是否有效，通常使用的实现类为`org.apache.zookeeper.server.quorum.flexible.QuorumMaj`，它会从配置文件中读取服务器的配置

```
public synchronized void startLeaderElection() {
    try {
        if (getPeerState() == ServerState.LOOKING) {
            currentVote = new Vote(myid, getLastLoggedZxid(), getCurrentEpoch());
        }
    } catch (IOException e) {
        RuntimeException re = new RuntimeException(e.getMessage());
        re.setStackTrace(e.getStackTrace());
        throw re;
    }

    this.electionAlg = createElectionAlgorithm(electionType);
}
```



处理从其他节点接收到的请求。从QuorumCnxManager#recvQueue中获取信息。并处理

```
org.apache.zookeeper.server.quorum.FastLeaderElection.Messenger.WorkerReceiver
```



接收到的消息

```java
    public static class Notification {
        public static final int CURRENTVERSION = 0x2;
        int version; // 消息的版本
         // Proposed leader 对方提议的leader
         long leader; 

        /*
         * zxid of the proposed leader 对方提议的leader的zxid
         */ long zxid;

        /*
         * Epoch 对方竞选的epoch
         */ long electionEpoch;

        /*
         * current state of sender 对方的节点状态
         */ QuorumPeer.ServerState state;

        /*
         * Address of sender 对方的sid
         */ long sid;

        QuorumVerifier qv; // 对方的配置
        /*
         * epoch of the proposed leader。对方提议的leader的epoch
         */ long peerEpoch;

    }
```







处理接收消息的流程

<img src="/开源框架/zookeeper/.assert/zab/image-20220327130251194.png" alt="image-20220327130251194" style="zoom:50%;" />





比较两个投票的方法

这里的zxid为节点接收到的最大zxid，而不是提交的最大zxid。节点最开始投票是给自己，节点每轮投票都会给自己的logicalClock加1。如果当前环境已经存在稳定的Leader，将会直接加入，并且会忽略logicalClock的影响

```java
        /*
         * We return true if one of the following three cases hold:
         * 1- New epoch is higher
         * 2- New epoch is the same as current epoch, but new zxid is higher
         * 3- New epoch is the same as current epoch, new zxid is the same
         *  as current zxid, but server id is higher.
         */

        return ((newEpoch > curEpoch)
                || ((newEpoch == curEpoch)
                    && ((newZxid > curZxid)
                        || ((newZxid == curZxid)
                            && (newId > curId)))));
```





### 集群的机器通信

节点与其他节点通信管理类为`org.apache.zookeeper.server.quorum.QuorumCnxManager`



它会监听所有服务端的竞选端口，使用`org.apache.zookeeper.server.quorum.QuorumCnxManager#handleConnection`处理接收到的请求



对于接收到id小于自己的节点连接，先拒绝，然后主动向其发送连接请求。

```java
    private void handleConnection(Socket sock, DataInputStream din) throws IOException {
        //If wins the challenge, then close the new connection.
        if (sid < self.getId()) {
            /*
             * This replica might still believe that the connection to sid is
             * up, so we have to shut down the workers before trying to open a
             * new connection.
             */
            SendWorker sw = senderWorkerMap.get(sid);
            if (sw != null) {
                sw.finish();
            }

            /*
             * Now we start a new connection
             */
            LOG.debug("Create new connection to server: {}", sid);
            closeSocket(sock);

            if (electionAddr != null) {
                connectOne(sid, electionAddr);
            } else {
                connectOne(sid);
            }

        } else if (sid == self.getId()) {
            // we saw this case in ZOOKEEPER-2164
            LOG.warn("We got a connection request from a server with our own ID. "
                     + "This should be either a configuration error, or a bug.");
        } else { // Otherwise start worker threads to receive data.
            SendWorker sw = new SendWorker(sock, sid);
            RecvWorker rw = new RecvWorker(sock, din, sid, sw);
            sw.setRecv(rw);

            SendWorker vsw = senderWorkerMap.get(sid);

            if (vsw != null) {
                vsw.finish();
            }

            senderWorkerMap.put(sid, sw);

            queueSendMap.putIfAbsent(sid, new CircularBlockingQueue<>(SEND_CAPACITY));

            sw.start();
            rw.start();
        }
    }
```



负责发送消息到不同节点

```
org.apache.zookeeper.server.quorum.QuorumCnxManager.SendWorker
```



负责从不同节点接收消息

```
org.apache.zookeeper.server.quorum.QuorumCnxManager.RecvWorker
```



保存要发送到不同节点的消息。如果与目标节点还没有建立连接，将会开始建立连接

```
org.apache.zookeeper.server.quorum.QuorumCnxManager#queueSendMap
final ConcurrentHashMap<Long, BlockingQueue<ByteBuffer>> queueSendMap;
```



保存从不同节点接收到的消息

```
org.apache.zookeeper.server.quorum.QuorumCnxManager#recvQueue
public final BlockingQueue<Message> recvQueue;
```





### 重新配置功能

如果配置文件中`reconfigEnabled`为true，本地配置文件会跟随更高版本的配置变化，默认为false。

1. 在竞选阶段时，接到一个更高版本的配置
2. 客户端发送一个reconfig类型请求给Leader，然后会同步给其他Follower
3. 

```
org.apache.zookeeper.server.quorum.QuorumPeer#processReconfig
```



### 成为Leader的动作



### 成为Follower的动作

#### 连接Leader

1. 连接一个Leader的地址
2. 发送Leader.FOLLOWERINFO给Leader。包括节点的id和期望接收到的zxid。
3. Leader返回一个Leader.LEADERINFO消息。包括Leader下个消息的zxid。节点将从zxid中提取acceptedEpoch保存到文件中。如果Leader的zxid提取的Epoch小于本地的acceptedEpoch，将会报错。如果等于本地的acceptedEpoch，将不会返回一个不纳入统计的ack。acceptedEpoch会限制一个Follower选择的Leader会越新。
4. 节点返回一个Leader.ACKEPOCH消息给Leader。会包含节点最新的zxid，用于与Leader同步数据



#### 与Leader同步数据

1. Leader发送请求。请求中包含同步的类型。Leader.DIFF：当前不需要做任何操作，后面直接通过log同步；Leader.SNAP：Follower通过Leader发送过来的Snapshot同步；Leader.TRUNC：去掉一部分log。
2. Leader后续的log发送给Follower。Follower将它们保存到本地并写入内存





### 退出Leader的原因

当有一个半的Follower连接不上，将会主动退出Leader



### 退出Follower的原因

1. 出现系统错误，主动调用shutdown方法关闭
2. 与Leader连接错误





## 服务端之间的连接

每个服务端与其他服务端都建立了一个连接。对于sid小于自己的服务端，是主动连接，对于sid大于自己的服务端，是被动连接。



每个连接的sendWorker放置在org.apache.zookeeper.server.quorum.QuorumCnxManager#senderWorkerMap



另外每个连接的recevWorker接收到的消息，会被放置到org.apache.zookeeper.server.quorum.QuorumCnxManager#recvQueue中

```java
            ToSend notmsg = new ToSend(
                ToSend.mType.notification,
                proposedLeader,
                proposedZxid,
                logicalclock.get(),
                QuorumPeer.ServerState.LOOKING,
                sid,
                proposedEpoch,
                qv.toString().getBytes(UTF_8));

            LOG.debug(
                "Sending Notification: {} (n.leader), 0x{} (n.zxid), 0x{} (n.round), {} (recipient),"
                    + " {} (myid), 0x{} (n.peerEpoch) ",
                proposedLeader,
                Long.toHexString(proposedZxid),
                Long.toHexString(logicalclock.get()),
                sid,
                self.getId(),
                Long.toHexString(proposedEpoch));
```





## 数据一致性

1. 已经commit的消息，不能丢失
2. 已经被丢弃的消息，不能被提交



如何保证

1. 选出的leader，proposal最大zxid必须要不小于一半节点的proposal最大zxid。这保证新leader拥有所有commit消息
2. 新leader会将其他节点上，所有旧epoch是上未commit的proposal删除，然后从新leader再同步。保证过时的消息不会再出现
3. 每一轮新leader选举，都需要重新计算zxid，使zxid是递增的。





## 重要的类

监听服务通信端口

org.apache.zookeeper.server.quorum.QuorumCnxManager.Listener#run



## 系统如何启动

服务的状态

```java
    public enum ServerState {
        LOOKING,
        FOLLOWING,
        LEADING,
        OBSERVING
    }
```



ZAB整个集群的状态

```java
    public enum ZabState {
        ELECTION,
        DISCOVERY,
        SYNCHRONIZATION,
        BROADCAST
    }
```







## 系统如何检查leader状态





## 系统如何开始选举