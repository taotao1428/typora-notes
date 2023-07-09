# kafka介绍

kafka是一款开源的分布式高性能消息队列中间件



## kafka基本概念

| 概念                  | 介绍                                                         |
| --------------------- | ------------------------------------------------------------ |
| producer              | 负责发送消息给kafka                                          |
| consumer              | 负责从kakfa拉取消息消费                                      |
| consumer group        | 每个consumer都属于一个group，如果不指定，将会属于一个默认的group |
| **topic**（主题）     | 主题可以看做是消息的类型，每个消息都有主题。不同topic的消息保存在不同的partition。生产者和消费者通过主题指定要生产和消费的消息 |
| **partition**（分区） | 分区是保存数据的物理概念，每个topic对应1个或多个partition。  |
| offset（偏移量）      | offset表示消费者在partition消费的位置                        |



### 优缺点

优点

1. 数据保存在磁盘上，可以保存更多数据
2. 使用零拷贝技术，性能高

适用于日志，监控指标等处理



缺点

1. 运维比较困难，缺少成熟的运维工具
2. 对zookeeper强依赖



## 搭建kafka集群

### 下载软件包

https://www.apache.org/dyn/closer.cgi?path=/kafka/3.1.0/kafka_2.13-3.1.0.tgz

### 启动

https://kafka.apache.org/quickstart

```shell
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```



### 配置文件

#### zookeeper.properties

zookeeper的配置文件



#### server.properties

kafka配置文件



## kafka使用



### 消费者

#### 消费者配置

`org.apache.kafka.clients.consumer.ConsumerConfig`

