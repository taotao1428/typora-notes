# redis整体知识



## redis的优点和缺点（为什么使用redis？）

优点

1. redis是一个基于内存的KV数据库，速度快（每秒10w的读写速度）
2. 有丰富的数据结构（string、list、set、sorted set、hash）
3. 支持数据持久化（rdb，aof）
4. 丰富的特性（发布订阅功能，key过期策略，事务，支持多个DB，技术）



缺点

1. 储存的数据量比较小，与内存大小有关
2. redis是单线程，无法利用服务器多核的优势

## 数据类型和使用场景

### string

1. 缓存
2. 计数
3. 保存全局session

### list

1. 大列表分页
2. 消息队列

### set

1. 去重
2. 用户的共同爱好



### sorted set

1. 排行榜



### hash

1. 保存结构化数据
2. 实现分布式锁

## 持久化

redis持久化有AOF和RDB两种

### AOF

AOF是增量备份，当执行写命令时，会将执行的命令写到备份中

### RDB

RDB是全量备份，可以手动触发，也可以根据配置自动触发



两者对比

1. AOF支持实时，但是效率低，备份文件较大
2. RDB不支持实时，但是效率高，备份文件较小



## Key的淘汰策略

当内存不足时，redis需要根据淘汰策略删除部分key，否者redis将不能再写入数据。

1. noevict不淘汰
2. volatile-lru 在设置了失效时间的key中，使用lru策略
3. allkeys-lru 在所有key中，使用lru策略
4. volatile-random 在设置了失效时间的key中，使用随机策略
5. allkeys-random在所有key中，使用随机策略
6. volitile-ttl 删除最快失效的key

通常淘汰有以下三个策略：

1. fifo 先进先出
2. lru 最近不使用
3. lfu 最近不经常使用

## 不同集群模式

集群模式

1. 单机 仅部署一台机器
2. 哨兵模式 主从部署，当主机出现异常时，哨兵将会选择一个备机升主
3. 集群模式 集群部署，多个主机同时工作，每个主机负责不同的slot



## 其他功能

1. sub/pub 订阅发布
2. 事务 同时执行或在同时不执行，就算中间某个命令失败，仍会执行后面的命令。multi exec
3. pipeline 多个命令一起执行，一起返回结果
3. lua 执行lua脚本
4. bitmap 位图，实现布隆过滤器
5. **HyperLogLog** 实现不精确去重
6. Geospatial地理位置距离计算



## 实战

