# druid机制



## 整体机制

<img src="/开源框架/druid/.assert/druid机制/image-20231108210124946.png" alt="image-20231108210124946" style="zoom:50%;" />





## 细节



### 创建连接

在activeCount+poolingCount<maxActive的情况下，有下面三种情况会直接开始创建连接，不会进行empty等待。

1. poolingCount < notEmptyWaitThreadCount，连接池中的空闲连接数，小于等待连接的线程数
2. keepAlive && activeCount + poolingCount < minIdle。开启了keeppAlive，并且总连接数小于最小空闲连接数
3. 创建连接连续报错



### 取连接

参数

1. maxWait，从连接池取连接最大等待时间ms，默认-1，表示需要一直等。
2. activeCount，当前从连接池取出的连接个数，不能大于maxActive
3. maxWaitThreadCount，最大的等待连接现场数，默认为-1，表示不校验
4. notEmptyWaitThreadCount，等待连接的线程数，如果maxWaitThreadCount>0，并且notEmptyWaitThreadCount>= maxWaitThreadCount，则获取连接立即失败
5. onFatalError，是否发生致命错误。在连接使用过程中，如果出现致命异常，则会设置为true
6. onFatalErrorMaxActive表示在发生致命异常时，activeActive最大值。默认为0
7. connectCount，进过一系列校验，开始从连接池获取连接时，会自动+1
8. failFast，是否快速失败。如果连接池中没有空闲，并且连续2次次创建连接失败，则不会继续等连接，直接抛出DataSourceNotAvailableException异常

<img src="/开源框架/druid/.assert/druid机制/image-20231108210909053.png" alt="image-20231108210909053" style="zoom: 25%;" />

从连接池取出后，需要进行连接检查

<img src="/开源框架/druid/.assert/druid机制/image-20231108211229045.png" alt="image-20231108211229045" style="zoom:25%;" />





### 归还连接

连接属性

1. disabled。disabled表示连接已经不能再使用。在连接出现致命异常时，会设置为true。连接回收完成后，也会设置为true
2. abandoned。表示连接被遗弃，会被直接kill掉。这种连接在关闭时，不会放回到连接池。
3. active。表示连接正在被使用



连接池属性

1. removeAbandoned。是否移除已放弃的连接
2. asyncCloseConnectionEnable。是否启动异步关闭线程。当获取连接和关闭连接不是同一个线程时，会被强制设置成true。
3. phyMaxUseCount。一个物理连接最多被使用次数，在连接归还时会检查，超过则会discard连接
4. testOnReturn。是否在归还时测试连接。



不回收连接的场景

1. 物理连接使用此时达到phyMaxUseCount配置的次数
2. testOnReturn开启时，连接测试没有通过
3. 物理连接使用达到phyTimeoutMillis配置的时间





### 处理连接致命问题



在连接执行sql时，如果发生致命异常，则会直接关闭连接
DruidDataSource#handleFatalError


mysql的异常处理器：**com.alibaba.druid.pool.vendor.MySqlExceptionSorter**



### 定时检查空闲连接

每间隔timeBetweenEvictionRunsMillis运行一次检查。运行的任务为DestroyTask。会做两种事情，清理多余的空闲连接和清理abandoned的链接


keepAlive检查时，会先从连接池把连接取出，检查通过后，才会放回。否者直接关闭。



<img src="/开源框架/druid/.assert/druid机制/image-20231108212002322.png" alt="image-20231108212002322" style="zoom:25%;" />