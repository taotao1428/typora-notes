# Kafka实现原理



## 启动过程

### ZkConfigRepository

```
configRepository = new ZkConfigRepository(new AdminZkClient(zkClient))
```



### FinalizedFeatureChangeListener

通过监听zookeeper事件，更新本地的api特性

```scala

/* initialize features */
_featureChangeListener = new FinalizedFeatureChangeListener(featureCache, _zkClient)
if (config.isFeatureVersioningSupported) {
  _featureChangeListener.initOrThrow(config.zkConnectionTimeoutMs)
}
```



### KafkaConfig

kafka的配置会跟随zookeeper自动变化



```scala
        // initialize dynamic broker configs from ZooKeeper. Any updates made after this will be
        // applied after DynamicConfigManager starts.
        config.dynamicConfig.initialize(Some(zkClient))
```



### KafkaScheduler

用于延时或者定时执行任务，里面包含一个线程池

```scala
        /* start scheduler */
        kafkaScheduler = new KafkaScheduler(config.backgroundThreads)
        kafkaScheduler.startup()
```



### LogManager

日志管理

```scala
        /* start log manager */
        _logManager = LogManager(config, initialOfflineDirs,
          new ZkConfigRepository(new AdminZkClient(zkClient)),
          kafkaScheduler, time, brokerTopicStats, logDirFailureChannel, config.usesTopicId)
        _brokerState = BrokerState.RECOVERY
        logManager.startup(zkClient.getAllTopicsInCluster())
```





### zkMetadataCache

用于保存元数据， 包括节点controllerId的信息。由controller通知其他节点更新

```scala
metadataCache = MetadataCache.zkMetadataCache(config.brokerId)
```



### BrokerToControllerChannelManager

管理节点到controller的通信，可以发送消息给controller。

```
        clientToControllerChannelManager = BrokerToControllerChannelManager(
          controllerNodeProvider = MetadataCacheControllerNodeProvider(config, metadataCache),
          time = time,
          metrics = metrics,
          config = config,
          channelName = "forwarding",
          threadNamePrefix = threadNamePrefix,
          retryTimeoutMs = config.requestTimeoutMs.longValue)
        clientToControllerChannelManager.start()
```



### ApiVersionManager

处理api版本相关的请求

```
        val apiVersionManager = ApiVersionManager(
          ListenerType.ZK_BROKER,
          config,
          forwardingManager,
          brokerFeatures,
          featureCache
        )
```



### SocketServer

启动服务

```scala
socketServer = new SocketServer(config, metrics, time, credentialProvider, apiVersionManager)
socketServer.startup(startProcessingRequests = false)
```



### AlterIsrManager

动态向controller，更新isr列表

```
        /* start replica manager */
        alterIsrManager = if (config.interBrokerProtocolVersion.isAlterIsrSupported) {
          AlterIsrManager(
            config = config,
            metadataCache = metadataCache,
            scheduler = kafkaScheduler,
            time = time,
            metrics = metrics,
            threadNamePrefix = threadNamePrefix,
            brokerEpochSupplier = () => kafkaController.brokerEpoch,
            config.brokerId
          )
        } else {
          AlterIsrManager(kafkaScheduler, time, zkClient)
        }
        alterIsrManager.start()
```





### ReplicaManager

创建复制管理器，用于处理复制相关事情

```scala
        _replicaManager = createReplicaManager(isShuttingDown)
        replicaManager.startup()
```



### DelegationTokenManager

```scala
        /* start token manager */
        tokenManager = new DelegationTokenManager(config, tokenCache, time , zkClient)
        tokenManager.startup()
```



### KafkaController

控制服务

```scala
        /* start kafka controller */
        _kafkaController = new KafkaController(config, zkClient, time, metrics, brokerInfo, brokerEpoch, tokenManager, brokerFeatures, featureCache, threadNamePrefix)
        kafkaController.startup()
```



### ZkAdminManager



```scala
adminManager = new ZkAdminManager(config, metrics, metadataCache, zkClient)
```



