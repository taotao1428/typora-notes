# mysql使用

## 编码

mysql中有存在charset和collation两种概念

1. charset表示数据的编码，就是utf8，latin1等
2. collation表示数据的排序方式。排序方式用于比较两个字符之间，谁大谁小。每一种charset可以对应1种或多种排序方式，并且有一个默认的排序方式



sql中哪些地方会用到charset和collation

1. 两个数据的比较，例如：`where a.name > b.name or a.nam > 'hwt'`。需要用到`name字段`和`hwt`的charset和collation。`hwt`是sql中的常量，sql中常量的编码由`character_set_connection,collation_connection`确定
2. 排序。例如：`order by name`。需要使用name字段的charset和collation
3. 函数。例如：`cancat(name, city)`。需要使用name字段和city字段的charset和collation，并且需要确定返回值的chaset和collation

如果两个数据的collation不一样，应该怎么选择？

https://dev.mysql.com/doc/refman/5.7/en/charset-collation-coercibility.html



### 查询系统支持的charset和collation

```sql
show character set; # 查询charset
show collation where charset='utf8'; # 查询collation
```

<img src="/开源框架/mysql/.assert/mysql使用/image-20220504102931334.png" alt="image-20220504102931334" style="zoom:50%;" />

<img src="/开源框架/mysql/.assert/mysql使用/image-20220504103127454.png" alt="image-20220504103127454" style="zoom:50%;" />



### charset和collation所起的作用

#### 连接

| 编码                                          | 说明                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| character_set_client                          | 表示客户端发送sql的编码，服务端会按照这个编码解析sql         |
| character_set_connection,collation_connection | 表示服务端会将sql再转码成该编码，主要用于指定sql中常量的编码。 |
| character_set_results                         | 表示服务端返回给客户端结果集的编码，如果为null或者binary，表示直接使用字段的编码，服务端不进行转换。 |



```sql
show variables where variable_name in ('character_set_client', 'character_set_connection', 'collation_connection', 'character_set_results')

# 不转换结果集的编码
SET character_set_results = NULL;
SET character_set_results = binary;
```

![image-20220504114347082](/开源框架/mysql/.assert/mysql使用/image-20220504114347082.png)



##### 交互过程

1. 客户端连接时，会向服务端发送一个charset和collation，服务端会使用该charset设置character_set_client，character_set_connection，character_set_results三个值。collation_connection设置成character_set_connection默认的collation。如果服务**不支持**客户端发送的charset和collation，将会使用服务端默认的charset和collation。
2. 客户端发送sql给服务端。服务端通过character_set_client解码sql。对于sql中的常量，服务端会再将其编码成character_set_connection。**因为客户端发送的报文中，不会包含当前sql的编码，服务端会使用character_set_client对sql解码，如果sql编码格式与character_set_client不一致，可能会导致数据乱码的问题**。
3. 服务端给客户端返回数据时，如果character_set_results不为或者binary，将会数据采用character_set_results编码，否者保持原编码。另外服务端返回的结果集中，会带有数据的编码。



#### 服务端编码

https://dev.mysql.com/doc/refman/5.7/en/charset-server.html

| 服务端编码                             | 说明             |
| -------------------------------------- | ---------------- |
| character_set_server，collation_server | 服务端编码和排序 |

服务端编码的用处

1. ~~作为客户端与服务连接时的默认编码~~
2. 作为创建数据库时的默认编码



##### 指定服务端编码

在启动服务时指定，所有连接生效

```shell
mysqld
mysqld --character-set-server=latin1
mysqld --character-set-server=latin1 \
  --collation-server=latin1_swedish_ci
```



修改配置文件

```ini
[mysqld]
character_set_server=utf8
```





通过过sql修改，当前连接生效

```sql
set character_set_server='utf8'
```



#### 数据库编码

