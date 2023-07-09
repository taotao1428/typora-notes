# kafka知识汇总

kafka是一个高性能分布式消息队列产品

## 概念介绍

| 概念                   | 介绍                                                         |
| ---------------------- | ------------------------------------------------------------ |
| producer               | 负责发送消息给kafka                                          |
| consumer               | 负责从kakfa拉取消息消费                                      |
| consumer group         | 每个consumer都属于一个group，如果不指定，将会属于一个默认的group |
| **topic**（主题）      | 主题可以看做是消息的类型，每个消息都有主题。不同topic的消息保存在不同的partition。生产者和消费者通过主题指定要生产和消费的消息 |
| **partition**（分区）  | 分区是保存数据的物理概念，每个topic对应1个或多个partition。  |
| offset（偏移量）       | offset表示消费者在partition消费的位置                        |
| **Broker**             | 表示一个kafka节点                                            |
| Controller             | kafka集群中的一个控制器，管理整个kafka集群。同一时间只有一个Broker是Controller |
| GroupCoordinator       | group协调者，用于协调同一个group中的消费者。每个Broker都是GroupCoordinator |
| TransactionCoordinator | 事务协调者，用于协调消费者的事务。每个Broker都是TransactionCoordinator |

![image-20220417232942494](/开源框架/kafka/.assert/kafka汇总/image-20220417232942494.png)



### 主题和分区

每个主题topic可以有多个分区partition，生产者可以将消息发送到不同的分区，消费者可以独立消费不同分区的消息，因此增加分区可以提高生产和消费消息的并发能力。但是要注意，不同分区的消息之间不能保证先后顺序。



每个分区可以拥有多个副本replica，副本中会有一个leader副本，其他的为follower副本。leader副本处理生产者和消费者的请求，follower副本从leader副本复制消息。当leader副本不可用时，会从follower副本中选择一个leader副本。



每个partition会维护一个isr（In-Sync Replica），其中leader副本一定在isr中。当生产者acks参数配置为-1或者all时，leader只有在isr的副本都复制了这条消息时，才会返回写入成功。此时leader并不会在消息写入到所有副本才返回。



<img src="/开源框架/kafka/.assert/kafka汇总/image-20220418084234712.png" alt="image-20220418084234712" style="zoom:50%;" />



## 使用

### 搭建服务端

#### 下载软件包

https://www.apache.org/dyn/closer.cgi?path=/kafka/3.1.0/kafka_2.13-3.1.0.tgz

#### 启动

https://kafka.apache.org/quickstart

```shell
# 启动zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties
# 启动kafka，可以通过不同的配置文件，启动多个kafka
bin/kafka-server-start.sh config/server.properties
# 创建topic
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```



#### 配置文件

zookeeper.properties为zookeeper的配置文件。server.properties为kafka配置文件

### 生产消息

通过kafka客户端生产消息



**客户端常用配置**，其他配置可以在ProducerConfig类中找到

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



**添加依赖**

```xml
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>3.1.0</version>
    </dependency>
```

**实例代码**

```java
package com.hewutao.kafka;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;
import org.junit.Before;
import org.junit.Test;

import java.util.Properties;
import java.util.concurrent.Future;

@Slf4j
public class KafkaProducerTest {

    private KafkaProducer<String, String> producer;
    private final String topic = "topic-01";

    @Before
    public void before() {
        Properties config = new Properties();
        config.put("bootstrap.servers", "127.0.0.1:9092");
        config.put("key.serializer", StringSerializer.class.getName());
        config.put("value.serializer", StringSerializer.class.getName());

        producer = new KafkaProducer<>(config);

    }

    @Test
    public void produce() throws Exception {
        ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic, "testkey", "my name is hewutao6");
        Future<RecordMetadata> metadataFuture = producer.send(producerRecord);
        RecordMetadata recordMetadata = metadataFuture.get();

        log.info("recordMetadata: {}", recordMetadata);
    }
}

```



### 消费消息

通过kafka客户端消费消息



重要的参数