### GroupCoordinator

消费组协调者

```scala
        /* start group coordinator */
        // Hardcode Time.SYSTEM for now as some Streams tests fail otherwise, it would be good to fix the underlying issue
        groupCoordinator = GroupCoordinator(config, replicaManager, Time.SYSTEM, metrics)
        groupCoordinator.startup(() => zkClient.getTopicPartitionCount(Topic.GROUP_METADATA_TOPIC_NAME).getOrElse(config.offsetsTopicPartitions))
```



### ProducerIdManager

用于生成生产者的id

```scala
        /* create producer ids manager */
        val producerIdManager = if (config.interBrokerProtocolVersion.isAllocateProducerIdsSupported) {
          ProducerIdManager.rpc(
            config.brokerId,
            brokerEpochSupplier = () => kafkaController.brokerEpoch,
            clientToControllerChannelManager,
            config.requestTimeoutMs
          )
        } else {
          ProducerIdManager.zk(config.brokerId, zkClient)
        }
```



### TransactionCoordinator

用于协调事务。类似于GroupCoordinator

```scala
        /* start transaction coordinator, with a separate background thread scheduler for transaction expiration and log loading */
        transactionCoordinator = TransactionCoordinator(config, replicaManager, new KafkaScheduler(threads = 1, threadNamePrefix = "transaction-log-manager-"),
          () => producerIdManager, metrics, metadataCache, Time.SYSTEM)
        transactionCoordinator.startup(
          () => zkClient.getTopicPartitionCount(Topic.TRANSACTION_STATE_TOPIC_NAME).getOrElse(config.transactionTopicPartitions))
```



### AutoTopicCreationManager

```scala
/* start auto topic creation manager */
this.autoTopicCreationManager = AutoTopicCreationManager(
  config,
  metadataCache,
  threadNamePrefix,
  autoTopicCreationChannel,
  Some(adminManager),
  Some(kafkaController),
  groupCoordinator,
  transactionCoordinator
)
```



### FetchManager

复制相关

```scala
val fetchManager = new FetchManager(Time.SYSTEM,
  new FetchSessionCache(config.maxIncrementalFetchSessionCacheSlots,
    KafkaServer.MIN_INCREMENTAL_FETCH_SESSION_EVICTION_MS))
```



### 处理请求

```scala
        /* start processing requests */
        val zkSupport = ZkSupport(adminManager, kafkaController, zkClient, forwardingManager, metadataCache)

        def createKafkaApis(requestChannel: RequestChannel): KafkaApis = new KafkaApis(
          requestChannel = requestChannel,
          metadataSupport = zkSupport,
          replicaManager = replicaManager,
          groupCoordinator = groupCoordinator,
          txnCoordinator = transactionCoordinator,
          autoTopicCreationManager = autoTopicCreationManager,
          brokerId = config.brokerId,
          config = config,
          configRepository = configRepository,
          metadataCache = metadataCache,
          metrics = metrics,
          authorizer = authorizer,
          quotas = quotaManagers,
          fetchManager = fetchManager,
          brokerTopicStats = brokerTopicStats,
          clusterId = clusterId,
          time = time,
          tokenManager = tokenManager,
          apiVersionManager = apiVersionManager)

        dataPlaneRequestProcessor = createKafkaApis(socketServer.dataPlaneRequestChannel)

        dataPlaneRequestHandlerPool = new KafkaRequestHandlerPool(config.brokerId, socketServer.dataPlaneRequestChannel, dataPlaneRequestProcessor, time,
          config.numIoThreads, s"${SocketServer.DataPlaneMetricPrefix}RequestHandlerAvgIdlePercent", SocketServer.DataPlaneThreadPrefix)

        socketServer.controlPlaneRequestChannelOpt.foreach { controlPlaneRequestChannel =>
          controlPlaneRequestProcessor = createKafkaApis(controlPlaneRequestChannel)
          controlPlaneRequestHandlerPool = new KafkaRequestHandlerPool(config.brokerId, socketServer.controlPlaneRequestChannelOpt.get, controlPlaneRequestProcessor, time,
            1, s"${SocketServer.ControlPlaneMetricPrefix}RequestHandlerAvgIdlePercent", SocketServer.ControlPlaneThreadPrefix)
        }
```