| 数据库编码                                | 说明                 |
| ----------------------------------------- | -------------------- |
| character_set_database,collation_database | 表示数据库的默认编码 |

在创建数据库时，如果没有指定编码，将会使用character_set_server，collation_server。

在查询character_set_database,collation_database变量时，如果当前没有选择数据库，将会显示character_set_server，collation_server的值

```sql
CREATE DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]

ALTER DATABASE db_name
    [[DEFAULT] CHARACTER SET charset_name]
    [[DEFAULT] COLLATE collation_name]
    
create DATAbase gbk_test default charset=gbk;
```



#### 表和列的编码

它们没有对应的系统参数。它们是在创建表时指定，也可以创建后修改。

```sql
CREATE TABLE tbl_name (column_list)
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]]

ALTER TABLE tbl_name
    [[DEFAULT] CHARACTER SET charset_name]
    [COLLATE collation_name]

col_name {CHAR | VARCHAR | TEXT} (col_length)
    [CHARACTER SET charset_name]
    [COLLATE collation_name]

```





## 语法

```
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_id` INT UNSIGNED AUTO_INCREMENT,
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL,
   `submission_date` DATE,
   PRIMARY KEY ( `runoob_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



### 函数

#### 日期函数

https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html



```sql
# 获取当前日期时间datetime
select NOW()  from person limit 1;
# 获取utc日期和时间
select UTC_DATE(), UTC_TIME() from person limit 1;
```









## 用户权限

### 用户管理

mysql的用户都保存在mysql.user表中。mysql中，通过username和hostname唯一表示一个用户

#### 查询用户

```
use mysql
select * from user;
```



#### 创建用户

```
create user '{username}'@'{hostname}' identified by '{password}'
```



hostname表示账号可以从哪些客户端访问数据库

1. `%`表示所有地方
2. `ip`或者`网段`表示指定ip的客户端
3. `localhost`表示本机



```
create user 'hewutao'@'%' identified by '******'
```



#### 删除用户

```
drop user '{username}'@'{hostname}'
```



#### 修改密码

```
set password for 'username'@'hostname' = password('new password');
update user set password=password('new password') where user = 'username'
```





### 权限管理

权限的类型

| **权 限**                   | **作用范围**         | **作 用**                     |
| --------------------------- | -------------------- | ----------------------------- |
| **all**                     | 服务器               | 所有权限                      |
| **select**                  | 表、列               | 选择行                        |
| **insert**                  | 表、列               | 插入行                        |
| **update**                  | 表、列               | 更新行                        |
| **delete**                  | 表                   | 删除行                        |
| **create**                  | 数据库、表、索引     | 创建                          |
| **drop**                    | 数据库、表、视图     | 删除                          |
| **reload**                  | 服务器               | 允许使用flush语句             |
| **shutdown**                | 服务器               | 关闭服务                      |
| **process**                 | 服务器               | 查看线程信息                  |
| **file**                    | 服务器               | 文件操作                      |
| **grant option**            | 数据库、表、存储过程 | 授权                          |
| **references**              | 数据库、表           | 外键约束的父表                |
| **index**                   | 表                   | 创建/删除索引                 |
| **alter**                   | 表                   | 修改表结构                    |
| **show databases**          | 服务器               | 查看数据库名称                |
| **super**                   | 服务器               | 超级权限                      |
| **create temporary tables** | 表                   | 创建临时表                    |
| **lock tables**             | 数据库               | 锁表                          |
| **execute**                 | 存储过程             | 执行                          |
| **replication client**      | 服务器               | 允许查看主/从/二进制日志状态  |
| **replication slave**       | 服务器               | 主从复制                      |
| **create view**             | 视图                 | 创建视图                      |
| **show view**               | 视图                 | 查看视图                      |
| **create routine**          | 存储过程             | 创建存储过程                  |
| **alter routine**           | 存储过程             | 修改/删除存储过程             |
| **create user**             | 服务器               | 创建用户                      |
| **event**                   | 数据库               | 创建/更改/删除/查看事件       |
| **trigger**                 | 表                   | 触发器                        |
| **create tablespace**       | 服务器               | 创建/更改/删除表空间/日志文件 |
| **proxy**                   | 服务器               | 代理成为其它用户              |
| **usage**                   | 服务器               | 没有权限                      |



#### 查询权限

```
show grants for 'username'%'hostname'
```



#### 授权

```
grant {权限} on {库}.{表} to 'username'%'hostname' 

grant {权限} on {库}.{表} to 'username'%'hostname' with grants
```



```
grant all on *.* to 'username'%'hostname'
```



#### 取消授权

```
revoke {权限} on {库}.{表} from 'username'%'hostname'
```







## 索引



### 索引类型

1. 聚簇索引
2. B+索引



## sql执行计划

当sql执行比较慢时，可以通过explain命令获取sql的执行计划，进一步分析原因



```
explain select * from person where id = 'lskdfklsdflkdlasew';

id|select_type|table |partitions|type |possible_keys|key    |key_len|ref  |rows|filtered|Extra|
--+-----------+------+----------+-----+-------------+-------+-------+-----+----+--------+-----+
 1|SIMPLE     |person|          |const|PRIMARY      |PRIMARY|194    |const|   1|   100.0|     |
```





### 索引类型（Type）

索引类型是指查询结果在join中的类型



```
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge  > unique_subquery > index_subquery > range > index  > ALL

system: 只有一行的系统表，这种表会加载到内存中，不会设计磁盘io
const: 常量等号查询，最多只有一条数据
eq_ref：在join场景，等号查询，每次查询最多只有一条数据
ref：在join场景，等号查询，可能查询出多条数据
fulltext：匹配到fulltext索引
ref_or_null：在ref的基础上，加上列为null的数据。name = 'hwt' or name is null;
index_merge: 将多个索引查询的结果合并成一个。https://dev.mysql.com/doc/refman/5.7/en/index-merge-optimization.html
unique_subquery：将子查询转化为join。value IN (SELECT primary_key FROM single_table WHERE some_expr)
index_subquery：将子查询转为join。value IN (SELECT key_column FROM single_table WHERE some_expr)
range：范围查询
index：只扫描索引
all：全表扫描
```





参考

https://zhuanlan.zhihu.com/p/358920539



#### system

表示表中只有一条数据，是const类型的一种特殊形式



#### const

表示查询的结果可以被看做一个常量。

当where条件中使用主键或者唯一索引与常量比较时，此时最多产生一条数据。

下面的sql中，tble_name可以看做一个常量，因为最多只能查询出一条数据。

```sql
SELECT * FROM tbl_name WHERE primary_key=1;

# 主键的两个列都与常量比较相等
SELECT * FROM tbl_name
  WHERE primary_key_part1=1 AND primary_key_part2=2;
```



#### eq_ref

表示在与之前的表进行join时，每次级联查询最多只能查询一条数据。

当join的on字段为primary key或者unique not null key时，并且操作符为`=`



下面sql中，ref_table可以看做eq_ref

```sql
SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

# 主键有2列组成，其中一列为常量，另外一列与其他表的列比较相等
SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
```



#### ref

表示在与之前的表进行join时，每次级联查询最可能查询出多条数据。

当join的on字段为非主键或、非唯一非空键、多列索引的前缀时，比较操作为`=`,`<=>`



```sql
# 非唯一索引
SELECT * FROM ref_table WHERE key_column=expr;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
```



#### ref_or_null

与ref一致，只是多了为null的可能

```java
SELECT * FROM ref_table
  WHERE key_column=expr OR key_column IS NULL;
```







## 复制



### 初始备机

当需要做主从复制时，如果主机已经有数据，需要先将主机的数据先同步到备机，然后再通过binlog使用主备复制。初始备机有两种

1. 逻辑备份。使用mysqldump从主机导出sql，再在备机执行sql，这种方式适用于数据小的情况
2. 物理备份。使用xtrabackup在主机做备份，然后在备机上恢复，这种方式适用于数据大的情况



可以指定备机服务端使用仅复制哪些数据库或者忽略哪些数据库

https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html



### 开启复制



#### 开启binlog

```
[mysqld]
# 配置serverid
server-id=1
# 配置binlog和redolog的写入时间
innodb_flush_log_at_trx_commit=1
sync_binlog=1

#设置日志三种格式：STATEMENT、ROW、MIXED 。
binlog_format = ROW
#设置日志路径，注意路经需要mysql用户有权限写
log-bin = /usr/local/mysql/data/bin_log/mysql-bin
#设置binlog清理时间
expire_logs_days = 7
#binlog每个日志文件大小
max_binlog_size = 100m
#binlog缓存大小
binlog_cache_size = 4m
#最大binlog缓存大小
max_binlog_cache_size = 512m
```



#### 开启GTID（全局事务ID）

在mysql配置文件中添加下面配置

```
gtid_mode=ON
enforce-gtid-consistency=ON
```



开启GTID后，每个事务都会有唯一的GTID，可以保证相同事务不会被重复执行



#### 创建复制账号

```sql
CREATE USER 'repl'@'%.example.com' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.example.com';

CREATE USER 'repl'@'192.168.3.%' IDENTIFIED BY '******';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.3.%';
```



#### 配置从节点

使用change master to修改master配置，再使用start slave 开始复制

```
CHANGE MASTER TO option [, option] ... [ channel_option ]

option: {
    MASTER_BIND = 'interface_name'
  | MASTER_HOST = 'host_name'
  | MASTER_USER = 'user_name'
  | MASTER_PASSWORD = 'password'
  | MASTER_PORT = port_num
  | MASTER_CONNECT_RETRY = interval
  | MASTER_RETRY_COUNT = count
  | MASTER_DELAY = interval
  | MASTER_HEARTBEAT_PERIOD = interval
  | MASTER_LOG_FILE = 'source_log_name'
  | MASTER_LOG_POS = source_log_pos
  | MASTER_AUTO_POSITION = {0|1}
  | RELAY_LOG_FILE = 'relay_log_name'
  | RELAY_LOG_POS = relay_log_pos
  | MASTER_SSL = {0|1}
  | MASTER_SSL_CA = 'ca_file_name'
  | MASTER_SSL_CAPATH = 'ca_directory_name'
  | MASTER_SSL_CERT = 'cert_file_name'
  | MASTER_SSL_CRL = 'crl_file_name'
  | MASTER_SSL_CRLPATH = 'crl_directory_name'
  | MASTER_SSL_KEY = 'key_file_name'
  | MASTER_SSL_CIPHER = 'cipher_list'
  | MASTER_SSL_VERIFY_SERVER_CERT = {0|1}
  | MASTER_TLS_VERSION = 'protocol_list'
  | IGNORE_SERVER_IDS = (server_id_list)
}

channel_option:
    FOR CHANNEL channel

server_id_list:
    [server_id [, server_id] ... ]
```



```
change master to MASTER_HOST='192.168.3.69',MASTER_USER='repl',MASTER_PASSWORD='******'  ,MASTER_AUTO_POSITION=1;
start slave;
```







### 复制的范围



### 复制的进度



### binlog解析



## 备份



### mysqldump

mysqldump是mysql自带的备份工具

https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html

#### 备份数据

```
mysqldump [options] db_name [tbl_name ...]
mysqldump [options] --databases db_name ...
mysqldump [options] --all-databases

常用连接参数
1. --host -h: mysql的ip
2. --user -u:连接的用户
3. --password -p:连接的密码
4. --socket -S: 使用socket连接

常用导出参数
1.--single-transaction: 在一个事务中导出数据，保证一致性
2.--quick：加快导出速度，大表可以加上该参数

常用过滤参数
1. --ignore-table=db_name.tbl_name：过滤表
2. --no-data, -d：不导出数据，只导出表结构
3. --where='where_condition', -w 'where_condition'：只导出满足条件的行

常用复制属性
1. --master-data:导出的sql中会包含change master to和位置，可以直接给从库使用

```





```
mysqldump -h192.168.3.69 -uhewutao -p****** --databases test_db > test_db_database.sql

mysqldump -h192.168.3.69 -uhewutao -p****** -A > all_database.sql

mysqldump -h192.168.3.69 -uhewutao -p****** --master-data --databases test_db > test_db_master_data.sql
```





### xtrabackup

https://github.com/percona/percona-xtrabackup/tags

```sql
# 使用 `xtrabackup --backup` 执行备份
xtrabackup --backup --host=127.0.0.1 --user=hewutao --password='******' --target-dir=/root/xtrabackup/bk_20220508

# `xtrabackup --prepare` 执行准备。
# 如果要将本次备份的数据作为后续增量备份的基础，应该使用 `xtrabackup –apply-log-only`（见增量备份步骤）
xtrabackup --prepare --target-dir=/root/xtrabackup/bk_20220508

# 恢复备份
xtrabackup --copy-back --target-dir=/root/bk_20220508 --datadir=/usr/local/mysql/data
 chown -R mysql:mysql /usr/local/mysql/data
```



问题？

1. 备份之后是否可以通过gtid设置进度？



```sql
# 1. 在备份xtrabackup_binlog_info找到当前binlog的位置
# mysql-bin.000005        1046    1fd6b13f-c9eb-11ec-b339-000c29e8420c:1-3

# 2. 服务启动后，手动指定位置
change master to MASTER_HOST='192.168.3.69',MASTER_USER='repl',MASTER_PASSWORD='******' ,MASTER_LOG_FILE = 'mysql-bin.000005',MASTER_LOG_POS = 1046
```



1. 如何使用binlog增量备份？



```sql

```



### mysqlbinlog

mysqlbinlog用于解析binlog成sql。当使用binlog实现增备时，就需要使用mysqlbinlog解析binlog日志。

```
Usage: mysqlbinlog [options] log-files
1. --base64-output=[decode-rows|never|auto] 是否将base64的sql解析出来。默认为auto：对于非rows格式的events，将会解析sql，rows格式的event，仍保存base64。decode-rows:将rows格式的events也解析sql，并放在注释中
2. -d, --database=name。仅解析指定数据库的事件
3. --server-id=#。仅解析某个mysql服务的时间
4. -j, --start-position=#。开始读取binlog日志的位置
5. --stop-datetime=name。停止读取binlog日志的时间。参数格式为2022-01-01 10:10:10。停止在第一个等于或者大于该时间的前面
6. --stop-position=#。停止读取binlog的位置



mysqlbinlog --start-position=1046 --base64-output=decode-rows --stop-position=1359 mysql-bin.000005
```





起始点

mysql-bin.000006 517

mysql-bin.000006 2189

```shell
mysqlbinlog --start-position=517 --stop-position=2189 mysql-bin.000006 | mysql -h192.168.3.72 -uhewutao -p******
```



```
set @@SESSION.GTID_NEXT='1fd6b13f-c9eb-11ec-b339-000c29e8420c:5'
```





![image-20220510082608841](/开源框架/mysql/.assert/mysql使用/image-20220510082608841.png)

<img src="/开源框架/mysql/.assert/mysql使用/image-20220510082701828.png" alt="image-20220510082701828" style="zoom: 50%;" />

## 问题



### 连接不上

```
➜  ~ mysql -h192.168.3.69   -uhewutao -p******
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.3.69:3306' (61)
```

服务器开启了防火墙，关闭防火墙后，可以正常连接