# 哨兵模式

在实际生产环境上，不会使用单机模式的redis。因为单机会出现性能和单点故障问题。redis的哨兵模式可以解决单点故障问题：

1. redis哨兵模式通过主从复制的方式，将数据复制给从节点，当主节点出现故障时，将从节点升为主。
2. 客户端通过可以通过哨兵获取主节点信息，从而可以自动连接到新主节点上，实现业务快速恢复。



## 从节点



配置文件中增加下面配置，或者执行下面的命令，可以在启动时与主节点建立主从关系。

```conf
replicaof 172.17.0.2 6379
```







## 哨兵节点

需要在配置文件中指定需要监控的主节点ip和端口信息，quorum表示多少台哨兵主观下线会开始从节点升主操作。

```properties
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached replica or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000

# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
#
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
#
# - The time needed for a replica replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
#
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted replica).
#
# - The maximum time a failover in progress waits for all the replicas to be
#   reconfigured as replicas of the new master. However even after this time
#   the replicas will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
#
# Default is 3 minutes.
sentinel failover-timeout mymaster 180000

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
# 
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# sentinel notification-script mymaster /var/redis/notify.sh
```



### 哨兵与主从节点和其他哨兵节点通信

#### 连接

sentinel会与每个已知的sentinel建立连接。sentinel会有主实例和从实例建立连接。每个节点会建立两个连接

1. 执行命令的连接
2. 订阅连接

#### 非选举时动作

1. sentinel会每1s（SENTINEL_PING_PERIOD）向其他sentinel或者主从实例发送ping，检查它们服务是否正常。如果超过down-after-milliseconds时间没有正常返回pong响应，那么认为节点下线了。

2. sentinel会订阅所有节点（主从和哨兵）的`__sentinel__:hello`通道，sentinel会每2s（SENTINEL_PUBLISH_PERIOD）在该通道发送消息，消息包含sentinel自己的信息和主节点的配置。

   ```
   sentinel_ip,sentinel_port,sentinel_runid,current_epoch,
   master_name,master_ip,master_port,master_config_epoch.
   ```

   sentinel在接收到`__sentinel__:hello`通道的信息时，会有下面三个动作：

   1. 根据runid和ip信息更新sentinel列表
   2. 如果current_epoch>自己的current_epoch，则更新自己的current_epoch为current_epoch
   3. 如果master_config_epoch>自己的master_config_epoch，则更新自己的master_config_epoch为master_config_epoch

3. sentinel会每10s（SENTINEL_INFO_PERIOD）向主从节点发送info命令，当主节点处于客观下线时，将会每1s发送info命令



分析：

1. sentinel通过订阅和发布`__sentinel__:hello`共享自己和master的信息。由于会向每个节点提供订阅和发布该通道，因此信息发布会非常可靠，不会因为少数节点故障导致无法共享信息。
2. 如果发现其他sentinel的current_epoch和配置比自己新（通过比较current_epoch和master_config_epoch），sentinel节点会更新自己的current_epoch和配置。这样可以保证每个节点可以获得新信息。
3. info命令用于获取主从节点的信息和主从关系。包括：主节点有哪些从节点，从节点与主节点连接情况，从节点的复制进度。
4. 通过通道和info命令，每个sentinel可以获取主从节点和其他sentinel的信息。





sentinelSendPeriodicCommands

sentinelTimer: 定时执行的函数

sentinelCheckSubjectivelyDown：检查主观下线

```
        master->leader = sdsnew(req_runid); // 选举的leader sentinel的run_id
        master->leader_epoch = sentinel.current_epoch; // 选举的leader sentinel的current_epoch
```



## 主从复制，过程







## 哨兵选举





## 哨兵从机升主的过程

判断节点不可用的规则

1. 长时间没有回复ping请求（act_ping_time上一次发出ping的时间，没有返回ping的最长时间，last_avail_time：上一次正常响应ping的时间）



选择从机的规则

1. 从机不能下线
2. 从机的权重不能为0
3. 从机当前是可以连接上
4. 上一次正常响应ping请求不超过5个SENTINEL_PING_PERIOD
5. 从机的信息不能过时（主机主观下线或客观下线时，不超过5个SENTINEL_PING_PERIOD，其他情况不超过3个SENTINEL_INFO_PERIOD）



从机排序规则

权重(priority)>复制位置(repl_offset)>run_id





1. 判断主节点主观下线
2. 询问其他节点，当判定客观下线时，执行下面的步骤
3. 竞选主节点
4. 执行从升主的操作



is-master-down-by-addr