### DynamicConfigManager

```scala
/* Add all reconfigurables for config change notification before starting config handlers */
config.dynamicConfig.addReconfigurables(this)

/* start dynamic config manager */
dynamicConfigHandlers = Map[String, ConfigHandler](ConfigType.Topic -> new TopicConfigHandler(logManager, config, quotaManagers, Some(kafkaController)),
                                                   ConfigType.Client -> new ClientIdConfigHandler(quotaManagers),
                                                   ConfigType.User -> new UserConfigHandler(quotaManagers, credentialProvider),
                                                   ConfigType.Broker -> new BrokerConfigHandler(config, quotaManagers),
                                                   ConfigType.Ip -> new IpConfigHandler(socketServer.connectionQuotas))

// Create the config manager. start listening to notifications
dynamicConfigManager = new DynamicConfigManager(zkClient, dynamicConfigHandlers)
dynamicConfigManager.startup()
```



## 机制

### 重新配置机制

kafka.server.DynamicConfigManager#DynamicConfigManager

监听/config/changes/的child变化，每个节点代表一个通知事件。节点是PERSIST_SEQUENCE类型，内容格式如下

```json
{"version" : 2, "entity_path":"entity_type/entity_name"}
```



不同entity的handler如下

```scala
ConfigType.Topic -> new TopicConfigHandler(logManager, config, quotaManagers, Some(kafkaController)),
ConfigType.Client -> new ClientIdConfigHandler(quotaManagers),
ConfigType.User -> new UserConfigHandler(quotaManagers, credentialProvider),
ConfigType.Broker -> new BrokerConfigHandler(config, quotaManagers),
ConfigType.Ip -> new IpConfigHandler(socketServer.connectionQuotas)
```



当节配置更新后，也会通知对象

kafka.server.DynamicBrokerConfig#addReconfigurables

```scala
  def addReconfigurables(kafkaServer: KafkaBroker): Unit = {
    kafkaServer.authorizer match {
      case Some(authz: Reconfigurable) => addReconfigurable(authz)
      case _ =>
    }
    addReconfigurable(kafkaServer.kafkaYammerMetrics)
    addReconfigurable(new DynamicMetricsReporters(kafkaConfig.brokerId, kafkaServer))
    addReconfigurable(new DynamicClientQuotaCallback(kafkaConfig.brokerId, kafkaServer))

    addBrokerReconfigurable(new DynamicThreadPool(kafkaServer))
    if (kafkaServer.logManager.cleaner != null)
      addBrokerReconfigurable(kafkaServer.logManager.cleaner)
    addBrokerReconfigurable(new DynamicLogConfig(kafkaServer.logManager, kafkaServer))
    addBrokerReconfigurable(new DynamicListenerConfig(kafkaServer))
    addBrokerReconfigurable(kafkaServer.socketServer)
  }
```



### 客户端请求处理

kafka会监听listeners指定的ip和端口，然后开始接收客户端和其他broker的请求



#### 端口配置

1.  ` listeners `指定所有的listener。`listener_name://地址:端口`。多个listener使用逗号分隔
2.  `control.plane.listener.name`指定control平面使用的listener。如果指定，对应的listener将被用于控制平面，其他listener为数据平面。**通常不指定控制平面的listener**。
3. `listener.security.protocol.map`指定listener对应的协议
4. `num.network.threads`指定数据平面acceptor对应的Processor数（线程数），Processor用于接收请求和发送响应
5. `num.io.threads`指定处理器请求的线程数，KafkaRequestHandler



```properties
# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
#advertised.listeners=PLAINTEXT://your.host.name:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
```





#### 监听端口

