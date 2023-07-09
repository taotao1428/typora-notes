# zk实现原理



zookeeper会需要设置3个端口

1. 客户端端口。用于客户端连接服务端
2. 服务端之间通信端口。只有Leader节点会监听这个端口，Follower和Observer节点会连接Leader的该端口
3. 服务端选举端口。用于服务通信



每中类型的节点由Leader，Follower，Observer实现业务逻辑。每个节点都是一个QuorumPeer对象



复制是Follower和Observer节点主动向Leader节点建立连接



## Follower节点处理Leader请求

org.apache.zookeeper.server.quorum.Follower#processPacket

```java
    protected void processPacket(QuorumPacket qp) throws Exception {
        switch (qp.getType()) {
        case Leader.PING:
            ping(qp); // 会将本地会话发送给leader
            break;
        case Leader.PROPOSAL: // 处理proposal消息
            TxnLogEntry logEntry = SerializeUtils.deserializeTxn(qp.getData());
            TxnHeader hdr = logEntry.getHeader();
            Record txn = logEntry.getTxn();
            TxnDigest digest = logEntry.getDigest();

            fzk.logRequest(hdr, txn, digest);

            break;
        case Leader.COMMIT: // 处理commit消息
            fzk.commit(qp.getZxid());
            break;

        case Leader.COMMITANDACTIVATE: // 处理新配置
            // get the new configuration from the request
            Request request = fzk.pendingTxns.element();
            SetDataTxn setDataTxn = (SetDataTxn) request.getTxn();
            QuorumVerifier qv = self.configFromString(new String(setDataTxn.getData(), UTF_8));

            // get new designated leader from (current) leader's message
            ByteBuffer buffer = ByteBuffer.wrap(qp.getData());
            long suggestedLeaderId = buffer.getLong();
            final long zxid = qp.getZxid();
            boolean majorChange = self.processReconfig(qv, suggestedLeaderId, zxid, true);
            // commit (writes the new config to ZK tree (/zookeeper/config)
            fzk.commit(zxid);

            break;
        case Leader.UPTODATE:
            LOG.error("Received an UPTODATE message after Follower started");
            break;
        case Leader.REVALIDATE:
            if (om == null || !om.revalidateLearnerSession(qp)) {
                revalidate(qp);
            }
            break;
        case Leader.SYNC:
            fzk.sync();
            break;
        default:
            LOG.warn("Unknown packet type: {}", LearnerHandler.packetToString(qp));
            break;
        }
    }
```







## Follower处理向Leader发送的请求

org.apache.zookeeper.server.quorum.FollowerRequestProcessor#run

```java
    public void run() {
        try {
            while (!finished) {
                ServerMetrics.getMetrics().LEARNER_REQUEST_PROCESSOR_QUEUE_SIZE.add(queuedRequests.size());

                Request request = queuedRequests.take();
                if (LOG.isTraceEnabled()) {
                    ZooTrace.logRequest(LOG, ZooTrace.CLIENT_REQUEST_TRACE_MASK, 'F', request, "");
                }
                if (request == Request.requestOfDeath) {
                    break;
                }

                // Screen quorum requests against ACLs first
                if (!zks.authWriteRequest(request)) {
                    continue;
                }

                // We want to queue the request to be processed before we submit
                // the request to the leader so that we are ready to receive
                // the response
                maybeSendRequestToNextProcessor(request);

                if (request.isThrottled()) {
                    continue;
                }

                // We now ship the request to the leader. As with all
                // other quorum operations, sync also follows this code
                // path, but different from others, we need to keep track
                // of the sync operations this follower has pending, so we
                // add it to pendingSyncs.
                switch (request.type) {
                case OpCode.sync:
                    zks.pendingSyncs.add(request);
                    zks.getFollower().request(request);
                    break;
                case OpCode.create:
                case OpCode.create2:
                case OpCode.createTTL:
                case OpCode.createContainer:
                case OpCode.delete:
                case OpCode.deleteContainer:
                case OpCode.setData:
                case OpCode.reconfig:
                case OpCode.setACL:
                case OpCode.multi:
                case OpCode.check:
                    zks.getFollower().request(request);
                    break;
                case OpCode.createSession:
                case OpCode.closeSession:
                    // Don't forward local sessions to the leader.
                    if (!request.isLocalSession()) {
                        zks.getFollower().request(request);
                    }
                    break;
                }
            }
        } catch (RuntimeException e) { // spotbugs require explicit catch of RuntimeException
            handleException(this.getName(), e);
        } catch (Exception e) {
            handleException(this.getName(), e);
        }
        LOG.info("FollowerRequestProcessor exited loop!");
    }
```



