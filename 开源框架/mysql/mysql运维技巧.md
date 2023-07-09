# mysql运维技巧



## 查询mysql使用内存



**1.buffer pool内存占用计算方式**  

buffer pool目前主要有innodb和myism2个存储引擎，还有一个是redo日志占用内存大小，innodb_log_buffer_size

```
mysql> SELECT ( @@innodb_buffer_pool_size
    ->          + @@innodb_log_buffer_size
    ->          + @@key_buffer_size ) / 1024 / 1024 AS MEMORY_MB;
+--------------+
| MEMORY_MB    |
+--------------+
| 584.00000000 |
+--------------+
1 row in set (0.00 sec)
```



```
SELECT ( @@innodb_buffer_pool_size + @@innodb_log_buffer_size + @@key_buffer_size ) / 1024 / 1024 AS MEMORY_MB;
```



**2.MySQL数据库会话占用内存大小** 

可以使用下面的公式计算一个会话占用内存的大小

```
mysql> SELECT ( @@read_buffer_size + @@read_rnd_buffer_size
    ->          + @@sort_buffer_size + @@tmp_table_size
    ->          + @@join_buffer_size + @@binlog_cache_size ) / 1024 / 1024 AS Session_MB;
+-------------+
| Session_MB  |
+-------------+
| 16.90625000 |
+-------------+
1 row in set (0.00 sec)
```



```
SELECT ( @@read_buffer_size + @@read_rnd_buffer_size + @@sort_buffer_size + @@tmp_table_size + @@join_buffer_size + @@binlog_cache_size ) / 1024 / 1024 AS Session_MB
```



**计算所有连接数占用的总内存大小。**

```
先查询出数据库有多少会话
mysql> select count(*) from INFORMATION_SCHEMA.PROCESSLIST;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

根据会话数去乘一个会话占用内存的大小，最后得到的就是会话占用的内存总大小。
```



## 最大连接数

所有用户最大连接数

```
select @@max_connections;
```

<img src="/开源框架/mysql/.assert/mysql运维技巧/image-20230618182741886.png" alt="image-20230618182741886" style="zoom:50%;" />

当前用户可以使用的连接数

```
select @@max_user_connections;
```

<img src="/开源框架/mysql/.assert/mysql运维技巧/image-20230618182838203.png" alt="image-20230618182838203" style="zoom:50%;" />



```
set global max_user_connections = 10;
```



## 查询session和关闭session



```
show full processlist;
```



```
kill <sessionId>
```