整个监听端口的事情是由kafka.network.SocketServer完成。

1. 所有的请求会分两个平面：控制平面和数据平面。控制平面用于接收其他broker的控制请求，数据平面用于接收客户端的数据请求
2. 每个listener有一个单独的Acceptor。
3. 控制平面的每个Acceptor只有1个Processor。数据平面每个Acceptor可以有多个Processor并行处理请求，n由参数`num.network.threads`指定。



```scala
// kafka.network.SocketServer#createDataPlaneAcceptorsAndProcessors
private def createDataPlaneAcceptorsAndProcessors(dataProcessorsPerListener: Int,
                                                  endpoints: Seq[EndPoint]): Unit = {
  endpoints.foreach { endpoint =>
    connectionQuotas.addListener(config, endpoint.listenerName)
    val dataPlaneAcceptor = createAcceptor(endpoint, DataPlaneMetricPrefix)
    addDataPlaneProcessors(dataPlaneAcceptor, endpoint, dataProcessorsPerListener)
    dataPlaneAcceptors.put(endpoint, dataPlaneAcceptor)
    info(s"Created data-plane acceptor and processors for endpoint : ${endpoint.listenerName}")
  }
}

private def createControlPlaneAcceptorAndProcessor(endpointOpt: Option[EndPoint]): Unit = {
  endpointOpt.foreach { endpoint =>
    connectionQuotas.addListener(config, endpoint.listenerName)
    val controlPlaneAcceptor = createAcceptor(endpoint, ControlPlaneMetricPrefix)
    val controlPlaneProcessor = newProcessor(nextProcessorId, controlPlaneRequestChannelOpt.get,
                                             connectionQuotas, endpoint.listenerName, endpoint.securityProtocol, memoryPool, isPrivilegedListener = true)
    controlPlaneAcceptorOpt = Some(controlPlaneAcceptor)
    controlPlaneProcessorOpt = Some(controlPlaneProcessor)
    val listenerProcessors = new ArrayBuffer[Processor]()
    listenerProcessors += controlPlaneProcessor
    controlPlaneRequestChannelOpt.foreach(_.addProcessor(controlPlaneProcessor))
    nextProcessorId += 1
    controlPlaneAcceptor.addProcessors(listenerProcessors, ControlPlaneThreadPrefix)
    info(s"Created control-plane acceptor and processor for endpoint : ${endpoint.listenerName}")
  }
}
```



#### 请求处理流程

| 组件                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| RequestChannel      | 保存接收到的响应                                             |
| Processsor          | 用于从Acceptor接收新连接，请求和发送响应。将请求放置到Processor。有多个Processor，每个Processor代表一个线程 |
| KafkaRequestHandler | 从Processor中获取请求，交给KafkaApis处理。有多个KafkaRequestHandler，每个handler代表一个线程 |
| KafkaApis           | 实际的处理请求类                                             |

<img src="/开源框架/kafka/.assert/kafka实现原理/image-20220405114109997.png" alt="image-20220405114109997" style="zoom:50%;" />



### 创建topic

每个topic有下面几个属性

```java
// org.apache.kafka.common.message.CreateTopicsRequestData.CreatableTopic
public static class CreatableTopic {
  String name; // 名称
  int numPartitions; // partition的个数
  short replicationFactor; // 每个partition的副本个数，如果为-1，表示副本个数由assignments指定
  CreatableReplicaAssignmentCollection assignments; // 手动指定partition的副本分配方案，也就是分配到哪个节点
  CreateableTopicConfigCollection configs;
}
```



请求会发送到controller节点上，请求与普通消息一样，由KafkaApis处理



#### 预处理

kafkaApis处理创建topic消息，仅是将topic的信息放置到zookeeper的`/brokers/topic/<topic_name>`节点下，节点数据如下。

```json
{
    "partitions": {
        "0": [
            0
        ],
        "1": [
            0
        ]
    },
    "topic_id": "RQoOQGaARJuI7CYGRuKDgA",
    "adding_replicas": {},
    "removing_replicas": {},
    "version": 3
}
```

