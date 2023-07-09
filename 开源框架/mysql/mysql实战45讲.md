# mysql实战45讲



## 索引



多列索引，会根据字段的顺序排序，所以sql是否走索引需要与字段的顺序有关。

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617110454070.png" alt="image-20230617110454070" style="zoom:50%;" />





## mvcc

undo log

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617140007629.png" alt="image-20230617140007629" style="zoom:50%;" />

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617140043816.png" alt="image-20230617140043816" style="zoom:50%;" />



<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617140146000.png" alt="image-20230617140146000" style="zoom:50%;" />



<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617141322603.png" alt="image-20230617141322603" style="zoom:50%;" />



## 页

innodb_page_size=16k

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617151516646.png" alt="image-20230617151516646" style="zoom:50%;" />

数据时变长的或者过长，将会把数据保存到其他地方，通过指针引用。

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617151815401.png" alt="image-20230617151815401" style="zoom:50%;" />

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617152215693.png" alt="image-20230617152215693" style="zoom:50%;" />



<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617152324697.png" alt="image-20230617152324697" style="zoom:50%;" />



## free和flush链表

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617152825459.png" alt="image-20230617152825459" style="zoom:50%;" />



避免一次大查询把所有缓存的数据都换掉。

第二次访问时间-第一次访问时间>1s，冷数据可以迁移到热数据

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617153635637.png" alt="image-20230617153635637" style="zoom:50%;" />





## redolog

在数据修改时，会将修改保存到redolog内存中（对应着log buffer），在事务提交时，会将内存中的数据修改写入到redolog文件中。



如果redolog文件会重复使用，如果之前的redolog中的页还没有写入到磁盘中，将会触发checkpoint，主动写入页到磁盘。



<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617152324697.png" alt="image-20230617152324697" style="zoom:50%;" />



log buffer刷新到磁盘的时机控制。

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617154708049.png" alt="image-20230617154708049" style="zoom:50%;" />



## double write buffer

内存数据写入到double write buffer后，可以将redo log删除

<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617163335557.png" alt="image-20230617163335557" style="zoom:50%;" />



## change buffer

用于保存索引页的变化，提高写操作效率

1. 更新一条数据后，如果需要修改的索引没有加载到buffer pool中，将索引的变化保存到change Buffer中。
2. 如果下次要加载索引到内存中，将会应用changeBuffer到索引中，从而内存中会保存最新的索引。



<img src="/开源框架/mysql/.assert/mysql实战45讲/image-20230617152324697.png" alt="image-20230617152324697" style="zoom:50%;" />