```
IS-MASTER-DOWN-BY-ADDR <ip> <port> <current-epoch> <runid>
```



节点投票规则

1. 竞选节点的current_epoch要大于当前节点的current_epoch
2. 竞选节点的current_epoch要大于当前master的leader_epoch



节点给自己投票

1. 如果还没有其他节点投票，则给自己投票
2. 如果有其他已经投票，则给票数最多的节点投票
3. 当节点给其他人投票后，它发起failover的时间重新开始计算（也就是start_failover_time=now()）



节点的状态机

1. 没有开始failover
2. 开始了failover，等待投票
3. 选择从节点作为新主
4. 给新主节点发送slave of none
5. 等待新主节点升为主
6. 给其他从节点发送slave of {new master}





1. 什么时候会询问其他节点，主机是否下线。当自己发现主节点下线时（客观下线）
2. 什么时候开始从机升主（当发现自己获得的票数超过总票数的一半时）
3. 升级时间是多少？
4. 升级的动作有哪些？（询问其他节点主机的情况，竞选leader，通知新主节点称为主节点，让其他从节点从新主节点复制，更新配置）
5. 如何保证不会有多个节点同时开始竞争呢？当一个节点竞选成功，其他节点节点就无法竞选成功，如果超过时间min（failover_timeout,SENTINEL_ELECTION_TIMEOUT(默认10s)），其他节点将会停止failover，等待（2*failover_timeout）时间后再次failover。一个节点竞选成功后，其它节点无法竞选











### 如何判断主节点出故障



```c
    /* Only masters */
    if (ri->flags & SRI_MASTER) {
        sentinelCheckObjectivelyDown(ri); // 检查主节点是否是否客观下线
        if (sentinelStartFailoverIfNeeded(ri))
            sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_ASK_FORCED);
        sentinelFailoverStateMachine(ri);
        sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_NO_FLAGS);
    }
```



1. sentinel标志位SRI_MASTER_DOWN，表示它认为主节点下线
2. 主节点标志位SRI_O_DOWN，表示主节点客观下线
3. 主节点o_down_since_time，表示上一次客观下线的时间
4. 主节点failover_start_time是哨兵节点上一次开始failover操作的时间
5. 主节点failover_timeout是配置文件中配置的，默认180000（3分钟）
6. 主节点failover_epoch是failover时sentinel的current_epoch（每次sentinel开始failover时，会将自己的current_epoch加1）





```c
// 判断是否需要开始Failover
int sentinelStartFailoverIfNeeded(sentinelRedisInstance *master) {
    /* We can't failover if the master is not in O_DOWN state. */
    if (!(master->flags & SRI_O_DOWN)) return 0;

    /* Failover already in progress? */
    if (master->flags & SRI_FAILOVER_IN_PROGRESS) return 0;

    /* Last failover attempt started too little time ago? */
    if (mstime() - master->failover_start_time <
        master->failover_timeout*2)
    {
        if (master->failover_delay_logged != master->failover_start_time) {
            time_t clock = (master->failover_start_time +
                            master->failover_timeout*2) / 1000;
            char ctimebuf[26];

            ctime_r(&clock,ctimebuf);
            ctimebuf[24] = '\0'; /* Remove newline. */
            master->failover_delay_logged = master->failover_start_time;
            serverLog(LL_WARNING,
                "Next failover delay: I will not start a failover before %s",
                ctimebuf);
        }
        return 0;
    }

    sentinelStartFailover(master);
    return 1;
}

/* Setup the master state to start a failover. */
void sentinelStartFailover(sentinelRedisInstance *master) {
    serverAssert(master->flags & SRI_MASTER);
		// 状态机标志位
    master->failover_state = SENTINEL_FAILOVER_STATE_WAIT_START;
  	// 开始Failover
    master->flags |= SRI_FAILOVER_IN_PROGRESS;
  	// 增加current_epoch，开始新一轮拉票
    master->failover_epoch = ++sentinel.current_epoch;
    sentinelEvent(LL_WARNING,"+new-epoch",master,"%llu",
        (unsigned long long) sentinel.current_epoch);
    sentinelEvent(LL_WARNING,"+try-failover",master,"%@");
    master->failover_start_time = mstime()+rand()%SENTINEL_MAX_DESYNC;
    master->failover_state_change_time = mstime();
}
```





## 疑问

如果少部分节点与其他节点隔离了，他们的epoch会不会一直增加。



实验设计

1. 开启3个redis，1主和两从
2. 开启5个哨兵



场景构造

1. 停止其中3个哨兵
2. 停止主节点



预期结果

1. 从无法升主，两个哨兵的epoch一直增加