里面包含了partition的分配策略，partition的分配策略可以由客户端自动指定，也可以由服务端自动生成。自动分配的策略为：

1. 获取当前存活的broker列表，列表长度为`nBrokers`
2. 在0到nBrokers中，**随机**选择一个`startIndex`和一个`nextReplicaShift`
3. 第`0`个partition的第一个副本，分配给`startIndex`位置的broker，剩余副本分配给剩余`nBrokers-1`个节点。剩余的第`i`个副本在分配给剩余节点`nextReplicaShift+i`位置的broker。
4. 第`k`个parition的第一个副本，分配给`startIndex+k`位置的broker。其他副本分配规则与第一个partition相同。
5. 当分配了`nBrokers`个partition后，`nextReplicaShift`加`1`

```scala
private def assignReplicasToBrokersRackUnaware(nPartitions: Int,
                                               replicationFactor: Int,
                                               brokerList: Iterable[Int],
                                               fixedStartIndex: Int, // 默认-1
                                               startPartitionId: Int // 默认-1
                                              ): Map[Int, Seq[Int]] = {
  val ret = mutable.Map[Int, Seq[Int]]()
  val brokerArray = brokerList.toArray
  val startIndex = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
  var currentPartitionId = math.max(0, startPartitionId)
  var nextReplicaShift = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
  for (_ <- 0 until nPartitions) {
    if (currentPartitionId > 0 && (currentPartitionId % brokerArray.length == 0))
    nextReplicaShift += 1
    val firstReplicaIndex = (currentPartitionId + startIndex) % brokerArray.length
    val replicaBuffer = mutable.ArrayBuffer(brokerArray(firstReplicaIndex))
    for (j <- 0 until replicationFactor - 1)
    replicaBuffer += brokerArray(replicaIndex(firstReplicaIndex, nextReplicaShift, j, brokerArray.length))
    ret.put(currentPartitionId, replicaBuffer)
    currentPartitionId += 1
  }
  ret
}

private def replicaIndex(firstReplicaIndex: Int, secondReplicaShift: Int, replicaIndex: Int, nBrokers: Int): Int = {
  val shift = 1 + (secondReplicaShift + replicaIndex) % (nBrokers - 1)
  (firstReplicaIndex + shift) % nBrokers
}
```



举例

```
有5个broker， 9个partition，每个partition有3个replica



假设：startIndex = 2，nextReplicaShift=3，nBrokers=5

节点为[0,1,2,3,4]
第0个partition
	第0个副本位置为startIndex=2，broker为2
	剩余的节点为[0,1,3,4]
	将2前面的节点调整到最后：[3,4,0,1]
	剩余了6个副本
	剩余的第i(0开始)个副本位置为(nextReplicaShift + i) % (nBrokers - 1)
	计算得到：[1,3]
	总分配为：[2,1,3]

第1个partition
	第0个副本位置为startIndex+1=3，broker为3
	剩余的节点为[0,1,2,4]
	将2前面的节点调整到最后：[4,0,1,2]
	剩余了6个副本
	剩余的第i个副本位置为(nextReplicaShift + i) % (nBrokers - 1)
	计算得到：[2,4]
	总分配为：[3,2,4]

第2，3，4为partition计算方式相同

第5个partition之前已经计算了5个parition，因此nextReplicaShif+1=4，其他计算方式一样

第6，7，8个partition计算方式也一样。
```





#### controller处理



controller接收的事件

```scala
        case TopicChange =>
          processTopicChange()
```

修改本地allTopics

注册监听`/brokers/topics/<topic>`节点数据，注册PartitionModifications事件

对于topic节点已经不存在的topic，将会从本地中移除数据

对于新topic，确保存在topicId，如果没有，生成，并保存到zookeeper上。同时更新到本地的topicNames和topicIds

将新partition放入本地的partitionAssignments



使用PartitionStateMachine处理partition的状态

