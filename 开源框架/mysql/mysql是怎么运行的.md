# mysql是怎么运行的



## 第4章　从一条记录说起——InnoDB记录存储结构　55 

行格式有*REDUNDANT* ，*COMPACT*、*DYNAMIC*（默认）、以及*COMPRESSED*



1. 如果表中没有定义主键，将会选择一个不为null的唯一索引的列为主键，如果没有这样一列，mysql将会添加名为row_id的隐藏列作为主键。
2. mysql对定长的判断是根据字节是否定长，如果列的字符是定长，但是每个字符的字节是不定的，此时该列依然不是定长。
3. 对于冗余行格式，对于固定字符长度的列，将会直接分配最长长度。
4. DYNAMIC特点是，将溢出列所有数据都保存在溢出页中，列中只保存响应的溢出页地址。
5. COMPRESSED在DYNAMIC基础上，会对列数据进行压缩。





## 第5章　盛放记录的大盒子——InnoDB数据页结构　72 

1. 每条记录的next_recode执行下一条记录的数据部分开头位置的偏移量（注意，不是下一条记录的开头位置的偏移量）。
2. 下一条记录可能会出现在当前记录的前面，此时偏移量为负数。
3. 会存在两条虚拟记录，这两条记录分别是最小数据和最大数据。分别处于记录链表的头部和尾部。
4. PageDirectory是记录的索引，加速数据查找。
5. file Trailer中保存校验和和LSN，与FileHeader中的相对应，用于校验页数据是否完整。





![image-20230617193228161](/开源框架/mysql/.assert/mysql是怎么运行的/image-20230617193228161.png)



<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230617193707853.png" alt="image-20230617193707853" style="zoom:50%;" />

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230617193736072.png" alt="image-20230617193736072" style="zoom:50%;" />



## 第8章　数据的家——MySQL的数据目录　132 

1. 系统表空间保存在idbdata1文件中，该文件的名称和大小可以通过参数innodb_data_file_path和innodb_data_home_dir修改，例如：innodb_data_file_path=*datal:512M;data2:512M:autoextend*

2. 每个表可以有独立表中间，每个表对应一个`.ibd`和`.frm`文件。

3. 当的`innodb_file_per_table`值为0 时，表示使用系统表空间，当`innodb_file_per_table`的值

   为 1 时，表示使用独立表空间

4. mysql默认有4个数据库，mysql中保存用户的账号信息、存储过程和运行产生的日志信息。information_schema保存表、视图、触发器和索引等信息。performance _ schema主要保存一些mysql执行过程的性能数据，例如最近执行哪些sql，使用多少内存。sys主要是通过视图的方式汇总information_schema和performance _ schema中的信息。



## 第9章　存放页面的大池子——InnoDB的表空间　140 

1. 区：64个连续的页组成区。对于16k每页的话，连续64个页组成1个区，所以区默认大小为1M

2. 组：256个连续的区组成组。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624170239358.png" alt="image-20230624170239358" style="zoom:50%;" />

3. 段：段是区的集合，是一个逻辑的概念。每个索引的叶子节点和非叶子节点属于两个段。

4. 碎片区：为了优化空间利用率，每个段最开始是以页为粒度，从碎片区申请空间。当申请的页超过32个后，才开始以区位粒度申请空间。

5. 区的类型：FREE：还没被分配的页；FREE_FRAG：页还没被分配完的碎片区；FULL_FRAG：页都被分配完的碎片区；FSEG：属于某个段的区。

6. XDES_Entry：区的描述符。是一个链表结构，可以指向前后两个Entry。State为区的类型，前面讲过。Page State Bitmap是描述区中每个页是否被使用。总共128位，每个页占用2位。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624172733205.png" alt="image-20230624172733205" style="zoom:50%;" />

7. 区的链表。为了更好的查找区，将相同类型的区通过链表组织在一起。FREE类型的页通过一条FREE链表组成；FREE_FRAG类型的页分为通过FREE FRAG和FULL FRAG两个链表组成。每个段的FSEG类型页面，通过FREE、NOT_FULL和FULL三条链表组成。

8. List Base Node。为了方便查找链表，通过List Base Node保存链表的首尾节点和链表长度。上面介绍介绍的链表都有对应的List Base Node。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624174834691.png" alt="image-20230624174834691" style="zoom:50%;" />

9. INODE Entry。用于描述段，前面介绍了，段里面包含32个碎片区的页和一些区。其中区通过三个链表组织，段里面就包含这三个链表的List Base Node

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624175059837.png" alt="image-20230624175059837" style="zoom:50%;" />

10. 页的类型。FSP_HDR：里面包含整个表空间和本组内区的信息；IBUF_BITMAP：里面保存Change Buffer相关的信息；INODE：保存每个段中的信息；XDES：保存本组内区的信息，与FSP_HDR类似。

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624170806638.png" alt="image-20230624170806638" style="zoom:50%;" />