## Follower处理客户端请求

org.apache.zookeeper.server.ZooKeeperServer#submitRequestNow



请求是由firstProcessor处理的

```java
    public void submitRequestNow(Request si) {
            touch(si.cnxn);
            boolean validpacket = Request.isValid(si.type);
            if (validpacket) {
                setLocalSessionFlag(si);
                firstProcessor.processRequest(si);
                if (si.cnxn != null) {
                    incInProcess();
                }
            } else {
                LOG.warn("Received packet at server of unknown type {}", si.type);
                // Update request accounting/throttling limits
                requestFinished(si);
                new UnimplementedRequestProcessor().processRequest(si);
            }
    }
```



组装firstProcessor

org.apache.zookeeper.server.quorum.FollowerZooKeeperServer#setupRequestProcessors

```java
    @Override
    protected void setupRequestProcessors() {
        RequestProcessor finalProcessor = new FinalRequestProcessor(this);
        commitProcessor = new CommitProcessor(finalProcessor, Long.toString(getServerId()), true, getZooKeeperServerListener());
        commitProcessor.start();
        firstProcessor = new FollowerRequestProcessor(this, commitProcessor);
        ((FollowerRequestProcessor) firstProcessor).start();
        syncProcessor = new SyncRequestProcessor(this, new SendAckRequestProcessor(getFollower()));
        syncProcessor.start();
    }
```









```
FollowerRequestProcessor --> CommitProcessor --> FinalRequestProcessor
firstProcessor               commitProcessor


SyncRequestProcessor    --> SendAckRequestProcessor
syncProcessor
```



客户端请求处理流程

```
FollowerRequestProcessor 将请求放在queuedRequest队列中，自己的run方法消费队列中的请求
```



```sequence
client -> ZooKeeperServer: 发送request
ZooKeeperServer -> ZooKeeperServer: 接收请求submitRequestNow
ZooKeeperServer -> FollowerRequestProcessor: 转发请求processRequest
FollowerRequestProcessor -> FollowerRequestProcessor: 提交请求到queuedRequest中\n如果有必要，增加创建全局session的请求

FollowerRequestProcessor -> CommitProcessor: 转发请求processRequest
FollowerRequestProcessor -> Follower: 如果是写请求，将请求转发给Follower\nFollower会转发给Leader

```







## 会话功能原理



```java
    long createSession(ServerCnxn cnxn, byte[] passwd, int timeout) {
        if (passwd == null) {
            // Possible since it's just deserialized from a packet on the wire.
            passwd = new byte[0];
        }
        long sessionId = sessionTracker.createSession(timeout);
        Random r = new Random(sessionId ^ superSecret);
        r.nextBytes(passwd);
        ByteBuffer to = ByteBuffer.allocate(4);
        to.putInt(timeout);
        cnxn.setSessionId(sessionId);
        Request si = new Request(cnxn, sessionId, 0, OpCode.createSession, to, null);
        submitRequest(si);
        return sessionId;
    }
```



当follower处理创建临时节点时，会将session upgrade到leader

```java
    private Request makeUpgradeRequest(long sessionId) {
        // Make sure to atomically check local session status, upgrade
        // session, and make the session creation request.  This is to
        // avoid another thread upgrading the session in parallel.
        synchronized (upgradeableSessionTracker) {
            if (upgradeableSessionTracker.isLocalSession(sessionId)) {
                int timeout = upgradeableSessionTracker.upgradeSession(sessionId);
                ByteBuffer to = ByteBuffer.allocate(4);
                to.putInt(timeout);
                return new Request(null, sessionId, 0, OpCode.createSession, to, null);
            }
        }
        return null;
    }
```



## Leader组装Processor

firstProcessor用于处理客户端请求

```java
    protected void setupRequestProcessors() {
        RequestProcessor finalProcessor = new FinalRequestProcessor(this);
        RequestProcessor toBeAppliedProcessor = new Leader.ToBeAppliedRequestProcessor(finalProcessor, getLeader());
        commitProcessor = new CommitProcessor(toBeAppliedProcessor, Long.toString(getServerId()), false, getZooKeeperServerListener());
        commitProcessor.start();
        ProposalRequestProcessor proposalProcessor = new ProposalRequestProcessor(this, commitProcessor);
        proposalProcessor.initialize();
        prepRequestProcessor = new PrepRequestProcessor(this, proposalProcessor);
        prepRequestProcessor.start();
        firstProcessor = new LeaderRequestProcessor(this, prepRequestProcessor);

        setupContainerManager();
    }
```