将新partition的状态设置为NewPartition



使用ReplicStateMachine处理replic的状态

将replic的状态设置为NewReplic



将partition状态转为Online

1. 创建partitions节点，包括leaderAndIsr信息
2. 更新controllerContext的 partitionLeadershipInfo



向leaderAndIsrRequestMap添加到节点的leadership情况。仅限在isr中的broker

向updateMetadataRequestBrokerSet，updateMetadataRequestPartitionInfoMap中添加每个partition的详情。包括offline的replic。updateMetadataRequestBrokerSet中包含所有存活的broker

修改parition的状态为Online



将添加到leaderAndIsrRequestMap和updateMetadataRequestPartitionInfoMap的数据发送给broker。



将replica状态转为Online。



---

##### LeaderAndIsrRequest处理

leaderAndIsrRequestMap的响应

请求中包含

1. 当前partition的leader和isr情况。包含partition是不是新的
2. 存活的leader
3. topicId



##### LeaderAndIsrResponseReceived处理

触发Controller的LeaderAndIsrResponseReceived

`kafka.controller.KafkaController#processLeaderAndIsrResponseReceived`





kafka.controller.PartitionLeaderElectionAlgorithms#offlinePartitionLeaderElection



从响应中提取创建失败的replica，放入到replicasOnOfflineDirs中



leader下线的partition状态改为offline。partitionStates



触发Offline的partition重新上线。操作：重新竞选leader，修改isr。会修改zookeeper和本地leaderAndIsr，并通知其他broker



将下线的replica放置到stopReplicaRequestMap中，告诉节点不要停止该Replica



将下线的replic，从zookeeper上的partition的isr移除，并更新本地的`partitionLeadershipInfo`。此处不会重新选举leader。

通知同partition的其他节点，leaderAndIsr已经更新了

更新本地replicaState为offline



#####  UpdateMetadataRequest处理

修改本地元数据

##### UpdateMetadataResponseReceived处理

无处理

##### StopReplicaRequest处理

停止replica





| ControllerContext属性                                   | 说明                                                 |
| ------------------------------------------------------- | ---------------------------------------------------- |
| `allTopics` `mutable.Set.empty[String]`                 | 保存当前存在的topic，与zookeeper /brokers/topics一致 |
| `partitionAssignments`                                  | 保存分区的分配情况                                   |
| `partitionLeadershipInfo`                               | 保存分区的leadership请情况                           |
| `topicIds`                                              | topic name到id的映射                                 |
| `topicNames`                                            | topic id 到name的映射                                |
| `topicsToBeDeleted`                                     |                                                      |
| `topicsWithDeletionStarted`                             |                                                      |
| `replicasOnOfflineDirs = Map[Int, Set[TopicPartition]]` | 保存每个broker上工作不正常的的partition              |



| ControllerContext属性                                    | 说明              |
| -------------------------------------------------------- | ----------------- |
| `partitionStates = Map[TopicPartition, PartitionState]`  | 放置分区的状态    |
| `replicaStates = Map[PartitionAndReplica, ReplicaState]` | 放置replica的状态 |
|                                                          |                   |
|                                                          |                   |
|                                                          |                   |
|                                                          |                   |



| ControllerChannelManager属性                                 | 说明                                      |
| ------------------------------------------------------------ | ----------------------------------------- |
| `leaderAndIsrRequestMap = Map[Int, Map[TopicPartition, LeaderAndIsrPartitionState]` | key为brokerId，map为分区的leader和isr情况 |
| `updateMetadataRequestBrokerSet = Set[Int]`                  | 需要通知的broker                          |
| `updateMetadataRequestPartitionInfoMap = Map[TopicPartition, UpdateMetadataPartitionState]` | 需要通知的分区变化                        |
| `stopReplicaRequestMap = Map[Int, mutable.Map[TopicPartition, StopReplicaPartitionState]]` | 停止replica的请求                         |
|                                                              |                                           |
|                                                              |                                           |