11. FSP_HDR。第1个组的第一个区的第一个页为FSP_HDR类型，里面保存了该组中每个区的XDES Entry。另外还有FileSpaceHeader，里面包含了表空间的信息。

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624180655938.png" alt="image-20230624180655938" style="zoom:50%;" />

12. FileSpaceHeader。其中包括了属于表空间的三个链表的List Base Node。以及INODE类型页面组成的2条链表（因为当INODE Entry比较多时，需要多个INODE页面保存）

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624180957908.png" alt="image-20230624180957908" style="zoom:50%;" />

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624181635284.png" alt="image-20230624181635284" style="zoom:50%;" />

13. XDES 类型页面。每个组（除第一个组）的第一个页面为XDES页面。与FSP_HDR类似，只是少了FileSpaceHeader信息。

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624182022401.png" alt="image-20230624182022401" style="zoom:50%;" />

14. IBUF_BITMAP 类型页面。用于保存Change Buffer相关的东西

15. INODE 类型页面。用于保存段的INODE Entry，每个页面最多保存85个INODE Entry。

    <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624182511281.png" alt="image-20230624182511281" style="zoom:50%;" />

16. 系统表空间相对于独立表空间，多了内部系统相关信息，例如changeBuffer页面，内部系统表数据。







## 第 11章　两个表的亲密接触——连接的原理　178 

1. 由驱动表的数据查询被驱动表数据
2. where中的条件会被下推到查询驱动表数据和被驱动表数据。
3. join方式有为嵌套循环连接( Nest-Loop Join) 和基于块的嵌套循环连接（Nested-Loop Join）





## 第 12章　谁最便宜就选谁——基于成本的优化　190 

1. mysql会基于成本优化执行计划。优化方式包括选择合适的索引，选择合适的join顺序，是否创建临时表等。

2. mysql对于每个操作的代价都进行了量化。例如从磁盘读取一页的代价为1，判断一条记录是否匹配的代价为0.2。这些代价保存在`mysql.server _cost`和`mysql.engine_cost`表中，可以修改。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230618172227785.png" alt="image-20230618172227785" style="zoom:50%;" />

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230618172255321.png" alt="image-20230618172255321" style="zoom:50%;" />

3. 对于in过滤数据数量的统计，根据in后面的关键字数量大小，mysql可能会通过索引或者表统计数据计算数量。eq_range_index_dive_limit参数配置走索引的关键字数量最大值，默认200。`show index from <table>`可以查询表索引的统计值。



## 第 17章　调节磁盘和CPU的矛盾——InnoDB的Buffer Pool　278 

1. innodb_buffer_pool_size参数控制buffer pool的大小，默认125M，最小5M

2. 每页数据除了有页数据，每个页都对应一个控制块。控制块中保存页的表空间、页号等信息。控制块的大小大约为800字节，是页大小的5%左右，所以mysql申请的空间会比innodb_buffer_pool_size大5%左右

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624210712869.png" alt="image-20230624210712869" style="zoom:50%;" />

3. free链表。用于保存buffer pool中空闲的页面

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624211232012.png" alt="image-20230624211232012" style="zoom:50%;" />

4. 数据页hash处理。为了方便查询某一页是否已经加载到内存中，会将已加载的页放置到hash表中，key为表空间+页号

5. flush链表。用于保存所有的被修改的脏页，方便刷入到磁盘中。

6. lru链表。为了方便内存页的淘汰，通过lru链表记录每页的访问情况。lru链表分为2热数据区和冷数据区，默认热数据占37%，由参数innodb_old_blocks_pct控制。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624212840030.png" alt="image-20230624212840030" style="zoom:50%;" />

7. 刷新内存页到磁盘的策略分为BUF FLUSH LRU和BUF-FLUSH-LIST两种。第一种是从lru链表的尾部查找若干页刷入磁盘，查询的页受参数innodb_Iru_scan_depth控制。第二种就是从flush链表中查找脏页刷入到磁盘。

8. 如果一个buffer pool很大，可能会影响系统速度，通过系统参数innodb_buffer_pool_instances修改buffer pool的个数。所有buffer pool平分innodb_buffer_pool_size配置的值

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624214102074.png" alt="image-20230624214102074" style="zoom:50%;" />

9. mysql在5.7.5之后，申请内存从一次性申请修改为按需申请，每次申请的大小默认为125M，由参数innodb_buffer_pool_chunk_size控制。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230624214509041.png" alt="image-20230624214509041" style="zoom:50%;" />

10. 查询buffer pool的信息。SHOW ENGINE INNODB STATUS。



## 第 18章　从猫爷借钱说起——事务简介　294 

1. 一致性是表示事务执行结束后，当前数据满足设置的一致性条件，例如主键，唯一键，外键约束。
2. 隐式提交。当执行一些操作时，会自动触发事务提交。例如执行DDL语句，执行事务控制语句等。