```java
package com.hewutao.kafka;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.junit.Before;
import org.junit.Test;

import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Properties;

@Slf4j
public class KafkaConsumerTest {
    private final String groupId = "myGroup";
    private final String topic = "quickstart-events";
    private KafkaConsumer<String, String> consumer;

    @Before
    public void before() {
        Properties config = new Properties();
        config.put("bootstrap.servers", "127.0.0.1:9092");
        config.put("key.deserializer", StringDeserializer.class.getName());
        config.put("value.deserializer", StringDeserializer.class.getName());
        // 手动提交commit
        config.put("enable.auto.commit", "false");
        config.put("group.id", groupId);

        consumer = new KafkaConsumer<>(config);
        consumer.subscribe(Collections.singleton(topic));

    }

    @Test
    public void consume() {
        log.info("start poll");
        while(true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(3));

            for (ConsumerRecord<String, String> record : records) {
                log.info("record: {}", record);

                commit(record);
            }
        }
    }

    private void commit(ConsumerRecord<String, String> record) {
        TopicPartition topicPartition = new TopicPartition(record.topic(), record.partition());
        OffsetAndMetadata offsetAndMetadata = new OffsetAndMetadata(record.offset());

        HashMap<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
        offsets.put(topicPartition, offsetAndMetadata);
        consumer.commitSync(offsets);
    }
}

```



### 事务

kafka提供了事务功能，保证在生产消息和消费消息可以实现原子性。

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





## 实现



### 节点运行机制

1. 每个节点都会竞选Controller角色，但同一个时间只会有一个节点竞选成功
2. Controller负责集群管理，主要管理topic。Controller处理来自客户端的请求和Zookeeper的事件
3. 每个broker会监听一个端口，用于接收客户端请求和其他broker节点的请求。



#### 请求处理流程

| 组件                | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| RequestChannel      | 保存接收到的响应                                             |
| Processsor          | 用于从Acceptor接收新连接，请求和发送响应。将请求放置到Processor。有多个Processor，每个Processor代表一个线程 |
| KafkaRequestHandler | 从Processor中获取请求，交给KafkaApis处理。有多个KafkaRequestHandler，每个handler代表一个线程 |
| KafkaApis           | 实际的处理请求类                                             |

<img src="/开源框架/kafka/.assert/kafka汇总/image-20220405114109997.png" alt="image-20220405114109997" style="zoom:50%;" />



### 主题



#### 主题元数据

主题的元数据保存在`/brokers/topics/<topic_name>`节点。该节点的数据是json格式，包含主题的分区和副本信息。

```json
{
    "partitions": {
        "0": [1,0],
        "1": [0,1],
        "2": [1,0],
        "3": [0,1]
    },
    "topic_id": "hRF5yTDISDyM47Qni4NEGw",
    "adding_replicas": {},
    "removing_replicas": {},
    "version": 3
}
```

分区的信息保存在节点`/brokers/topics/<topic_name>/partitions/<partition>/state`。节点的数据也是json格式，包含分区的leader和isr信息

```json
{"controller_epoch":4,"leader":1,"version":1,"leader_epoch":0,"isr":[1,0]}
```



1. 在创建主题时，controller会先创建`/brokers/topics/<topic_name>`节点，放置主题的分区和副本信息
2. controller会监听`/brokers/topics`子节点变化。当创建主题节点时，controller会接收到通知，开始通知broker创建副本，在创建过程中，controller会生成parition的leader和isr信息，并将这些信息保存到`/brokers/topics/<topic_name>/partitions/<partition>/state`节点。同时会监听`/brokers/topics/<topic_name>`节点变化
3. 在增加主题的partition时，Controller也是先把新分区和副本的信息保存到`/brokers/topics/<topic_name>`节点。controller会接收到节点变化的事件通知，开始通知节点创建新副本，同时把新partition的leader和isr信息保存到`/brokers/topics/<topic_name>/partitions/<partition>/state`节点。
4. 在reassign中，







### 创建主题

kafka创建主题的流程

1. 根据客户端提交的parition个数和replica-factor，生成replica分配方案
2. 将主题的分配方案保存到zookeeper节点/brokers/topic/<topic_name>上
3. 增加节点后，会触发Controller的listener，开始创建topic。
4. Controller会从每个parition的replica中选取一个leader并生成isr列表，最开始isr包含所有replica。并将partition的信息保存到zookeeper节点上。并且发送消息给replica所在broker，让他们在本地创建replica。



#### topic节点数据

每个topic

```json
{
    "partitions": {
        "0": [
            1,
            0
        ],
        "1": [
            0,
            1
        ],
        "2": [
            1,
            0
        ],
        "3": [
            0,
            1
        ]
    },
    "topic_id": "hRF5yTDISDyM47Qni4NEGw",
    "adding_replicas": {},
    "removing_replicas": {},
    "version": 3
}
```







### 生产消息



### 消费消息



### 事务

https://blog.csdn.net/qq_36628536/article/details/118089422



### rebalance



