kafka.controller.AbstractControllerBrokerRequestBatch#sendRequestsToBrokers

发送消息给其他broker。最新的消息

kafka.controller.AbstractControllerBrokerRequestBatch#sendLeaderAndIsrRequest

```scala
//LeaderAndIsrRequest
val leaderAndIsrRequestBuilder = new LeaderAndIsrRequest.Builder(leaderAndIsrRequestVersion, controllerId,
                                                                 controllerEpoch, brokerEpoch, leaderAndIsrPartitionStates.values.toBuffer.asJava, topicIds.asJava, leaders.asJava)
sendRequest(broker, leaderAndIsrRequestBuilder, (r: AbstractResponse) => {
  val leaderAndIsrResponse = r.asInstanceOf[LeaderAndIsrResponse]
  sendEvent(LeaderAndIsrResponseReceived(leaderAndIsrResponse, broker))
})
```



发送UpdateMetadataRequest消息

```scala
val updateMetadataRequestBuilder = new UpdateMetadataRequest.Builder(updateMetadataRequestVersion,
                                                                 controllerId, controllerEpoch, brokerEpoch, partitionStates.asJava, liveBrokers.asJava, topicIds.asJava)
sendRequest(broker, updateMetadataRequestBuilder, (r: AbstractResponse) => {
  val updateMetadataResponse = r.asInstanceOf[UpdateMetadataResponse]
  sendEvent(UpdateMetadataResponseReceived(updateMetadataResponse, broker))
})
```



发送StopReplicaRequest

```scala
val stopReplicaRequestBuilder = new StopReplicaRequest.Builder(
  stopReplicaRequestVersion, controllerId, controllerEpoch, brokerEpoch,
  false, stopReplicaTopicState.values.toBuffer.asJava)
sendRequest(brokerId, stopReplicaRequestBuilder,
  responseCallback(brokerId, tp => partitionStates.get(tp).exists(_.deletePartition)))
```



### controller

```properties
controller.listener.names
```





### 生产消息



#### ack机制

当客户端向broker发送消息时，为了确保消息的可靠性，可以设置ack的个数。也就是当接收到消息的broker个数达到ack的个数，服务端才会响应客户端，消息写入成功。



### 消息复制



### 消费消息

<img src="/开源框架/kafka/.assert/kafka实现原理/image-20220416083125206.png" alt="image-20220416083125206" style="zoom:50%;" />



#### coordinator

客户端

org.apache.kafka.clients.consumer.internals.AbstractCoordinator#sendJoinGroupRequest





### rebalance

如果消费以group消费消息，kafka会通过coordinator协调组内每个消费者消费的partition。当组内的成员或者topic发生变化时，kafka会通过rebalance机制重新分配partition。



https://www.cnblogs.com/listenfwind/p/14146727.html



#### rebalance的流程

rebalance包含下面5个请求。

| 请求          | 说明                                                 |
| ------------- | ---------------------------------------------------- |
| JoinGroup     | 消费者请求加入group                                  |
| SyncGroup     | 将分配方案同步给消费者                               |
| Heartbeat     | 消费者向coordinator发送心跳                          |
| LeaveGroup    | 消费者主动离开group                                  |
| DescribeGroup | 获取group成员、分配和消费情况。主要是admin使用该节点 |





<img src="/开源框架/kafka/.assert/kafka实现原理/image-20220410164338592.png" alt="image-20220410164338592" style="zoom:50%;" />







### 事务

https://blog.csdn.net/qq_36628536/article/details/118089422

![image-20220417113740460](/开源框架/kafka/.assert/kafka实现原理/image-20220417113740460.png)











## zookeeper节点