## 第 19章　说过的话就一定要做到——redo日志　308 









## 第 21章　一条记录的多副面孔——事务隔离级别和MVCC　379 

1. 事务的一致性问题

   1. 脏写，事务A修改了事务B正在修改且未提交的数据，**引发事务A和事务B写覆盖**。（这种会通过锁等待解决）
   2. 脏读，一个事务读取到另外一个事务正在修改且未提交的数据（通过事务隔离级别解决）
   3. 不可重复读，一个事务读取数据后，另外一个事务修改数据后并提交，事务再次读取数据时数据出现跟之前不一样（事务隔离级别解决）
   4. 幻读，事务A查询一批数据后，事务B删除或者添加了一条符合事务A查询规则的记录。（并发问题，只能串行解决）

2. 事务隔离级别可能出现的问题

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230701155829136.png" alt="image-20230701155829136" style="zoom:50%;" />

3. 事务隔离级别变量，transaction_isolation

4. 版本链

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230701160801573.png" alt="image-20230701160801573" style="zoom:50%;" />

5. 可重复读是在第一次执行select时才会创建readview，而不是在开启事务时创建。

6. 事务时可重复读，并且事务长事件不提交，会导致undo log无法清理，导致表空间数据变大。



## 第 22章　工作面试老大难——锁　401 

1. MVCC会使用行锁

2. mysql支持表级锁，并且在加行锁时，会在表上加对应的意向锁，避免表锁和行锁冲突。

3. gap lock，Next-Key Lock，Insert Tntention Lock（意向锁），当加了行锁时，会顺带在表上加意向锁。

4. 在可重复读的隔离级别下，如果执行了update语句，可能会更新了其他事务提交的数据，导致下次能查询出这些数据，出现幻读。

5. 对于不同事务隔离级别，mysql加锁的范围也不一样。

6. 对于不大于read commited级别范围会比不小于repeated read级别范围小。

7. next key锁等于行锁+间隙锁，间隙锁加在记录的左边间隙

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230701181341325.png" alt="image-20230701181341325" style="zoom:50%;" />

8. 执行SHOW ENGINE INNODB STATUS查看数据库的状态，也会输出加了事务信息。如果设置SET GLOBAL innodb_status_output_locks=ON，将会输出详细的锁信息。

   <img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230701191724428.png" alt="image-20230701191724428" style="zoom:50%;" />





## 加锁测试

对repeated-read隔离级别测试



```
CREATE TABLE `test_db` (
  `id` int(11) NOT NULL,
  `card_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `card_id` (`card_id`),
  KEY `age_idx` (`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```



<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702120312247.png" alt="image-20230702120312247" style="zoom:50%;" />



### 主键 + 唯一

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702120327128.png" alt="image-20230702120327128" style="zoom:50%;" />

可以看出在test_db表中加了一个意向锁，在id=2的记录上加了行锁。说明主键唯一查找只会锁查找到的记录。**即使主键匹配，但是其他字段不匹配，主键匹配到的记录依然会被加锁**。

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702120410639.png" alt="image-20230702120410639" style="zoom:50%;" />

### 主键 + 范围

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702121415634.png" alt="image-20230702121415634" style="zoom:50%;" />

表上加了意向锁，id=3,4,5,7的记录都加锁了。**其中id=7的记录不在结果中却依然加入了next-key锁**

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702121458834.png" alt="image-20230702121458834" style="zoom:50%;" />



### 二级唯一索引+唯一

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702124632929.png" alt="image-20230702124632929" style="zoom:50%;" />

加了表级意向锁。并且二级索引中card_id=5被加行锁，主键中id=1的记录被加行锁。

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702124806261.png" alt="image-20230702124806261" style="zoom:50%;" />



### 二级唯一索引+范围

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702125126408.png" alt="image-20230702125126408" style="zoom:50%;" />

加了表级意向锁

card_id索引中card_id=5,4,8的记录被加next-key锁。其中card_id=8是二级索引中查询的最后一条记录，也会被加锁。

主键索引中id=1,2的记录被加行锁



<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702125242191.png" alt="image-20230702125242191" style="zoom:50%;" />



### 非唯一索引+唯一

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702125612046.png" alt="image-20230702125612046" style="zoom:50%;" />

加了表级意向锁

age_idex索引中age=3的两条记录加了next-key锁，另外age=4加了间隙锁

主键中id=1,3的两条记录加了行锁。



<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702130057840.png" alt="image-20230702130057840" style="zoom:50%;" />



### 非唯一索引+范围

强制走索引

<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702131102964.png" alt="image-20230702131102964" style="zoom:50%;" />



加了表级意向锁

二级索引age_idx中age=3,4,6的3条记录都加了next-key锁

主键索引id=1,2,3加了行锁。



<img src="/开源框架/mysql/.assert/mysql是怎么运行的/image-20230702131039065.png" alt="image-20230702131039065" style="zoom:50%;" />