从其他Follower和Observer上来的请求，由prepRequestProcessor

```java
    public void submitLearnerRequest(Request request) {
        /*
         * Requests coming from the learner should have gone through
         * submitRequest() on each server which already perform some request
         * validation, so we don't need to do it again.
         *
         * Additionally, LearnerHandler should start submitting requests into
         * the leader's pipeline only when the leader's server is started, so we
         * can submit the request directly into PrepRequestProcessor.
         *
         * This is done so that requests from learners won't go through
         * LeaderRequestProcessor which perform local session upgrade.
         */
        prepRequestProcessor.processRequest(request);
    }
```





## 连接使用的数据结构

### 客户端刚连接到服务端时，服务端发送

1. timeout：session的超时时间
2. sessionId：sessionId，唯一表示会话
3. passwd：session的密码，由服务端生成

```java
public class ConnectResponse implements Record {
  private int protocolVersion;
  private int timeOut;
  private long sessionId;
  private byte[] passwd;
}
  
```



```java
public class ConnectRequest implements Record {
  private int protocolVersion;
  private long lastZxidSeen;
  private int timeOut;
  private long sessionId;
  private byte[] passwd;
}
  
```



```java
public class RequestHeader implements Record {
  private int xid; // 客户端生成的递增整数
  private int type; // 请求类型
} 
```



```java
public class ReplyHeader implements Record {
  private int xid; // 请求中的xid
  private long zxid; // 服务端处理请求的id，也是递增id
  private int err; // 错误码
}
```



### sessionId初始值

org.apache.zookeeper.server.SessionTrackerImpl#initializeNextSessionId

```java
public static long initializeNextSessionId(long id) {
    long nextSid;
    nextSid = (Time.currentElapsedTime() << 24) >>> 8;
    nextSid = nextSid | (id << 56);
    if (nextSid == EphemeralType.CONTAINER_EPHEMERAL_OWNER) {
        ++nextSid;  // this is an unlikely edge case, but check it just in case
    }
    return nextSid;
}
```



### 服务端的sessionTimeout

服务端的sessionTimeout存在边界，如果客户端sessionTimeout在不在边界内，将会使用边界

### clientCnxn

Xid: 递增整数，表示包的顺序，放在请求头中



### 客户端请求与向leader请求的转换

org.apache.zookeeper.server.quorum.Learner#request

```java
    void request(Request request) throws IOException {
        if (request.isThrottled()) {
            LOG.error("Throttled request sent to leader: {}. Exiting", request);
            ServiceUtils.requestSystemExit(ExitCode.UNEXPECTED_ERROR.getValue());
        }
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream oa = new DataOutputStream(baos);
        oa.writeLong(request.sessionId);
        oa.writeInt(request.cxid);
        oa.writeInt(request.type);
        if (request.request != null) {
            request.request.rewind();
            int len = request.request.remaining();
            byte[] b = new byte[len];
            request.request.get(b);
            request.request.rewind();
            oa.write(b);
        }
        oa.close();
        QuorumPacket qp = new QuorumPacket(Leader.REQUEST, -1, baos.toByteArray(), request.authInfo);
        writePacket(qp, true);
    }
```





### 服务端处理连接

org.apache.zookeeper.server.ZooKeeperServer#processConnectRequest



### watcher管理



setData注册的watcher

放置到dataWatchers中



existed注册的watcher

放置到dataWatchers中



getChildren注册的watcher

放置在childWatches中



采用addWatcher注册的watcher

放置到dataWatchers和childWatchers





创建节点触发

```java
        dataWatches.triggerWatch(path, Event.EventType.NodeCreated);
        childWatches.triggerWatch(parentName.equals("") ? "/" : parentName, Event.EventType.NodeChildrenChanged);
```



修改节点数据

```java
        dataWatches.triggerWatch(path, EventType.NodeDataChanged);
```



删除节点



```
        WatcherOrBitSet processed = dataWatches.triggerWatch(path, EventType.NodeDeleted);
        childWatches.triggerWatch(path, EventType.NodeDeleted, processed);
        childWatches.triggerWatch("".equals(parentName) ? "/" : parentName, EventType.NodeChildrenChanged);
```