| 路径                        | 说明                               |
| --------------------------- | ---------------------------------- |
| /consumers                  | 用于存放消费者                     |
| /brokers/topics             | 用于存放节点上主题的信息           |
| `/brokers/topics/<topic>`   | 保存主题的partition情况            |
| `/brokers/ids/<broker_id>`  | 保存节点信息（临时节点）           |
| /config/changes             | 配置修改节点，用于通知配置修改事件 |
| /admin/delete_topics        | 用于存放删除的topics               |
| /brokers/seqids             | 用于存放节点的序列id               |
| /isr_change_notification    | 用于存在isr列表变化通知            |
| /latest_producer_id_block   | 用于记录producer的id范围           |
| /log_dir_event_notification |                                    |
| /config/topics              | 用于保存topic                      |
| /config/clients             | 用于保存客户端                     |
| /config/users               | 用于保存用户                       |
| /config/brokers             | 用于保存节点                       |
| /config/ips                 |                                    |
| /cluster/id                 | 集群id                             |



## 文件

| 文件                             | 说明                                        |
| -------------------------------- | ------------------------------------------- |
| ${log.dirs}/meta.properties      | 保存需要的元数据，例如broker.id，cluster.id |
| ${log.dirs}/${topic}-${patition} | 每个分区的文件                              |
|                                  |                                             |
|                                  |                                             |
|                                  |                                             |
|                                  |                                             |
|                                  |                                             |
|                                  |                                             |
|                                  |                                             |

### 分区的文件结构

![img](/开源框架/kafka/.assert/kafka实现原理/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6ZqP57yY5riF6aOO5q6H,size_20,color_FFFFFF,t_70,g_se,x_16.png)





## 节点状态

```java
_brokerState = BrokerState.STARTING

public enum BrokerState {
    /**
     * The state the broker is in when it first starts up.
     */
    NOT_RUNNING((byte) 0),

    /**
     * The state the broker is in when it is catching up with cluster metadata.
     */
    STARTING((byte) 1),

    /**
     * The broker has caught up with cluster metadata, but has not yet
     * been unfenced by the controller.
     */
    RECOVERY((byte) 2),

    /**
     * The state the broker is in when it has registered at least once, and is
     * accepting client requests.
     */
    RUNNING((byte) 3),

    /**
     * The state the broker is in when it is attempting to perform a controlled
     * shutdown.
     */
    PENDING_CONTROLLED_SHUTDOWN((byte) 6),

    /**
     * The state the broker is in when it is shutting down.
     */
    SHUTTING_DOWN((byte) 7),

    /**
     * The broker is in an unknown state.
     */
    UNKNOWN((byte) 127);
```





## 对象



### Replica

表示远程follower副本

| 属性                          | 说明 |
| ----------------------------- | ---- |
| `_logEndOffsetMetadata`       |      |
| `_logStartOffset`             |      |
| `lastFetchLeaderLogEndOffset` |      |
| `lastFetchTimeMs`             |      |
| `_lastCaughtUpTimeMs`         |      |
| ``                            |      |





### Partition

broker本地管理的分区的类

| 属性                                         | 说明                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| `remoteReplicasMap = new Pool[Int, Replica]` | 保存分区非本地的副本。Replica对象中包含了复制的偏移量和时间差 |
| `assignmentState: AssignmentState`           | 保存分区的副本信息                                           |
| `isrState: IsrState`                         | 保存分区的isr，包括CommittedIsr，PendingShrinkIsr和PendingExpandIsr |
| `log: Option[UnifiedLog]`                    | 保存分区副本与本地对应的日志                                 |
|                                              |                                                              |
|                                              |                                                              |



#### makeLeader

变成leader



#### makeFollower

变成follower





### LogManager

管理log文件









### ReplicaManager

| 属性                                                        | 说明                   |
| ----------------------------------------------------------- | ---------------------- |
| `allPartitions = new Pool[TopicPartition, HostedPartition]` | 保存本地的partition    |
| `replicaFetcherManager:ReplicaFetcherManager`               | 管理follower的复制问题 |
|                                                             |                        |
|                                                             |                        |
|                                                             |                        |
|                                                             |                        |



#### becomeLeaderOrFollower

处理Controller发送过来的LeaderAndIsrRequest



`kafka.server.ReplicaManager#makeLeaders`创建leader



