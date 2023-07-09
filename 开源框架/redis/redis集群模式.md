# 集群模式

哨兵模式和单机模式都只是单个节点提供服务，如果并发数高、数据量大时可以使用集群模式。集群模式下，多个redis节点同时提供服务，通过slot指定每个节点处理的请求。集群模式下，可以为主节点配置从节点，当主节点故障时，从节点升为主节点。





### 配置

```ini
#配置端口
port 6379
#以守护进程模式启动
#pid的存放文件
pidfile "/usr/local/etc/redis/redis_6379.pid"
#日志文件名
loglevel notice
logfile "/usr/local/etc/redis/redis_6379.log"
#存放备份文件以及日志等文件的目录
dir "/usr/local/etc/redis/data"  
#rdb备份文件名
dbfilename "dump_6379.rdb"
#开启集群功能
cluster-enabled yes
#集群配置文件，节点自动维护
cluster-config-file /usr/local/etc/redis/nodes_6379.conf
#集群能够运行不需要集群中所有节点都是成功的
cluster-require-full-coverage no

```



### 启动命令

```
docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf
```





### 相互通知

让集群知道其他节点

```
cluster meet {ip} {port}

redis-cli -h 172.17.0.2 cluster meet 172.17.0.4 6379
```



![image-20220220152018145](/开源框架/redis/.assert/redis集群模式/image-20220220152018145.png)

```


node1  172.17.0.2
node2  172.17.0.3
node3  172.17.0.4
node4  172.17.0.5
node5  172.17.0.6
node6  172.17.0.7
```



### 分配槽位



```shell
cluster addslots slot [slot ...]
```





```
Node1:0~5460
Node2:5461~10922
Node3:10923~16383
```



### 确定复制关系

```shell
cluster replicate {master_node_id}
```





```shell
docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf


docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node2:/usr/local/etc/redis --name cluster-node2 redis redis-server /usr/local/etc/redis/redis.conf

docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf

docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf

docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf

docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node1:/usr/local/etc/redis --name cluster-node1 redis redis-server /usr/local/etc/redis/redis.conf

docker run -d -v /Users/hewutao/docker/extfile/redis-cluster/node8:/usr/local/etc/redis --name cluster-node8 redis redis-server /usr/local/etc/redis/redis.conf
```





### 转移slots



```


redis-cli --cluster reshard --cluster-from  bb17d66d19b1f08668f22610bae750a0e4130e94 --cluster-to 93738114b2d75a5c3a8ee2d9b1043a61b68780c1 --cluster-slots 100  172.17.0.2:6379
```







### 其他命令

```
# 查看集群节点信息
cluster nodes 

# 查看集群概览信息
cluster info
```





## 实现原理

gossip



### 一、配置是如何在多个节点中同步的？

slot配置

如何更新configEpoch

手动reshard



addslots





```
7 2 0      7
节点1向节点2转移slot
7 8 0      8
添加节点4
7 8 0 9    9


```





节点配置

在ping和meet信息中随机携带节点信息

### 二、如何实现节点倒换？

从节点自己主动升主



### 正常运行

节点会定时（1次/s）向其他节点发出ping消息，每一次发送只会随机选择**1**个节点，接收到ping消息的节点会返回一个pong消息。



ping消息包括一下信息

1. 当前集群的的currentEpoch
2. 当前节点信息和部分其他节点信息，主要包含ip，端口和configEpoch信息。每次会包括1/10之一的节点信息和全部pfail的节点信息
3. 当前节点slot信息

![Gossip_PING](/开源框架/redis/.assert/redis集群模式/5b0a014046f04a19af921a449ab05522~tplv-k3u1fbpfcp-watermark.awebp)





节点定时任务：clusterCron

发送ping消息：clusterSendPing







### 节点加入

节点加入时，会执行cluster meet命令。

```shell
cluster meet <ip> <port> <cport>
```



处理方法为：clusterStartHandshake，将新节点添加到自己的数据结构中

redis会有10次/s的定时任务，在这个任务中，会向没有link的节点发送请求。其中对于刚刚加入的节点，还没有请求，节点会向它发送meet请求。meet请求与普通ping请求是一样的，只是消息类型不一样。

![pfail](/开源框架/redis/.assert/redis集群模式/e7bf4bcf7e7f47e684f3623239d8c68b~tplv-k3u1fbpfcp-watermark.awebp)

发送meet命令的clusterLinkConnectHandler



### 节点下线

当节点长时间（cluster_node_timeout）没有返回pong命令，节点将会被标记成pfail。当节点一发现有1半以上节点，认为节点二是pfail，则节点一会将该节点标记为fail，然后向其他节点广播节点二 fail的消息。fail消息与ping消息结构是一样，只是不包含其他节点信息和pfail节点信息。pfail状态的超时时间为2*node_timeout。





标记下线的方法：markNodeAsFailingIfNeeded



### 主备倒换

```
clusterRequestFailoverAuth
```

```
clusterHandleSlaveFailover
```

## 运维措施



### 手动倒换