<ul class="config-list">
<li>
<h4><a id="key.deserializer"></a><a id="consumerconfigs_key.deserializer" href="#consumerconfigs_key.deserializer">key.deserializer</a></h4>
<p>Deserializer class for key that implements the <code>org.apache.kafka.common.serialization.Deserializer</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td></td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="value.deserializer"></a><a id="consumerconfigs_value.deserializer" href="#consumerconfigs_value.deserializer">value.deserializer</a></h4>
<p>Deserializer class for value that implements the <code>org.apache.kafka.common.serialization.Deserializer</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td></td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="bootstrap.servers"></a><a id="consumerconfigs_bootstrap.servers" href="#consumerconfigs_bootstrap.servers">bootstrap.servers</a></h4>
<p>A list of host/port pairs to use for establishing the initial connection to the Kafka cluster. The client will make use of all servers irrespective of which servers are specified here for bootstrapping&mdash;this list only impacts the initial hosts used to discover the full set of servers. This list should be in the form <code>host1:port1,host2:port2,...</code>. Since these servers are just used for the initial connection to discover the full cluster membership (which may change dynamically), this list need not contain the full set of servers (you may want more than one, though, in case a server is down).</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="fetch.min.bytes"></a><a id="consumerconfigs_fetch.min.bytes" href="#consumerconfigs_fetch.min.bytes">fetch.min.bytes</a></h4>
<p>The minimum amount of data the server should return for a fetch request. If insufficient data is available the request will wait for that much data to accumulate before answering the request. The default setting of 1 byte means that fetch requests are answered as soon as a single byte of data is available or the fetch request times out waiting for data to arrive. Setting this to something greater than 1 will cause the server to wait for larger amounts of data to accumulate which can improve server throughput a bit at the cost of some additional latency.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>1</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="group.id"></a><a id="consumerconfigs_group.id" href="#consumerconfigs_group.id">group.id</a></h4>
<p>A unique string that identifies the consumer group this consumer belongs to. This property is required if the consumer uses either the group management functionality by using <code>subscribe(topic)</code> or the Kafka-based offset management strategy.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="heartbeat.interval.ms"></a><a id="consumerconfigs_heartbeat.interval.ms" href="#consumerconfigs_heartbeat.interval.ms">heartbeat.interval.ms</a></h4>
<p>The expected time between heartbeats to the consumer coordinator when using Kafka's group management facilities. Heartbeats are used to ensure that the consumer's session stays active and to facilitate rebalancing when new consumers join or leave the group. The value must be set lower than <code>session.timeout.ms</code>, but typically should be set no higher than 1/3 of that value. It can be adjusted even lower to control the expected time for normal rebalances.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>3000 (3 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.partition.fetch.bytes"></a><a id="consumerconfigs_max.partition.fetch.bytes" href="#consumerconfigs_max.partition.fetch.bytes">max.partition.fetch.bytes</a></h4>
<p>The maximum amount of data per-partition the server will return. Records are fetched in batches by the consumer. If the first record batch in the first non-empty partition of the fetch is larger than this limit, the batch will still be returned to ensure that the consumer can make progress. The maximum record batch size accepted by the broker is defined via <code>message.max.bytes</code> (broker config) or <code>max.message.bytes</code> (topic config). See fetch.max.bytes for limiting the consumer request size.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>1048576 (1 mebibyte)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="session.timeout.ms"></a><a id="consumerconfigs_session.timeout.ms" href="#consumerconfigs_session.timeout.ms">session.timeout.ms</a></h4>
<p>The timeout used to detect client failures when using Kafka's group management facility. The client sends periodic heartbeats to indicate its liveness to the broker. If no heartbeats are received by the broker before the expiration of this session timeout, then the broker will remove this client from the group and initiate a rebalance. Note that the value must be in the allowable range as configured in the broker configuration by <code>group.min.session.timeout.ms</code> and <code>group.max.session.timeout.ms</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>45000 (45 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.key.password"></a><a id="consumerconfigs_ssl.key.password" href="#consumerconfigs_ssl.key.password">ssl.key.password</a></h4>
<p>The password of the private key in the key store file orthe PEM key specified in `ssl.keystore.key'. This is required for clients only if two-way authentication is configured.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.certificate.chain"></a><a id="consumerconfigs_ssl.keystore.certificate.chain" href="#consumerconfigs_ssl.keystore.certificate.chain">ssl.keystore.certificate.chain</a></h4>
<p>Certificate chain in the format specified by 'ssl.keystore.type'. Default SSL engine factory supports only PEM format with a list of X.509 certificates</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.key"></a><a id="consumerconfigs_ssl.keystore.key" href="#consumerconfigs_ssl.keystore.key">ssl.keystore.key</a></h4>
<p>Private key in the format specified by 'ssl.keystore.type'. Default SSL engine factory supports only PEM format with PKCS#8 keys. If the key is encrypted, key password must be specified using 'ssl.key.password'</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.location"></a><a id="consumerconfigs_ssl.keystore.location" href="#consumerconfigs_ssl.keystore.location">ssl.keystore.location</a></h4>
<p>The location of the key store file. This is optional for client and can be used for two-way authentication for client.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.password"></a><a id="consumerconfigs_ssl.keystore.password" href="#consumerconfigs_ssl.keystore.password">ssl.keystore.password</a></h4>
<p>The store password for the key store file. This is optional for client and only needed if 'ssl.keystore.location' is configured.  Key store password is not supported for PEM format.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.certificates"></a><a id="consumerconfigs_ssl.truststore.certificates" href="#consumerconfigs_ssl.truststore.certificates">ssl.truststore.certificates</a></h4>
<p>Trusted certificates in the format specified by 'ssl.truststore.type'. Default SSL engine factory supports only PEM format with X.509 certificates.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.location"></a><a id="consumerconfigs_ssl.truststore.location" href="#consumerconfigs_ssl.truststore.location">ssl.truststore.location</a></h4>
<p>The location of the trust store file. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.password"></a><a id="consumerconfigs_ssl.truststore.password" href="#consumerconfigs_ssl.truststore.password">ssl.truststore.password</a></h4>
<p>The password for the trust store file. If a password is not set, trust store file configured will still be used, but integrity checking is disabled. Trust store password is not supported for PEM format.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="allow.auto.create.topics"></a><a id="consumerconfigs_allow.auto.create.topics" href="#consumerconfigs_allow.auto.create.topics">allow.auto.create.topics</a></h4>
<p>Allow automatic topic creation on the broker when subscribing to or assigning a topic. A topic being subscribed to will be automatically created only if the broker allows for it using `auto.create.topics.enable` broker configuration. This configuration must be set to `false` when using brokers older than 0.11.0</p>
<table><tbody>
<tr><th>Type:</th><td>boolean</td></tr>
<tr><th>Default:</th><td>true</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="auto.offset.reset"></a><a id="consumerconfigs_auto.offset.reset" href="#consumerconfigs_auto.offset.reset">auto.offset.reset</a></h4>
<p>What to do when there is no initial offset in Kafka or if the current offset does not exist any more on the server (e.g. because that data has been deleted): <ul><li>earliest: automatically reset the offset to the earliest offset<li>latest: automatically reset the offset to the latest offset</li><li>none: throw exception to the consumer if no previous offset is found for the consumer's group</li><li>anything else: throw exception to the consumer.</li></ul></p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>latest</td></tr>
<tr><th>Valid Values:</th><td>[latest, earliest, none]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="client.dns.lookup"></a><a id="consumerconfigs_client.dns.lookup" href="#consumerconfigs_client.dns.lookup">client.dns.lookup</a></h4>
<p>Controls how the client uses DNS lookups. If set to <code>use_all_dns_ips</code>, connect to each returned IP address in sequence until a successful connection is established. After a disconnection, the next IP is used. Once all IPs have been used once, the client resolves the IP(s) from the hostname again (both the JVM and the OS cache DNS name lookups, however). If set to <code>resolve_canonical_bootstrap_servers_only</code>, resolve each bootstrap address into a list of canonical names. After the bootstrap phase, this behaves the same as <code>use_all_dns_ips</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>use_all_dns_ips</td></tr>
<tr><th>Valid Values:</th><td>[use_all_dns_ips, resolve_canonical_bootstrap_servers_only]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="connections.max.idle.ms"></a><a id="consumerconfigs_connections.max.idle.ms" href="#consumerconfigs_connections.max.idle.ms">connections.max.idle.ms</a></h4>
<p>Close idle connections after the number of milliseconds specified by this config.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>540000 (9 minutes)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="default.api.timeout.ms"></a><a id="consumerconfigs_default.api.timeout.ms" href="#consumerconfigs_default.api.timeout.ms">default.api.timeout.ms</a></h4>
<p>Specifies the timeout (in milliseconds) for client APIs. This configuration is used as the default timeout for all client operations that do not specify a <code>timeout</code> parameter.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>60000 (1 minute)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="enable.auto.commit"></a><a id="consumerconfigs_enable.auto.commit" href="#consumerconfigs_enable.auto.commit">enable.auto.commit</a></h4>
<p>If true the consumer's offset will be periodically committed in the background.</p>
<table><tbody>
<tr><th>Type:</th><td>boolean</td></tr>
<tr><th>Default:</th><td>true</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="exclude.internal.topics"></a><a id="consumerconfigs_exclude.internal.topics" href="#consumerconfigs_exclude.internal.topics">exclude.internal.topics</a></h4>
<p>Whether internal topics matching a subscribed pattern should be excluded from the subscription. It is always possible to explicitly subscribe to an internal topic.</p>
<table><tbody>
<tr><th>Type:</th><td>boolean</td></tr>
<tr><th>Default:</th><td>true</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="fetch.max.bytes"></a><a id="consumerconfigs_fetch.max.bytes" href="#consumerconfigs_fetch.max.bytes">fetch.max.bytes</a></h4>
<p>The maximum amount of data the server should return for a fetch request. Records are fetched in batches by the consumer, and if the first record batch in the first non-empty partition of the fetch is larger than this value, the record batch will still be returned to ensure that the consumer can make progress. As such, this is not a absolute maximum. The maximum record batch size accepted by the broker is defined via <code>message.max.bytes</code> (broker config) or <code>max.message.bytes</code> (topic config). Note that the consumer performs multiple fetches in parallel.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>52428800 (50 mebibytes)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="group.instance.id"></a><a id="consumerconfigs_group.instance.id" href="#consumerconfigs_group.instance.id">group.instance.id</a></h4>
<p>A unique identifier of the consumer instance provided by the end user. Only non-empty strings are permitted. If set, the consumer is treated as a static member, which means that only one instance with this ID is allowed in the consumer group at any time. This can be used in combination with a larger session timeout to avoid group rebalances caused by transient unavailability (e.g. process restarts). If not set, the consumer will join the group as a dynamic member, which is the traditional behavior.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="isolation.level"></a><a id="consumerconfigs_isolation.level" href="#consumerconfigs_isolation.level">isolation.level</a></h4>
<p>Controls how to read messages written transactionally. If set to <code>read_committed</code>, consumer.poll() will only return transactional messages which have been committed. If set to <code>read_uncommitted</code> (the default), consumer.poll() will return all messages, even transactional messages which have been aborted. Non-transactional messages will be returned unconditionally in either mode. <p>Messages will always be returned in offset order. Hence, in  <code>read_committed</code> mode, consumer.poll() will only return messages up to the last stable offset (LSO), which is the one less than the offset of the first open transaction. In particular any messages appearing after messages belonging to ongoing transactions will be withheld until the relevant transaction has been completed. As a result, <code>read_committed</code> consumers will not be able to read up to the high watermark when there are in flight transactions.</p><p> Further, when in <code>read_committed</code> the seekToEnd method will return the LSO</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>read_uncommitted</td></tr>
<tr><th>Valid Values:</th><td>[read_committed, read_uncommitted]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.poll.interval.ms"></a><a id="consumerconfigs_max.poll.interval.ms" href="#consumerconfigs_max.poll.interval.ms">max.poll.interval.ms</a></h4>
<p>The maximum delay between invocations of poll() when using consumer group management. This places an upper bound on the amount of time that the consumer can be idle before fetching more records. If poll() is not called before expiration of this timeout, then the consumer is considered failed and the group will rebalance in order to reassign the partitions to another member. For consumers using a non-null <code>group.instance.id</code> which reach this timeout, partitions will not be immediately reassigned. Instead, the consumer will stop sending heartbeats and partitions will be reassigned after expiration of <code>session.timeout.ms</code>. This mirrors the behavior of a static consumer which has shutdown.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>300000 (5 minutes)</td></tr>
<tr><th>Valid Values:</th><td>[1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.poll.records"></a><a id="consumerconfigs_max.poll.records" href="#consumerconfigs_max.poll.records">max.poll.records</a></h4>
<p>The maximum number of records returned in a single call to poll(). Note, that <code>max.poll.records</code> does not impact the underlying fetching behavior. The consumer will cache the records from each fetch request and returns them incrementally from each poll.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>500</td></tr>
<tr><th>Valid Values:</th><td>[1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="partition.assignment.strategy"></a><a id="consumerconfigs_partition.assignment.strategy" href="#consumerconfigs_partition.assignment.strategy">partition.assignment.strategy</a></h4>
<p>A list of class names or class types, ordered by preference, of supported partition assignment strategies that the client will use to distribute partition ownership amongst consumer instances when group management is used. Available options are:<ul><li><code>org.apache.kafka.clients.consumer.RangeAssignor</code>: Assigns partitions on a per-topic basis.</li><li><code>org.apache.kafka.clients.consumer.RoundRobinAssignor</code>: Assigns partitions to consumers in a round-robin fashion.</li><li><code>org.apache.kafka.clients.consumer.StickyAssignor</code>: Guarantees an assignment that is maximally balanced while preserving as many existing partition assignments as possible.</li><li><code>org.apache.kafka.clients.consumer.CooperativeStickyAssignor</code>: Follows the same StickyAssignor logic, but allows for cooperative rebalancing.</li></ul><p>The default assignor is [RangeAssignor, CooperativeStickyAssignor], which will use the RangeAssignor by default, but allows upgrading to the CooperativeStickyAssignor with just a single rolling bounce that removes the RangeAssignor from the list.</p><p>Implementing the <code>org.apache.kafka.clients.consumer.ConsumerPartitionAssignor</code> interface allows you to plug in a custom assignment strategy.</p></p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>class org.apache.kafka.clients.consumer.RangeAssignor,class org.apache.kafka.clients.consumer.CooperativeStickyAssignor</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="receive.buffer.bytes"></a><a id="consumerconfigs_receive.buffer.bytes" href="#consumerconfigs_receive.buffer.bytes">receive.buffer.bytes</a></h4>
<p>The size of the TCP receive buffer (SO_RCVBUF) to use when reading data. If the value is -1, the OS default will be used.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>65536 (64 kibibytes)</td></tr>
<tr><th>Valid Values:</th><td>[-1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="request.timeout.ms"></a><a id="consumerconfigs_request.timeout.ms" href="#consumerconfigs_request.timeout.ms">request.timeout.ms</a></h4>
<p>The configuration controls the maximum amount of time the client will wait for the response of a request. If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.client.callback.handler.class"></a><a id="consumerconfigs_sasl.client.callback.handler.class" href="#consumerconfigs_sasl.client.callback.handler.class">sasl.client.callback.handler.class</a></h4>
<p>The fully qualified name of a SASL client callback handler class that implements the AuthenticateCallbackHandler interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.jaas.config"></a><a id="consumerconfigs_sasl.jaas.config" href="#consumerconfigs_sasl.jaas.config">sasl.jaas.config</a></h4>
<p>JAAS login context parameters for SASL connections in the format used by JAAS configuration files. JAAS configuration file format is described <a href="http://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html">here</a>. The format for the value is: <code>loginModuleClass controlFlag (optionName=optionValue)*;</code>. For brokers, the config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=com.example.ScramLoginModule required;</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.service.name"></a><a id="consumerconfigs_sasl.kerberos.service.name" href="#consumerconfigs_sasl.kerberos.service.name">sasl.kerberos.service.name</a></h4>
<p>The Kerberos principal name that Kafka runs as. This can be defined either in Kafka's JAAS config or in Kafka's config.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.callback.handler.class"></a><a id="consumerconfigs_sasl.login.callback.handler.class" href="#consumerconfigs_sasl.login.callback.handler.class">sasl.login.callback.handler.class</a></h4>
<p>The fully qualified name of a SASL login callback handler class that implements the AuthenticateCallbackHandler interface. For brokers, login callback handler config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.login.callback.handler.class=com.example.CustomScramLoginCallbackHandler</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.class"></a><a id="consumerconfigs_sasl.login.class" href="#consumerconfigs_sasl.login.class">sasl.login.class</a></h4>
<p>The fully qualified name of a class that implements the Login interface. For brokers, login config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.login.class=com.example.CustomScramLogin</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.mechanism"></a><a id="consumerconfigs_sasl.mechanism" href="#consumerconfigs_sasl.mechanism">sasl.mechanism</a></h4>
<p>SASL mechanism used for client connections. This may be any mechanism for which a security provider is available. GSSAPI is the default mechanism.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>GSSAPI</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.url"></a><a id="consumerconfigs_sasl.oauthbearer.jwks.endpoint.url" href="#consumerconfigs_sasl.oauthbearer.jwks.endpoint.url">sasl.oauthbearer.jwks.endpoint.url</a></h4>
<p>The OAuth/OIDC provider URL from which the provider's <a href="https://datatracker.ietf.org/doc/html/rfc7517#section-5">JWKS (JSON Web Key Set)</a> can be retrieved. The URL can be HTTP(S)-based or file-based. If the URL is HTTP(S)-based, the JWKS data will be retrieved from the OAuth/OIDC provider via the configured URL on broker startup. All then-current keys will be cached on the broker for incoming requests. If an authentication request is received for a JWT that includes a "kid" header claim value that isn't yet in the cache, the JWKS endpoint will be queried again on demand. However, the broker polls the URL every sasl.oauthbearer.jwks.endpoint.refresh.ms milliseconds to refresh the cache with any forthcoming keys before any JWT requests that include them are received. If the URL is file-based, the broker will load the JWKS file from a configured location on startup. In the event that the JWT includes a "kid" header value that isn't in the JWKS file, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.token.endpoint.url"></a><a id="consumerconfigs_sasl.oauthbearer.token.endpoint.url" href="#consumerconfigs_sasl.oauthbearer.token.endpoint.url">sasl.oauthbearer.token.endpoint.url</a></h4>
<p>The URL for the OAuth/OIDC identity provider. If the URL is HTTP(S)-based, it is the issuer's token endpoint URL to which requests will be made to login based on the configuration in sasl.jaas.config. If the URL is file-based, it specifies a file containing an access token (in JWT serialized form) issued by the OAuth/OIDC identity provider to use for authorization.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="security.protocol"></a><a id="consumerconfigs_security.protocol" href="#consumerconfigs_security.protocol">security.protocol</a></h4>
<p>Protocol used to communicate with brokers. Valid values are: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>PLAINTEXT</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="send.buffer.bytes"></a><a id="consumerconfigs_send.buffer.bytes" href="#consumerconfigs_send.buffer.bytes">send.buffer.bytes</a></h4>
<p>The size of the TCP send buffer (SO_SNDBUF) to use when sending data. If the value is -1, the OS default will be used.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>131072 (128 kibibytes)</td></tr>
<tr><th>Valid Values:</th><td>[-1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="socket.connection.setup.timeout.max.ms"></a><a id="consumerconfigs_socket.connection.setup.timeout.max.ms" href="#consumerconfigs_socket.connection.setup.timeout.max.ms">socket.connection.setup.timeout.max.ms</a></h4>
<p>The maximum amount of time the client will wait for the socket connection to be established. The connection setup timeout will increase exponentially for each consecutive connection failure up to this maximum. To avoid connection storms, a randomization factor of 0.2 will be applied to the timeout resulting in a random range between 20% below and 20% above the computed value.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="socket.connection.setup.timeout.ms"></a><a id="consumerconfigs_socket.connection.setup.timeout.ms" href="#consumerconfigs_socket.connection.setup.timeout.ms">socket.connection.setup.timeout.ms</a></h4>
<p>The amount of time the client will wait for the socket connection to be established. If the connection is not built before the timeout elapses, clients will close the socket channel.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.enabled.protocols"></a><a id="consumerconfigs_ssl.enabled.protocols" href="#consumerconfigs_ssl.enabled.protocols">ssl.enabled.protocols</a></h4>
<p>The list of protocols enabled for SSL connections. The default is 'TLSv1.2,TLSv1.3' when running with Java 11 or newer, 'TLSv1.2' otherwise. With the default value for Java 11, clients and servers will prefer TLSv1.3 if both support it and fallback to TLSv1.2 otherwise (assuming both support at least TLSv1.2). This default should be fine for most cases. Also see the config documentation for `ssl.protocol`.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>TLSv1.2,TLSv1.3</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.type"></a><a id="consumerconfigs_ssl.keystore.type" href="#consumerconfigs_ssl.keystore.type">ssl.keystore.type</a></h4>
<p>The file format of the key store file. This is optional for client.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>JKS</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.protocol"></a><a id="consumerconfigs_ssl.protocol" href="#consumerconfigs_ssl.protocol">ssl.protocol</a></h4>
<p>The SSL protocol used to generate the SSLContext. The default is 'TLSv1.3' when running with Java 11 or newer, 'TLSv1.2' otherwise. This value should be fine for most use cases. Allowed values in recent JVMs are 'TLSv1.2' and 'TLSv1.3'. 'TLS', 'TLSv1.1', 'SSL', 'SSLv2' and 'SSLv3' may be supported in older JVMs, but their usage is discouraged due to known security vulnerabilities. With the default value for this config and 'ssl.enabled.protocols', clients will downgrade to 'TLSv1.2' if the server does not support 'TLSv1.3'. If this config is set to 'TLSv1.2', clients will not use 'TLSv1.3' even if it is one of the values in ssl.enabled.protocols and the server only supports 'TLSv1.3'.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>TLSv1.3</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.provider"></a><a id="consumerconfigs_ssl.provider" href="#consumerconfigs_ssl.provider">ssl.provider</a></h4>
<p>The name of the security provider used for SSL connections. Default value is the default security provider of the JVM.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.type"></a><a id="consumerconfigs_ssl.truststore.type" href="#consumerconfigs_ssl.truststore.type">ssl.truststore.type</a></h4>
<p>The file format of the trust store file.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>JKS</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="auto.commit.interval.ms"></a><a id="consumerconfigs_auto.commit.interval.ms" href="#consumerconfigs_auto.commit.interval.ms">auto.commit.interval.ms</a></h4>
<p>The frequency in milliseconds that the consumer offsets are auto-committed to Kafka if <code>enable.auto.commit</code> is set to <code>true</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>5000 (5 seconds)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="check.crcs"></a><a id="consumerconfigs_check.crcs" href="#consumerconfigs_check.crcs">check.crcs</a></h4>
<p>Automatically check the CRC32 of the records consumed. This ensures no on-the-wire or on-disk corruption to the messages occurred. This check adds some overhead, so it may be disabled in cases seeking extreme performance.</p>
<table><tbody>
<tr><th>Type:</th><td>boolean</td></tr>
<tr><th>Default:</th><td>true</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="client.id"></a><a id="consumerconfigs_client.id" href="#consumerconfigs_client.id">client.id</a></h4>
<p>An id string to pass to the server when making requests. The purpose of this is to be able to track the source of requests beyond just ip/port by allowing a logical application name to be included in server-side request logging.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="client.rack"></a><a id="consumerconfigs_client.rack" href="#consumerconfigs_client.rack">client.rack</a></h4>
<p>A rack identifier for this client. This can be any string value which indicates where this client is physically located. It corresponds with the broker config 'broker.rack'</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="fetch.max.wait.ms"></a><a id="consumerconfigs_fetch.max.wait.ms" href="#consumerconfigs_fetch.max.wait.ms">fetch.max.wait.ms</a></h4>
<p>The maximum amount of time the server will block before answering the fetch request if there isn't sufficient data to immediately satisfy the requirement given by fetch.min.bytes.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>500</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="interceptor.classes"></a><a id="consumerconfigs_interceptor.classes" href="#consumerconfigs_interceptor.classes">interceptor.classes</a></h4>
<p>A list of classes to use as interceptors. Implementing the <code>org.apache.kafka.clients.consumer.ConsumerInterceptor</code> interface allows you to intercept (and possibly mutate) records received by the consumer. By default, there are no interceptors.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metadata.max.age.ms"></a><a id="consumerconfigs_metadata.max.age.ms" href="#consumerconfigs_metadata.max.age.ms">metadata.max.age.ms</a></h4>
<p>The period of time in milliseconds after which we force a refresh of metadata even if we haven't seen any partition leadership changes to proactively discover any new brokers or partitions.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>300000 (5 minutes)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metric.reporters"></a><a id="consumerconfigs_metric.reporters" href="#consumerconfigs_metric.reporters">metric.reporters</a></h4>
<p>A list of classes to use as metrics reporters. Implementing the <code>org.apache.kafka.common.metrics.MetricsReporter</code> interface allows plugging in classes that will be notified of new metric creation. The JmxReporter is always included to register JMX statistics.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.num.samples"></a><a id="consumerconfigs_metrics.num.samples" href="#consumerconfigs_metrics.num.samples">metrics.num.samples</a></h4>
<p>The number of samples maintained to compute metrics.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>2</td></tr>
<tr><th>Valid Values:</th><td>[1,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.recording.level"></a><a id="consumerconfigs_metrics.recording.level" href="#consumerconfigs_metrics.recording.level">metrics.recording.level</a></h4>
<p>The highest recording level for metrics.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>INFO</td></tr>
<tr><th>Valid Values:</th><td>[INFO, DEBUG, TRACE]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.sample.window.ms"></a><a id="consumerconfigs_metrics.sample.window.ms" href="#consumerconfigs_metrics.sample.window.ms">metrics.sample.window.ms</a></h4>
<p>The window of time a metrics sample is computed over.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="reconnect.backoff.max.ms"></a><a id="consumerconfigs_reconnect.backoff.max.ms" href="#consumerconfigs_reconnect.backoff.max.ms">reconnect.backoff.max.ms</a></h4>
<p>The maximum amount of time in milliseconds to wait when reconnecting to a broker that has repeatedly failed to connect. If provided, the backoff per host will increase exponentially for each consecutive connection failure, up to this maximum. After calculating the backoff increase, 20% random jitter is added to avoid connection storms.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>1000 (1 second)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="reconnect.backoff.ms"></a><a id="consumerconfigs_reconnect.backoff.ms" href="#consumerconfigs_reconnect.backoff.ms">reconnect.backoff.ms</a></h4>
<p>The base amount of time to wait before attempting to reconnect to a given host. This avoids repeatedly connecting to a host in a tight loop. This backoff applies to all connection attempts by the client to a broker.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>50</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="retry.backoff.ms"></a><a id="consumerconfigs_retry.backoff.ms" href="#consumerconfigs_retry.backoff.ms">retry.backoff.ms</a></h4>
<p>The amount of time to wait before attempting to retry a failed request to a given topic partition. This avoids repeatedly sending requests in a tight loop under some failure scenarios.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.kinit.cmd"></a><a id="consumerconfigs_sasl.kerberos.kinit.cmd" href="#consumerconfigs_sasl.kerberos.kinit.cmd">sasl.kerberos.kinit.cmd</a></h4>
<p>Kerberos kinit command path.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>/usr/bin/kinit</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.min.time.before.relogin"></a><a id="consumerconfigs_sasl.kerberos.min.time.before.relogin" href="#consumerconfigs_sasl.kerberos.min.time.before.relogin">sasl.kerberos.min.time.before.relogin</a></h4>
<p>Login thread sleep time between refresh attempts.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>60000</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.ticket.renew.jitter"></a><a id="consumerconfigs_sasl.kerberos.ticket.renew.jitter" href="#consumerconfigs_sasl.kerberos.ticket.renew.jitter">sasl.kerberos.ticket.renew.jitter</a></h4>
<p>Percentage of random jitter added to the renewal time.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.05</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.ticket.renew.window.factor"></a><a id="consumerconfigs_sasl.kerberos.ticket.renew.window.factor" href="#consumerconfigs_sasl.kerberos.ticket.renew.window.factor">sasl.kerberos.ticket.renew.window.factor</a></h4>
<p>Login thread will sleep until the specified window factor of time from last refresh to ticket's expiry has been reached, at which time it will try to renew the ticket.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.8</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.connect.timeout.ms"></a><a id="consumerconfigs_sasl.login.connect.timeout.ms" href="#consumerconfigs_sasl.login.connect.timeout.ms">sasl.login.connect.timeout.ms</a></h4>
<p>The (optional) value in milliseconds for the external authentication provider connection timeout. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.read.timeout.ms"></a><a id="consumerconfigs_sasl.login.read.timeout.ms" href="#consumerconfigs_sasl.login.read.timeout.ms">sasl.login.read.timeout.ms</a></h4>
<p>The (optional) value in milliseconds for the external authentication provider read timeout. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.buffer.seconds"></a><a id="consumerconfigs_sasl.login.refresh.buffer.seconds" href="#consumerconfigs_sasl.login.refresh.buffer.seconds">sasl.login.refresh.buffer.seconds</a></h4>
<p>The amount of buffer time before credential expiration to maintain when refreshing a credential, in seconds. If a refresh would otherwise occur closer to expiration than the number of buffer seconds then the refresh will be moved up to maintain as much of the buffer time as possible. Legal values are between 0 and 3600 (1 hour); a default value of  300 (5 minutes) is used if no value is specified. This value and sasl.login.refresh.min.period.seconds are both ignored if their sum exceeds the remaining lifetime of a credential. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>short</td></tr>
<tr><th>Default:</th><td>300</td></tr>
<tr><th>Valid Values:</th><td>[0,...,3600]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.min.period.seconds"></a><a id="consumerconfigs_sasl.login.refresh.min.period.seconds" href="#consumerconfigs_sasl.login.refresh.min.period.seconds">sasl.login.refresh.min.period.seconds</a></h4>
<p>The desired minimum time for the login refresh thread to wait before refreshing a credential, in seconds. Legal values are between 0 and 900 (15 minutes); a default value of 60 (1 minute) is used if no value is specified.  This value and  sasl.login.refresh.buffer.seconds are both ignored if their sum exceeds the remaining lifetime of a credential. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>short</td></tr>
<tr><th>Default:</th><td>60</td></tr>
<tr><th>Valid Values:</th><td>[0,...,900]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.window.factor"></a><a id="consumerconfigs_sasl.login.refresh.window.factor" href="#consumerconfigs_sasl.login.refresh.window.factor">sasl.login.refresh.window.factor</a></h4>
<p>Login refresh thread will sleep until the specified window factor relative to the credential's lifetime has been reached, at which time it will try to refresh the credential. Legal values are between 0.5 (50%) and 1.0 (100%) inclusive; a default value of 0.8 (80%) is used if no value is specified. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.8</td></tr>
<tr><th>Valid Values:</th><td>[0.5,...,1.0]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.window.jitter"></a><a id="consumerconfigs_sasl.login.refresh.window.jitter" href="#consumerconfigs_sasl.login.refresh.window.jitter">sasl.login.refresh.window.jitter</a></h4>
<p>The maximum amount of random jitter relative to the credential's lifetime that is added to the login refresh thread's sleep time. Legal values are between 0 and 0.25 (25%) inclusive; a default value of 0.05 (5%) is used if no value is specified. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.05</td></tr>
<tr><th>Valid Values:</th><td>[0.0,...,0.25]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.retry.backoff.max.ms"></a><a id="consumerconfigs_sasl.login.retry.backoff.max.ms" href="#consumerconfigs_sasl.login.retry.backoff.max.ms">sasl.login.retry.backoff.max.ms</a></h4>
<p>The (optional) value in milliseconds for the maximum wait between login attempts to the external authentication provider. Login uses an exponential backoff algorithm with an initial wait based on the sasl.login.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.login.retry.backoff.max.ms setting. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.retry.backoff.ms"></a><a id="consumerconfigs_sasl.login.retry.backoff.ms" href="#consumerconfigs_sasl.login.retry.backoff.ms">sasl.login.retry.backoff.ms</a></h4>
<p>The (optional) value in milliseconds for the initial wait between login attempts to the external authentication provider. Login uses an exponential backoff algorithm with an initial wait based on the sasl.login.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.login.retry.backoff.max.ms setting. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.clock.skew.seconds"></a><a id="consumerconfigs_sasl.oauthbearer.clock.skew.seconds" href="#consumerconfigs_sasl.oauthbearer.clock.skew.seconds">sasl.oauthbearer.clock.skew.seconds</a></h4>
<p>The (optional) value in seconds to allow for differences between the time of the OAuth/OIDC identity provider and the broker.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>30</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.expected.audience"></a><a id="consumerconfigs_sasl.oauthbearer.expected.audience" href="#consumerconfigs_sasl.oauthbearer.expected.audience">sasl.oauthbearer.expected.audience</a></h4>
<p>The (optional) comma-delimited setting for the broker to use to verify that the JWT was issued for one of the expected audiences. The JWT will be inspected for the standard OAuth "aud" claim and if this value is set, the broker will match the value from JWT's "aud" claim  to see if there is an exact match. If there is no match, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.expected.issuer"></a><a id="consumerconfigs_sasl.oauthbearer.expected.issuer" href="#consumerconfigs_sasl.oauthbearer.expected.issuer">sasl.oauthbearer.expected.issuer</a></h4>
<p>The (optional) setting for the broker to use to verify that the JWT was created by the expected issuer. The JWT will be inspected for the standard OAuth "iss" claim and if this value is set, the broker will match it exactly against what is in the JWT's "iss" claim. If there is no match, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.refresh.ms"></a><a id="consumerconfigs_sasl.oauthbearer.jwks.endpoint.refresh.ms" href="#consumerconfigs_sasl.oauthbearer.jwks.endpoint.refresh.ms">sasl.oauthbearer.jwks.endpoint.refresh.ms</a></h4>
<p>The (optional) value in milliseconds for the broker to wait between refreshing its JWKS (JSON Web Key Set) cache that contains the keys to verify the signature of the JWT.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>3600000 (1 hour)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms"></a><a id="consumerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms" href="#consumerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms">sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms</a></h4>
<p>The (optional) value in milliseconds for the maximum wait between attempts to retrieve the JWKS (JSON Web Key Set) from the external authentication provider. JWKS retrieval uses an exponential backoff algorithm with an initial wait based on the sasl.oauthbearer.jwks.endpoint.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms setting.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.retry.backoff.ms"></a><a id="consumerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.ms" href="#consumerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.ms">sasl.oauthbearer.jwks.endpoint.retry.backoff.ms</a></h4>
<p>The (optional) value in milliseconds for the initial wait between JWKS (JSON Web Key Set) retrieval attempts from the external authentication provider. JWKS retrieval uses an exponential backoff algorithm with an initial wait based on the sasl.oauthbearer.jwks.endpoint.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms setting.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.scope.claim.name"></a><a id="consumerconfigs_sasl.oauthbearer.scope.claim.name" href="#consumerconfigs_sasl.oauthbearer.scope.claim.name">sasl.oauthbearer.scope.claim.name</a></h4>
<p>The OAuth claim for the scope is often named "scope", but this (optional) setting can provide a different name to use for the scope included in the JWT payload's claims if the OAuth/OIDC provider uses a different name for that claim.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>scope</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.sub.claim.name"></a><a id="consumerconfigs_sasl.oauthbearer.sub.claim.name" href="#consumerconfigs_sasl.oauthbearer.sub.claim.name">sasl.oauthbearer.sub.claim.name</a></h4>
<p>The OAuth claim for the subject is often named "sub", but this (optional) setting can provide a different name to use for the subject included in the JWT payload's claims if the OAuth/OIDC provider uses a different name for that claim.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>sub</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="security.providers"></a><a id="consumerconfigs_security.providers" href="#consumerconfigs_security.providers">security.providers</a></h4>
<p>A list of configurable creator classes each returning a provider implementing security algorithms. These classes should implement the <code>org.apache.kafka.common.security.auth.SecurityProviderCreator</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.cipher.suites"></a><a id="consumerconfigs_ssl.cipher.suites" href="#consumerconfigs_ssl.cipher.suites">ssl.cipher.suites</a></h4>
<p>A list of cipher suites. This is a named combination of authentication, encryption, MAC and key exchange algorithm used to negotiate the security settings for a network connection using TLS or SSL network protocol. By default all the available cipher suites are supported.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.endpoint.identification.algorithm"></a><a id="consumerconfigs_ssl.endpoint.identification.algorithm" href="#consumerconfigs_ssl.endpoint.identification.algorithm">ssl.endpoint.identification.algorithm</a></h4>
<p>The endpoint identification algorithm to validate server hostname using server certificate. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>https</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.engine.factory.class"></a><a id="consumerconfigs_ssl.engine.factory.class" href="#consumerconfigs_ssl.engine.factory.class">ssl.engine.factory.class</a></h4>
<p>The class of type org.apache.kafka.common.security.auth.SslEngineFactory to provide SSLEngine objects. Default value is org.apache.kafka.common.security.ssl.DefaultSslEngineFactory</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keymanager.algorithm"></a><a id="consumerconfigs_ssl.keymanager.algorithm" href="#consumerconfigs_ssl.keymanager.algorithm">ssl.keymanager.algorithm</a></h4>
<p>The algorithm used by key manager factory for SSL connections. Default value is the key manager factory algorithm configured for the Java Virtual Machine.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>SunX509</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.secure.random.implementation"></a><a id="consumerconfigs_ssl.secure.random.implementation" href="#consumerconfigs_ssl.secure.random.implementation">ssl.secure.random.implementation</a></h4>
<p>The SecureRandom PRNG implementation to use for SSL cryptography operations. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.trustmanager.algorithm"></a><a id="consumerconfigs_ssl.trustmanager.algorithm" href="#consumerconfigs_ssl.trustmanager.algorithm">ssl.trustmanager.algorithm</a></h4>
<p>The algorithm used by trust manager factory for SSL connections. Default value is the trust manager factory algorithm configured for the Java Virtual Machine.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>PKIX</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
</ul>




##### 基础配置

| 消费者配置              | 是否必须 | 说明                                           | 示例                                                     |
| ----------------------- | -------- | ---------------------------------------------- | -------------------------------------------------------- |
| boostrap.servers        | 是       | 表示kafka的服务端                              | 127.0.0.1:9092                                           |
| key.deserializer        | 是       | 表示消息key部分的反序列话                      | org.apache.kafka.common.serialization.StringDeserializer |
| value.deserializer      | 是       | 表示消息value部分的反序列化                    | org.apache.kafka.common.serialization.StringDeserializer |
| group.id                | 否       | 表示消费者所在的group，可以不配置。            | log_group                                                |
| client.id               | 否       | 唯一标识客户端，用于追踪request的客户端        | 默认为空字符串，client_192.168.0.1                       |
| send.buffer.bytes       | 否       | tcp send buffer大小                            | 默认128 * 1024                                           |
| receive.buffer.bytes    | 否       | tcp receive buffer大小                         | 默认64*1024                                              |
| request.timeout.ms      | 否       | 请求的最长等待时间，如果超过，将会重新发送消息 | 默认30000                                                |
| default.api.timeout.ms  | 否       | 所有api的默认超时时间                          | 默认60000                                                |
| **interceptor.classes** | 否       | 拦截器类名，可以用来处理record                 | 默认为空                                                 |
|                         |          |                                                |                                                          |



##### assign相关配置

| 配置                          | 必须 | 说明                                                         | 示例                                           |
| ----------------------------- | ---- | ------------------------------------------------------------ | ---------------------------------------------- |
| session.timeout.ms            | 否   | 当使用group功能时，消费者会向broker发送心跳，如果session.timeout.ms没有接收到心跳，broker将会移除该消费者 | 默认值：45000                                  |
| heartbeat.interval.ms         | 否   | 当使用group功能时，heartbeat.interval.ms为发送心跳的间隔     | 默认值：3000                                   |
| partition.assignment.strategy | 否   | 表示partition分配的策略                                      | 默认为RangeAssignor, CooperativeStickyAssignor |
|                               |      |                                                              |                                                |
|                               |      |                                                              |                                                |
|                               |      |                                                              |                                                |
|                               |      |                                                              |                                                |
|                               |      |                                                              |                                                |
|                               |      |                                                              |                                                |



##### poll相关配置

| 配置                      | 必须 | 说明                                                         | 示例                 |
| ------------------------- | ---- | ------------------------------------------------------------ | -------------------- |
| max.partition.fetch.bytes | 否   | 拉取消息时每个分区的最大数据量。如果分区的第一个消息超过该大小，为了保证消息可以正常消费，依然可以发送。 | 默认为`1024*1024`    |
| fetch.min.bytes           | 否   | 拉取数据的最小值，如果数据量不满足，将会等待                 | 默认为`1`            |
| fetch.max.bytes           | 否   | 拉取数据的最大值，如果第一个分区第一个消息大于该值，为了保证消息可以正常消费，依然可以发送 | 默认为`50*1024*1024` |
| fetch.max.wait.ms         | 否   | 当数据量不满足fetch.min.bytes时，broker最长等待时间          | 默认`500`            |
| auto.offset.reset         | 否   | 当之前不存在offset记录时，如何设置初始offset。latest：等待最新的消息；earliest：从最早的消息开始消费；none：如果不存在offset，抛出异常 | 默认latest           |
| max.poll.records          | 否   | poll函数每次返回的最大消息数，不会影响底层fetch操作，如果fetch更多消息，将会缓存 | 默认500              |
|                           |      |                                                              |                      |
|                           |      |                                                              |                      |
|                           |      |                                                              |                      |



##### 消息提交

| 配置                    | 必须 | 说明                                               | 示例       |
| ----------------------- | ---- | -------------------------------------------------- | ---------- |
| enable.auto.commit      | 否   | 是否自动提交offset                                 | 默认为true |
| auto.commit.interval.ms | 否   | 如果enable.auto.commit为true时，指定自动提交的周期 | 默认5000   |
|                         |      |                                                    |            |
|                         |      |                                                    |            |



#### 分区分配策略

分区分配策略是指coordinator如何将partition分配给消费者

| 策略                                                        | 说明                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| org.apache.kafka.clients.consumer.RangeAssignor             | 每个topic单独分配，平均分配parition                          |
| org.apache.kafka.clients.consumer.RoundRobinAssignor        | 把partition和consumer分别排序，然后轮流分配                  |
| org.apache.kafka.clients.consumer.StickyAssignor            | 在分配均衡的情况下，尽量让consumer分配的partition变动减少    |
| org.apache.kafka.clients.consumer.CooperativeStickyAssignor | 在StickyAssignor前提下，将分配调整为多次，实现在重分配之前不停止所有消费 |



### 生产者

#### 生产者配置

```
org.apache.kafka.clients.producer.ProducerConfig
```



| 参数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| key.serializer      | 消息的key序列化                                              |
| value.serializer    | 消息的value序列化                                            |
| bootstrap.servers   | kafka服务端                                                  |
| retries             | 重试次数，默认最大整数                                       |
| delivery.timeout.ms | 消息发送的最长时间，默认120000。包括重试的时间               |
| partitioner.class   | 为消息选择分区的类。默认：org.apache.kafka.clients.producer.internals.DefaultPartitioner |
| acks                | 消息需要的ack。默认all,-1表示需要所有的isr持久化消息，0表示只需要leader接收到消息，不保存leader持久化，1表示只需要leader持久化 |
| enable.idempotence  | 是否幂等，默认false。开启可以保证消息只提交一次              |
| transactional.id    | 事务id，如果需要使用事务特性，需要配置                       |



<ul class="config-list">
<li>
<h4><a id="key.serializer"></a><a id="producerconfigs_key.serializer" href="#producerconfigs_key.serializer">key.serializer</a></h4>
<p>Serializer class for key that implements the <code>org.apache.kafka.common.serialization.Serializer</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td></td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="value.serializer"></a><a id="producerconfigs_value.serializer" href="#producerconfigs_value.serializer">value.serializer</a></h4>
<p>Serializer class for value that implements the <code>org.apache.kafka.common.serialization.Serializer</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td></td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="bootstrap.servers"></a><a id="producerconfigs_bootstrap.servers" href="#producerconfigs_bootstrap.servers">bootstrap.servers</a></h4>
<p>A list of host/port pairs to use for establishing the initial connection to the Kafka cluster. The client will make use of all servers irrespective of which servers are specified here for bootstrapping&mdash;this list only impacts the initial hosts used to discover the full set of servers. This list should be in the form <code>host1:port1,host2:port2,...</code>. Since these servers are just used for the initial connection to discover the full cluster membership (which may change dynamically), this list need not contain the full set of servers (you may want more than one, though, in case a server is down).</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="buffer.memory"></a><a id="producerconfigs_buffer.memory" href="#producerconfigs_buffer.memory">buffer.memory</a></h4>
<p>The total bytes of memory the producer can use to buffer records waiting to be sent to the server. If records are sent faster than they can be delivered to the server the producer will block for <code>max.block.ms</code> after which it will throw an exception.<p>This setting should correspond roughly to the total memory the producer will use, but is not a hard bound since not all memory the producer uses is used for buffering. Some additional memory will be used for compression (if compression is enabled) as well as for maintaining in-flight requests.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>33554432</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="compression.type"></a><a id="producerconfigs_compression.type" href="#producerconfigs_compression.type">compression.type</a></h4>
<p>The compression type for all data generated by the producer. The default is none (i.e. no compression). Valid  values are <code>none</code>, <code>gzip</code>, <code>snappy</code>, <code>lz4</code>, or <code>zstd</code>. Compression is of full batches of data, so the efficacy of batching will also impact the compression ratio (more batching means better compression).</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>none</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="retries"></a><a id="producerconfigs_retries" href="#producerconfigs_retries">retries</a></h4>
<p>Setting a value greater than zero will cause the client to resend any record whose send fails with a potentially transient error. Note that this retry is no different than if the client resent the record upon receiving the error. Allowing retries without setting <code>max.in.flight.requests.per.connection</code> to 1 will potentially change the ordering of records because if two batches are sent to a single partition, and the first fails and is retried but the second succeeds, then the records in the second batch may appear first. Note additionally that produce requests will be failed before the number of retries has been exhausted if the timeout configured by <code>delivery.timeout.ms</code> expires first before successful acknowledgement. Users should generally prefer to leave this config unset and instead use <code>delivery.timeout.ms</code> to control retry behavior.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>2147483647</td></tr>
<tr><th>Valid Values:</th><td>[0,...,2147483647]</td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.key.password"></a><a id="producerconfigs_ssl.key.password" href="#producerconfigs_ssl.key.password">ssl.key.password</a></h4>
<p>The password of the private key in the key store file orthe PEM key specified in `ssl.keystore.key'. This is required for clients only if two-way authentication is configured.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.certificate.chain"></a><a id="producerconfigs_ssl.keystore.certificate.chain" href="#producerconfigs_ssl.keystore.certificate.chain">ssl.keystore.certificate.chain</a></h4>
<p>Certificate chain in the format specified by 'ssl.keystore.type'. Default SSL engine factory supports only PEM format with a list of X.509 certificates</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.key"></a><a id="producerconfigs_ssl.keystore.key" href="#producerconfigs_ssl.keystore.key">ssl.keystore.key</a></h4>
<p>Private key in the format specified by 'ssl.keystore.type'. Default SSL engine factory supports only PEM format with PKCS#8 keys. If the key is encrypted, key password must be specified using 'ssl.key.password'</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.location"></a><a id="producerconfigs_ssl.keystore.location" href="#producerconfigs_ssl.keystore.location">ssl.keystore.location</a></h4>
<p>The location of the key store file. This is optional for client and can be used for two-way authentication for client.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.password"></a><a id="producerconfigs_ssl.keystore.password" href="#producerconfigs_ssl.keystore.password">ssl.keystore.password</a></h4>
<p>The store password for the key store file. This is optional for client and only needed if 'ssl.keystore.location' is configured.  Key store password is not supported for PEM format.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.certificates"></a><a id="producerconfigs_ssl.truststore.certificates" href="#producerconfigs_ssl.truststore.certificates">ssl.truststore.certificates</a></h4>
<p>Trusted certificates in the format specified by 'ssl.truststore.type'. Default SSL engine factory supports only PEM format with X.509 certificates.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.location"></a><a id="producerconfigs_ssl.truststore.location" href="#producerconfigs_ssl.truststore.location">ssl.truststore.location</a></h4>
<p>The location of the trust store file. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.password"></a><a id="producerconfigs_ssl.truststore.password" href="#producerconfigs_ssl.truststore.password">ssl.truststore.password</a></h4>
<p>The password for the trust store file. If a password is not set, trust store file configured will still be used, but integrity checking is disabled. Trust store password is not supported for PEM format.</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>high</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="batch.size"></a><a id="producerconfigs_batch.size" href="#producerconfigs_batch.size">batch.size</a></h4>
<p>The producer will attempt to batch records together into fewer requests whenever multiple records are being sent to the same partition. This helps performance on both the client and the server. This configuration controls the default batch size in bytes. <p>No attempt will be made to batch records larger than this size. <p>Requests sent to brokers will contain multiple batches, one for each partition with data available to be sent. <p>A small batch size will make batching less common and may reduce throughput (a batch size of zero will disable batching entirely). A very large batch size may use memory a bit more wastefully as we will always allocate a buffer of the specified batch size in anticipation of additional records.<p>Note: This setting gives the upper bound of the batch size to be sent. If we have fewer than this many bytes accumulated for this partition, we will 'linger' for the <code>linger.ms</code> time waiting for more records to show up. This <code>linger.ms</code> setting defaults to 0, which means we'll immediately send out a record even the accumulated batch size is under this <code>batch.size</code> setting.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>16384</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="client.dns.lookup"></a><a id="producerconfigs_client.dns.lookup" href="#producerconfigs_client.dns.lookup">client.dns.lookup</a></h4>
<p>Controls how the client uses DNS lookups. If set to <code>use_all_dns_ips</code>, connect to each returned IP address in sequence until a successful connection is established. After a disconnection, the next IP is used. Once all IPs have been used once, the client resolves the IP(s) from the hostname again (both the JVM and the OS cache DNS name lookups, however). If set to <code>resolve_canonical_bootstrap_servers_only</code>, resolve each bootstrap address into a list of canonical names. After the bootstrap phase, this behaves the same as <code>use_all_dns_ips</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>use_all_dns_ips</td></tr>
<tr><th>Valid Values:</th><td>[use_all_dns_ips, resolve_canonical_bootstrap_servers_only]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="client.id"></a><a id="producerconfigs_client.id" href="#producerconfigs_client.id">client.id</a></h4>
<p>An id string to pass to the server when making requests. The purpose of this is to be able to track the source of requests beyond just ip/port by allowing a logical application name to be included in server-side request logging.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="connections.max.idle.ms"></a><a id="producerconfigs_connections.max.idle.ms" href="#producerconfigs_connections.max.idle.ms">connections.max.idle.ms</a></h4>
<p>Close idle connections after the number of milliseconds specified by this config.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>540000 (9 minutes)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="delivery.timeout.ms"></a><a id="producerconfigs_delivery.timeout.ms" href="#producerconfigs_delivery.timeout.ms">delivery.timeout.ms</a></h4>
<p>An upper bound on the time to report success or failure after a call to <code>send()</code> returns. This limits the total time that a record will be delayed prior to sending, the time to await acknowledgement from the broker (if expected), and the time allowed for retriable send failures. The producer may report failure to send a record earlier than this config if either an unrecoverable error is encountered, the retries have been exhausted, or the record is added to a batch which reached an earlier delivery expiration deadline. The value of this config should be greater than or equal to the sum of <code>request.timeout.ms</code> and <code>linger.ms</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>120000 (2 minutes)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="linger.ms"></a><a id="producerconfigs_linger.ms" href="#producerconfigs_linger.ms">linger.ms</a></h4>
<p>The producer groups together any records that arrive in between request transmissions into a single batched request. Normally this occurs only under load when records arrive faster than they can be sent out. However in some circumstances the client may want to reduce the number of requests even under moderate load. This setting accomplishes this by adding a small amount of artificial delay&mdash;that is, rather than immediately sending out a record, the producer will wait for up to the given delay to allow other records to be sent so that the sends can be batched together. This can be thought of as analogous to Nagle's algorithm in TCP. This setting gives the upper bound on the delay for batching: once we get <code>batch.size</code> worth of records for a partition it will be sent immediately regardless of this setting, however if we have fewer than this many bytes accumulated for this partition we will 'linger' for the specified time waiting for more records to show up. This setting defaults to 0 (i.e. no delay). Setting <code>linger.ms=5</code>, for example, would have the effect of reducing the number of requests sent but would add up to 5ms of latency to records sent in the absence of load.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>0</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.block.ms"></a><a id="producerconfigs_max.block.ms" href="#producerconfigs_max.block.ms">max.block.ms</a></h4>
<p>The configuration controls how long the <code>KafkaProducer</code>'s <code>send()</code>, <code>partitionsFor()</code>, <code>initTransactions()</code>, <code>sendOffsetsToTransaction()</code>, <code>commitTransaction()</code> and <code>abortTransaction()</code> methods will block. For <code>send()</code> this timeout bounds the total time waiting for both metadata fetch and buffer allocation (blocking in the user-supplied serializers or partitioner is not counted against this timeout). For <code>partitionsFor()</code> this timeout bounds the time spent waiting for metadata if it is unavailable. The transaction-related methods always block, but may timeout if the transaction coordinator could not be discovered or did not respond within the timeout.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>60000 (1 minute)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.request.size"></a><a id="producerconfigs_max.request.size" href="#producerconfigs_max.request.size">max.request.size</a></h4>
<p>The maximum size of a request in bytes. This setting will limit the number of record batches the producer will send in a single request to avoid sending huge requests. This is also effectively a cap on the maximum uncompressed record batch size. Note that the server has its own cap on the record batch size (after compression if compression is enabled) which may be different from this.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>1048576</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="partitioner.class"></a><a id="producerconfigs_partitioner.class" href="#producerconfigs_partitioner.class">partitioner.class</a></h4>
<p>A class to use to determine which partition to be send to when produce the records. Available options are:<ul><li><code>org.apache.kafka.clients.producer.internals.DefaultPartitioner</code>: The default partitioner. This strategy will try sticking to a partition until the batch is full, or <code>linger.ms</code> is up. It works with the strategy:<ul><li>If no partition is specified but a key is present, choose a partition based on a hash of the key</li><li>If no partition or key is present, choose the sticky partition that changes when the batch is full, or <code>linger.ms</code> is up.</li></ul></li><li><code>org.apache.kafka.clients.producer.RoundRobinPartitioner</code>: This partitioning strategy is that each record in a series of consecutive records will be sent to a different partition(no matter if the 'key' is provided or not), until we run out of partitions and start over again. Note: There's a known issue that will cause uneven distribution when new batch is created. Please check KAFKA-9965 for more detail.</li><li><code>org.apache.kafka.clients.producer.UniformStickyPartitioner</code>: This partitioning strategy will try sticking to a partition(no matter if the 'key' is provided or not) until the batch is full, or <code>linger.ms</code> is up.</li></ul><p>Implementing the <code>org.apache.kafka.clients.producer.Partitioner</code> interface allows you to plug in a custom partitioner.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>org.apache.kafka.clients.producer.internals.DefaultPartitioner</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="receive.buffer.bytes"></a><a id="producerconfigs_receive.buffer.bytes" href="#producerconfigs_receive.buffer.bytes">receive.buffer.bytes</a></h4>
<p>The size of the TCP receive buffer (SO_RCVBUF) to use when reading data. If the value is -1, the OS default will be used.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>32768 (32 kibibytes)</td></tr>
<tr><th>Valid Values:</th><td>[-1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="request.timeout.ms"></a><a id="producerconfigs_request.timeout.ms" href="#producerconfigs_request.timeout.ms">request.timeout.ms</a></h4>
<p>The configuration controls the maximum amount of time the client will wait for the response of a request. If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted. This should be larger than <code>replica.lag.time.max.ms</code> (a broker configuration) to reduce the possibility of message duplication due to unnecessary producer retries.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.client.callback.handler.class"></a><a id="producerconfigs_sasl.client.callback.handler.class" href="#producerconfigs_sasl.client.callback.handler.class">sasl.client.callback.handler.class</a></h4>
<p>The fully qualified name of a SASL client callback handler class that implements the AuthenticateCallbackHandler interface.</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.jaas.config"></a><a id="producerconfigs_sasl.jaas.config" href="#producerconfigs_sasl.jaas.config">sasl.jaas.config</a></h4>
<p>JAAS login context parameters for SASL connections in the format used by JAAS configuration files. JAAS configuration file format is described <a href="http://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html">here</a>. The format for the value is: <code>loginModuleClass controlFlag (optionName=optionValue)*;</code>. For brokers, the config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=com.example.ScramLoginModule required;</p>
<table><tbody>
<tr><th>Type:</th><td>password</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.service.name"></a><a id="producerconfigs_sasl.kerberos.service.name" href="#producerconfigs_sasl.kerberos.service.name">sasl.kerberos.service.name</a></h4>
<p>The Kerberos principal name that Kafka runs as. This can be defined either in Kafka's JAAS config or in Kafka's config.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.callback.handler.class"></a><a id="producerconfigs_sasl.login.callback.handler.class" href="#producerconfigs_sasl.login.callback.handler.class">sasl.login.callback.handler.class</a></h4>
<p>The fully qualified name of a SASL login callback handler class that implements the AuthenticateCallbackHandler interface. For brokers, login callback handler config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.login.callback.handler.class=com.example.CustomScramLoginCallbackHandler</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.class"></a><a id="producerconfigs_sasl.login.class" href="#producerconfigs_sasl.login.class">sasl.login.class</a></h4>
<p>The fully qualified name of a class that implements the Login interface. For brokers, login config must be prefixed with listener prefix and SASL mechanism name in lower-case. For example, listener.name.sasl_ssl.scram-sha-256.sasl.login.class=com.example.CustomScramLogin</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.mechanism"></a><a id="producerconfigs_sasl.mechanism" href="#producerconfigs_sasl.mechanism">sasl.mechanism</a></h4>
<p>SASL mechanism used for client connections. This may be any mechanism for which a security provider is available. GSSAPI is the default mechanism.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>GSSAPI</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.url"></a><a id="producerconfigs_sasl.oauthbearer.jwks.endpoint.url" href="#producerconfigs_sasl.oauthbearer.jwks.endpoint.url">sasl.oauthbearer.jwks.endpoint.url</a></h4>
<p>The OAuth/OIDC provider URL from which the provider's <a href="https://datatracker.ietf.org/doc/html/rfc7517#section-5">JWKS (JSON Web Key Set)</a> can be retrieved. The URL can be HTTP(S)-based or file-based. If the URL is HTTP(S)-based, the JWKS data will be retrieved from the OAuth/OIDC provider via the configured URL on broker startup. All then-current keys will be cached on the broker for incoming requests. If an authentication request is received for a JWT that includes a "kid" header claim value that isn't yet in the cache, the JWKS endpoint will be queried again on demand. However, the broker polls the URL every sasl.oauthbearer.jwks.endpoint.refresh.ms milliseconds to refresh the cache with any forthcoming keys before any JWT requests that include them are received. If the URL is file-based, the broker will load the JWKS file from a configured location on startup. In the event that the JWT includes a "kid" header value that isn't in the JWKS file, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.token.endpoint.url"></a><a id="producerconfigs_sasl.oauthbearer.token.endpoint.url" href="#producerconfigs_sasl.oauthbearer.token.endpoint.url">sasl.oauthbearer.token.endpoint.url</a></h4>
<p>The URL for the OAuth/OIDC identity provider. If the URL is HTTP(S)-based, it is the issuer's token endpoint URL to which requests will be made to login based on the configuration in sasl.jaas.config. If the URL is file-based, it specifies a file containing an access token (in JWT serialized form) issued by the OAuth/OIDC identity provider to use for authorization.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="security.protocol"></a><a id="producerconfigs_security.protocol" href="#producerconfigs_security.protocol">security.protocol</a></h4>
<p>Protocol used to communicate with brokers. Valid values are: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>PLAINTEXT</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="send.buffer.bytes"></a><a id="producerconfigs_send.buffer.bytes" href="#producerconfigs_send.buffer.bytes">send.buffer.bytes</a></h4>
<p>The size of the TCP send buffer (SO_SNDBUF) to use when sending data. If the value is -1, the OS default will be used.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>131072 (128 kibibytes)</td></tr>
<tr><th>Valid Values:</th><td>[-1,...]</td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="socket.connection.setup.timeout.max.ms"></a><a id="producerconfigs_socket.connection.setup.timeout.max.ms" href="#producerconfigs_socket.connection.setup.timeout.max.ms">socket.connection.setup.timeout.max.ms</a></h4>
<p>The maximum amount of time the client will wait for the socket connection to be established. The connection setup timeout will increase exponentially for each consecutive connection failure up to this maximum. To avoid connection storms, a randomization factor of 0.2 will be applied to the timeout resulting in a random range between 20% below and 20% above the computed value.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="socket.connection.setup.timeout.ms"></a><a id="producerconfigs_socket.connection.setup.timeout.ms" href="#producerconfigs_socket.connection.setup.timeout.ms">socket.connection.setup.timeout.ms</a></h4>
<p>The amount of time the client will wait for the socket connection to be established. If the connection is not built before the timeout elapses, clients will close the socket channel.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.enabled.protocols"></a><a id="producerconfigs_ssl.enabled.protocols" href="#producerconfigs_ssl.enabled.protocols">ssl.enabled.protocols</a></h4>
<p>The list of protocols enabled for SSL connections. The default is 'TLSv1.2,TLSv1.3' when running with Java 11 or newer, 'TLSv1.2' otherwise. With the default value for Java 11, clients and servers will prefer TLSv1.3 if both support it and fallback to TLSv1.2 otherwise (assuming both support at least TLSv1.2). This default should be fine for most cases. Also see the config documentation for `ssl.protocol`.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>TLSv1.2,TLSv1.3</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keystore.type"></a><a id="producerconfigs_ssl.keystore.type" href="#producerconfigs_ssl.keystore.type">ssl.keystore.type</a></h4>
<p>The file format of the key store file. This is optional for client.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>JKS</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.protocol"></a><a id="producerconfigs_ssl.protocol" href="#producerconfigs_ssl.protocol">ssl.protocol</a></h4>
<p>The SSL protocol used to generate the SSLContext. The default is 'TLSv1.3' when running with Java 11 or newer, 'TLSv1.2' otherwise. This value should be fine for most use cases. Allowed values in recent JVMs are 'TLSv1.2' and 'TLSv1.3'. 'TLS', 'TLSv1.1', 'SSL', 'SSLv2' and 'SSLv3' may be supported in older JVMs, but their usage is discouraged due to known security vulnerabilities. With the default value for this config and 'ssl.enabled.protocols', clients will downgrade to 'TLSv1.2' if the server does not support 'TLSv1.3'. If this config is set to 'TLSv1.2', clients will not use 'TLSv1.3' even if it is one of the values in ssl.enabled.protocols and the server only supports 'TLSv1.3'.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>TLSv1.3</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.provider"></a><a id="producerconfigs_ssl.provider" href="#producerconfigs_ssl.provider">ssl.provider</a></h4>
<p>The name of the security provider used for SSL connections. Default value is the default security provider of the JVM.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.truststore.type"></a><a id="producerconfigs_ssl.truststore.type" href="#producerconfigs_ssl.truststore.type">ssl.truststore.type</a></h4>
<p>The file format of the trust store file.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>JKS</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>medium</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="acks"></a><a id="producerconfigs_acks" href="#producerconfigs_acks">acks</a></h4>
<p>The number of acknowledgments the producer requires the leader to have received before considering a request complete. This controls the  durability of records that are sent. The following settings are allowed:  <ul> <li><code>acks=0</code> If set to zero then the producer will not wait for any acknowledgment from the server at all. The record will be immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the <code>retries</code> configuration will not take effect (as the client won't generally know of any failures). The offset given back for each record will always be set to <code>-1</code>. <li><code>acks=1</code> This will mean the leader will write the record to its local log but will respond without awaiting full acknowledgement from all followers. In this case should the leader fail immediately after acknowledging the record but before the followers have replicated it then the record will be lost. <li><code>acks=all</code> This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee. This is equivalent to the acks=-1 setting.</ul></p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>all</td></tr>
<tr><th>Valid Values:</th><td>[all, -1, 0, 1]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="enable.idempotence"></a><a id="producerconfigs_enable.idempotence" href="#producerconfigs_enable.idempotence">enable.idempotence</a></h4>
<p>When set to 'true', the producer will ensure that exactly one copy of each message is written in the stream. If 'false', producer retries due to broker failures, etc., may write duplicates of the retried message in the stream. Note that enabling idempotence requires <code>max.in.flight.requests.per.connection</code> to be less than or equal to 5 (with message ordering preserved for any allowable value), <code>retries</code> to be greater than 0, and <code>acks</code> must be 'all'. If these values are not explicitly set by the user, suitable values will be chosen. If incompatible values are set, a <code>ConfigException</code> will be thrown.</p>
<table><tbody>
<tr><th>Type:</th><td>boolean</td></tr>
<tr><th>Default:</th><td>true</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="interceptor.classes"></a><a id="producerconfigs_interceptor.classes" href="#producerconfigs_interceptor.classes">interceptor.classes</a></h4>
<p>A list of classes to use as interceptors. Implementing the <code>org.apache.kafka.clients.producer.ProducerInterceptor</code> interface allows you to intercept (and possibly mutate) the records received by the producer before they are published to the Kafka cluster. By default, there are no interceptors.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="max.in.flight.requests.per.connection"></a><a id="producerconfigs_max.in.flight.requests.per.connection" href="#producerconfigs_max.in.flight.requests.per.connection">max.in.flight.requests.per.connection</a></h4>
<p>The maximum number of unacknowledged requests the client will send on a single connection before blocking. Note that if this config is set to be greater than 1 and <code>enable.idempotence</code> is set to false, there is a risk of message re-ordering after a failed send due to retries (i.e., if retries are enabled).</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>5</td></tr>
<tr><th>Valid Values:</th><td>[1,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metadata.max.age.ms"></a><a id="producerconfigs_metadata.max.age.ms" href="#producerconfigs_metadata.max.age.ms">metadata.max.age.ms</a></h4>
<p>The period of time in milliseconds after which we force a refresh of metadata even if we haven't seen any partition leadership changes to proactively discover any new brokers or partitions.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>300000 (5 minutes)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metadata.max.idle.ms"></a><a id="producerconfigs_metadata.max.idle.ms" href="#producerconfigs_metadata.max.idle.ms">metadata.max.idle.ms</a></h4>
<p>Controls how long the producer will cache metadata for a topic that's idle. If the elapsed time since a topic was last produced to exceeds the metadata idle duration, then the topic's metadata is forgotten and the next access to it will force a metadata fetch request.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>300000 (5 minutes)</td></tr>
<tr><th>Valid Values:</th><td>[5000,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metric.reporters"></a><a id="producerconfigs_metric.reporters" href="#producerconfigs_metric.reporters">metric.reporters</a></h4>
<p>A list of classes to use as metrics reporters. Implementing the <code>org.apache.kafka.common.metrics.MetricsReporter</code> interface allows plugging in classes that will be notified of new metric creation. The JmxReporter is always included to register JMX statistics.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>""</td></tr>
<tr><th>Valid Values:</th><td>non-null string</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.num.samples"></a><a id="producerconfigs_metrics.num.samples" href="#producerconfigs_metrics.num.samples">metrics.num.samples</a></h4>
<p>The number of samples maintained to compute metrics.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>2</td></tr>
<tr><th>Valid Values:</th><td>[1,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.recording.level"></a><a id="producerconfigs_metrics.recording.level" href="#producerconfigs_metrics.recording.level">metrics.recording.level</a></h4>
<p>The highest recording level for metrics.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>INFO</td></tr>
<tr><th>Valid Values:</th><td>[INFO, DEBUG, TRACE]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="metrics.sample.window.ms"></a><a id="producerconfigs_metrics.sample.window.ms" href="#producerconfigs_metrics.sample.window.ms">metrics.sample.window.ms</a></h4>
<p>The window of time a metrics sample is computed over.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>30000 (30 seconds)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="reconnect.backoff.max.ms"></a><a id="producerconfigs_reconnect.backoff.max.ms" href="#producerconfigs_reconnect.backoff.max.ms">reconnect.backoff.max.ms</a></h4>
<p>The maximum amount of time in milliseconds to wait when reconnecting to a broker that has repeatedly failed to connect. If provided, the backoff per host will increase exponentially for each consecutive connection failure, up to this maximum. After calculating the backoff increase, 20% random jitter is added to avoid connection storms.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>1000 (1 second)</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="reconnect.backoff.ms"></a><a id="producerconfigs_reconnect.backoff.ms" href="#producerconfigs_reconnect.backoff.ms">reconnect.backoff.ms</a></h4>
<p>The base amount of time to wait before attempting to reconnect to a given host. This avoids repeatedly connecting to a host in a tight loop. This backoff applies to all connection attempts by the client to a broker.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>50</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="retry.backoff.ms"></a><a id="producerconfigs_retry.backoff.ms" href="#producerconfigs_retry.backoff.ms">retry.backoff.ms</a></h4>
<p>The amount of time to wait before attempting to retry a failed request to a given topic partition. This avoids repeatedly sending requests in a tight loop under some failure scenarios.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td>[0,...]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.kinit.cmd"></a><a id="producerconfigs_sasl.kerberos.kinit.cmd" href="#producerconfigs_sasl.kerberos.kinit.cmd">sasl.kerberos.kinit.cmd</a></h4>
<p>Kerberos kinit command path.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>/usr/bin/kinit</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.min.time.before.relogin"></a><a id="producerconfigs_sasl.kerberos.min.time.before.relogin" href="#producerconfigs_sasl.kerberos.min.time.before.relogin">sasl.kerberos.min.time.before.relogin</a></h4>
<p>Login thread sleep time between refresh attempts.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>60000</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.ticket.renew.jitter"></a><a id="producerconfigs_sasl.kerberos.ticket.renew.jitter" href="#producerconfigs_sasl.kerberos.ticket.renew.jitter">sasl.kerberos.ticket.renew.jitter</a></h4>
<p>Percentage of random jitter added to the renewal time.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.05</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.kerberos.ticket.renew.window.factor"></a><a id="producerconfigs_sasl.kerberos.ticket.renew.window.factor" href="#producerconfigs_sasl.kerberos.ticket.renew.window.factor">sasl.kerberos.ticket.renew.window.factor</a></h4>
<p>Login thread will sleep until the specified window factor of time from last refresh to ticket's expiry has been reached, at which time it will try to renew the ticket.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.8</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.connect.timeout.ms"></a><a id="producerconfigs_sasl.login.connect.timeout.ms" href="#producerconfigs_sasl.login.connect.timeout.ms">sasl.login.connect.timeout.ms</a></h4>
<p>The (optional) value in milliseconds for the external authentication provider connection timeout. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.read.timeout.ms"></a><a id="producerconfigs_sasl.login.read.timeout.ms" href="#producerconfigs_sasl.login.read.timeout.ms">sasl.login.read.timeout.ms</a></h4>
<p>The (optional) value in milliseconds for the external authentication provider read timeout. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.buffer.seconds"></a><a id="producerconfigs_sasl.login.refresh.buffer.seconds" href="#producerconfigs_sasl.login.refresh.buffer.seconds">sasl.login.refresh.buffer.seconds</a></h4>
<p>The amount of buffer time before credential expiration to maintain when refreshing a credential, in seconds. If a refresh would otherwise occur closer to expiration than the number of buffer seconds then the refresh will be moved up to maintain as much of the buffer time as possible. Legal values are between 0 and 3600 (1 hour); a default value of  300 (5 minutes) is used if no value is specified. This value and sasl.login.refresh.min.period.seconds are both ignored if their sum exceeds the remaining lifetime of a credential. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>short</td></tr>
<tr><th>Default:</th><td>300</td></tr>
<tr><th>Valid Values:</th><td>[0,...,3600]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.min.period.seconds"></a><a id="producerconfigs_sasl.login.refresh.min.period.seconds" href="#producerconfigs_sasl.login.refresh.min.period.seconds">sasl.login.refresh.min.period.seconds</a></h4>
<p>The desired minimum time for the login refresh thread to wait before refreshing a credential, in seconds. Legal values are between 0 and 900 (15 minutes); a default value of 60 (1 minute) is used if no value is specified.  This value and  sasl.login.refresh.buffer.seconds are both ignored if their sum exceeds the remaining lifetime of a credential. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>short</td></tr>
<tr><th>Default:</th><td>60</td></tr>
<tr><th>Valid Values:</th><td>[0,...,900]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.window.factor"></a><a id="producerconfigs_sasl.login.refresh.window.factor" href="#producerconfigs_sasl.login.refresh.window.factor">sasl.login.refresh.window.factor</a></h4>
<p>Login refresh thread will sleep until the specified window factor relative to the credential's lifetime has been reached, at which time it will try to refresh the credential. Legal values are between 0.5 (50%) and 1.0 (100%) inclusive; a default value of 0.8 (80%) is used if no value is specified. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.8</td></tr>
<tr><th>Valid Values:</th><td>[0.5,...,1.0]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.refresh.window.jitter"></a><a id="producerconfigs_sasl.login.refresh.window.jitter" href="#producerconfigs_sasl.login.refresh.window.jitter">sasl.login.refresh.window.jitter</a></h4>
<p>The maximum amount of random jitter relative to the credential's lifetime that is added to the login refresh thread's sleep time. Legal values are between 0 and 0.25 (25%) inclusive; a default value of 0.05 (5%) is used if no value is specified. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>double</td></tr>
<tr><th>Default:</th><td>0.05</td></tr>
<tr><th>Valid Values:</th><td>[0.0,...,0.25]</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.retry.backoff.max.ms"></a><a id="producerconfigs_sasl.login.retry.backoff.max.ms" href="#producerconfigs_sasl.login.retry.backoff.max.ms">sasl.login.retry.backoff.max.ms</a></h4>
<p>The (optional) value in milliseconds for the maximum wait between login attempts to the external authentication provider. Login uses an exponential backoff algorithm with an initial wait based on the sasl.login.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.login.retry.backoff.max.ms setting. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.login.retry.backoff.ms"></a><a id="producerconfigs_sasl.login.retry.backoff.ms" href="#producerconfigs_sasl.login.retry.backoff.ms">sasl.login.retry.backoff.ms</a></h4>
<p>The (optional) value in milliseconds for the initial wait between login attempts to the external authentication provider. Login uses an exponential backoff algorithm with an initial wait based on the sasl.login.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.login.retry.backoff.max.ms setting. Currently applies only to OAUTHBEARER.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.clock.skew.seconds"></a><a id="producerconfigs_sasl.oauthbearer.clock.skew.seconds" href="#producerconfigs_sasl.oauthbearer.clock.skew.seconds">sasl.oauthbearer.clock.skew.seconds</a></h4>
<p>The (optional) value in seconds to allow for differences between the time of the OAuth/OIDC identity provider and the broker.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>30</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.expected.audience"></a><a id="producerconfigs_sasl.oauthbearer.expected.audience" href="#producerconfigs_sasl.oauthbearer.expected.audience">sasl.oauthbearer.expected.audience</a></h4>
<p>The (optional) comma-delimited setting for the broker to use to verify that the JWT was issued for one of the expected audiences. The JWT will be inspected for the standard OAuth "aud" claim and if this value is set, the broker will match the value from JWT's "aud" claim  to see if there is an exact match. If there is no match, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.expected.issuer"></a><a id="producerconfigs_sasl.oauthbearer.expected.issuer" href="#producerconfigs_sasl.oauthbearer.expected.issuer">sasl.oauthbearer.expected.issuer</a></h4>
<p>The (optional) setting for the broker to use to verify that the JWT was created by the expected issuer. The JWT will be inspected for the standard OAuth "iss" claim and if this value is set, the broker will match it exactly against what is in the JWT's "iss" claim. If there is no match, the broker will reject the JWT and authentication will fail.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.refresh.ms"></a><a id="producerconfigs_sasl.oauthbearer.jwks.endpoint.refresh.ms" href="#producerconfigs_sasl.oauthbearer.jwks.endpoint.refresh.ms">sasl.oauthbearer.jwks.endpoint.refresh.ms</a></h4>
<p>The (optional) value in milliseconds for the broker to wait between refreshing its JWKS (JSON Web Key Set) cache that contains the keys to verify the signature of the JWT.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>3600000 (1 hour)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms"></a><a id="producerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms" href="#producerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms">sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms</a></h4>
<p>The (optional) value in milliseconds for the maximum wait between attempts to retrieve the JWKS (JSON Web Key Set) from the external authentication provider. JWKS retrieval uses an exponential backoff algorithm with an initial wait based on the sasl.oauthbearer.jwks.endpoint.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms setting.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>10000 (10 seconds)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.jwks.endpoint.retry.backoff.ms"></a><a id="producerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.ms" href="#producerconfigs_sasl.oauthbearer.jwks.endpoint.retry.backoff.ms">sasl.oauthbearer.jwks.endpoint.retry.backoff.ms</a></h4>
<p>The (optional) value in milliseconds for the initial wait between JWKS (JSON Web Key Set) retrieval attempts from the external authentication provider. JWKS retrieval uses an exponential backoff algorithm with an initial wait based on the sasl.oauthbearer.jwks.endpoint.retry.backoff.ms setting and will double in wait length between attempts up to a maximum wait length specified by the sasl.oauthbearer.jwks.endpoint.retry.backoff.max.ms setting.</p>
<table><tbody>
<tr><th>Type:</th><td>long</td></tr>
<tr><th>Default:</th><td>100</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.scope.claim.name"></a><a id="producerconfigs_sasl.oauthbearer.scope.claim.name" href="#producerconfigs_sasl.oauthbearer.scope.claim.name">sasl.oauthbearer.scope.claim.name</a></h4>
<p>The OAuth claim for the scope is often named "scope", but this (optional) setting can provide a different name to use for the scope included in the JWT payload's claims if the OAuth/OIDC provider uses a different name for that claim.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>scope</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="sasl.oauthbearer.sub.claim.name"></a><a id="producerconfigs_sasl.oauthbearer.sub.claim.name" href="#producerconfigs_sasl.oauthbearer.sub.claim.name">sasl.oauthbearer.sub.claim.name</a></h4>
<p>The OAuth claim for the subject is often named "sub", but this (optional) setting can provide a different name to use for the subject included in the JWT payload's claims if the OAuth/OIDC provider uses a different name for that claim.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>sub</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="security.providers"></a><a id="producerconfigs_security.providers" href="#producerconfigs_security.providers">security.providers</a></h4>
<p>A list of configurable creator classes each returning a provider implementing security algorithms. These classes should implement the <code>org.apache.kafka.common.security.auth.SecurityProviderCreator</code> interface.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.cipher.suites"></a><a id="producerconfigs_ssl.cipher.suites" href="#producerconfigs_ssl.cipher.suites">ssl.cipher.suites</a></h4>
<p>A list of cipher suites. This is a named combination of authentication, encryption, MAC and key exchange algorithm used to negotiate the security settings for a network connection using TLS or SSL network protocol. By default all the available cipher suites are supported.</p>
<table><tbody>
<tr><th>Type:</th><td>list</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.endpoint.identification.algorithm"></a><a id="producerconfigs_ssl.endpoint.identification.algorithm" href="#producerconfigs_ssl.endpoint.identification.algorithm">ssl.endpoint.identification.algorithm</a></h4>
<p>The endpoint identification algorithm to validate server hostname using server certificate. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>https</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.engine.factory.class"></a><a id="producerconfigs_ssl.engine.factory.class" href="#producerconfigs_ssl.engine.factory.class">ssl.engine.factory.class</a></h4>
<p>The class of type org.apache.kafka.common.security.auth.SslEngineFactory to provide SSLEngine objects. Default value is org.apache.kafka.common.security.ssl.DefaultSslEngineFactory</p>
<table><tbody>
<tr><th>Type:</th><td>class</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.keymanager.algorithm"></a><a id="producerconfigs_ssl.keymanager.algorithm" href="#producerconfigs_ssl.keymanager.algorithm">ssl.keymanager.algorithm</a></h4>
<p>The algorithm used by key manager factory for SSL connections. Default value is the key manager factory algorithm configured for the Java Virtual Machine.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>SunX509</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.secure.random.implementation"></a><a id="producerconfigs_ssl.secure.random.implementation" href="#producerconfigs_ssl.secure.random.implementation">ssl.secure.random.implementation</a></h4>
<p>The SecureRandom PRNG implementation to use for SSL cryptography operations. </p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="ssl.trustmanager.algorithm"></a><a id="producerconfigs_ssl.trustmanager.algorithm" href="#producerconfigs_ssl.trustmanager.algorithm">ssl.trustmanager.algorithm</a></h4>
<p>The algorithm used by trust manager factory for SSL connections. Default value is the trust manager factory algorithm configured for the Java Virtual Machine.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>PKIX</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="transaction.timeout.ms"></a><a id="producerconfigs_transaction.timeout.ms" href="#producerconfigs_transaction.timeout.ms">transaction.timeout.ms</a></h4>
<p>The maximum amount of time in ms that the transaction coordinator will wait for a transaction status update from the producer before proactively aborting the ongoing transaction.If this value is larger than the transaction.max.timeout.ms setting in the broker, the request will fail with a <code>InvalidTxnTimeoutException</code> error.</p>
<table><tbody>
<tr><th>Type:</th><td>int</td></tr>
<tr><th>Default:</th><td>60000 (1 minute)</td></tr>
<tr><th>Valid Values:</th><td></td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
<li>
<h4><a id="transactional.id"></a><a id="producerconfigs_transactional.id" href="#producerconfigs_transactional.id">transactional.id</a></h4>
<p>The TransactionalId to use for transactional delivery. This enables reliability semantics which span multiple producer sessions since it allows the client to guarantee that transactions using the same TransactionalId have been completed prior to starting any new transactions. If no TransactionalId is provided, then the producer is limited to idempotent delivery. If a TransactionalId is configured, <code>enable.idempotence</code> is implied. By default the TransactionId is not configured, which means transactions cannot be used. Note that, by default, transactions require a cluster of at least three brokers which is the recommended setting for production; for development you can change this, by adjusting broker setting <code>transaction.state.log.replication.factor</code>.</p>
<table><tbody>
<tr><th>Type:</th><td>string</td></tr>
<tr><th>Default:</th><td>null</td></tr>
<tr><th>Valid Values:</th><td>non-empty string</td></tr>
<tr><th>Importance:</th><td>low</td></tr>
</tbody></table>
</li>
</ul>






## kafka特性

 



### 事务

```java
package com.hewutao.kafka.transactional;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.junit.Before;
import org.junit.Test;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.TimeUnit;

@Slf4j
public class TransactionTest {
    private KafkaProducer<String, String> producer;
    private KafkaConsumer<String, String> consumer;

    private final String topic01 = "topic-01";
    private final String topic02 = "topic-02";
    private final String groupId = "test_group";
    private final String transactionalId = "test_transactional_id";

    @Before
    public void before() {
        initProducer();
        initConsumer();
    }

    private void initProducer() {
        Properties config = new Properties();
        config.put("bootstrap.servers", "127.0.0.1:9092");
        config.put("key.serializer", StringSerializer.class.getName());
        config.put("value.serializer", StringSerializer.class.getName());
      	// 生产者需要配置enable.idempotence为true，和配置transactional.id
        config.put("enable.idempotence", "true");
        config.put("transactional.id", transactionalId);

        producer = new KafkaProducer<>(config);
    }

    private void initConsumer() {
        Properties config = new Properties();
        config.put("bootstrap.servers", "127.0.0.1:9092");
        config.put("key.deserializer", StringDeserializer.class.getName());
        config.put("value.deserializer", StringDeserializer.class.getName());
        // 消费者需要配置enable.auto.commit为false
        config.put("enable.auto.commit", "false");
        config.put("group.id", groupId);
        config.put("isolation.level", "read_committed");

        consumer = new KafkaConsumer<>(config);

    }

    @Test
    public void process() {
        consumer.subscribe(Collections.singleton(topic01));
        producer.initTransactions();
        while(true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(3));

            if (records.isEmpty()) {
                continue;
            }

            producer.beginTransaction();
            try {
                Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
                for (TopicPartition partition : records.partitions()) {

                    List<ConsumerRecord<String, String>> consumerRecordList = records.records(partition);
                    if (consumerRecordList.isEmpty()) {
                        continue;
                    }
                    for (ConsumerRecord<String, String> consumerRecord : consumerRecordList) {
                        log.info("receive record: {}", consumerRecord);

                        ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic02, consumerRecord.key() , consumerRecord.value());
                        producer.send(producerRecord);
                    }
                    offsets.put(partition, new OffsetAndMetadata(consumerRecordList.get(consumerRecordList.size() - 1).offset()));
                }
                // 有producer提交偏移量
                producer.sendOffsetsToTransaction(offsets, consumer.groupMetadata());
                producer.commitTransaction();
            } catch (Throwable e) {
                try {
                    producer.abortTransaction();
                } catch (Throwable abortException) {
                    log.error("abort transaction failed", abortException);
                }

                throw e;
            }
        }
    }

}

```





