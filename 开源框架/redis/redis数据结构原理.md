# redis数据结构原理

redis提供了5种数据类型，本文将探究这5种数据类型的底层实现。



## 数据对象

redis每个key值对应一个对象，对象的属性包括

1. 对象类型（对应redis的5中数据类型：string，list，set，zset，hash）
2. 底层数据编码类型（rawstr，ziplist等）
3. 底层数据指针
4. 对象引用次数
5. 对象上次访问时间

```c
typedef struct redisObject{
     //类型
     unsigned type:4;
     //编码
     unsigned encoding:4;
     //指向底层数据结构的指针
     void *ptr;
     //引用计数
     int refcount;
     //记录最后一次被程序访问的时间
     unsigned lru:22;
}robj
```



## 底层数据编码

底层数据用于保存对象中包含的数据，有以下8种编码类型

| 数据结构                  | 说明                                              |
| ------------------------- | ------------------------------------------------- |
| REDIS_ENCODING_INT        | long整形                                          |
| REDIS_ENCODING_EMBSTR     | 嵌入到数据对象的字符串                            |
| REDIS_ENCODING_RAW        | redis自己的字符串结构                             |
| REDIS_ENCODING_ZIPLIST    | 压缩list                                          |
| REDIS_ENCODING_SKIPLIST   | 跳表                                              |
| REDIS_ENCODING_LINKEDLIST | 双端链表（已经不再使用了）                        |
| REDIS_ENCODING_HASH       | hash表                                            |
| REDIS_ENCODING_INTSET     | 整数集合                                          |
| REDIS_ENCODING_QUICKLIST  | 双端ziplists（双端表，但是每个元素是一个ziplist） |
| REDIS_ENCODING_ZIPMAP     | Zipmap                                            |



```
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* Encoded as zipmap */
#define OBJ_ENCODING_LINKEDLIST 4 /* No longer used: old list encoding. */
#define OBJ_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6  /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8  /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9 /* Encoded as linked list of ziplists */
#define OBJ_ENCODING_STREAM 10 /* Encoded as a radix tree of listpacks */
```


### redis字符串

redis没有使用C字符串，而是使用新的数据结构表示字符串。如下图，redis字符串对象中包含了free，len，buf三个属性，其中

1. free表示buf中剩余的空间大小
2. len表示buf中实际使用的空间大小
3. buf表示存放数据的char[]数组

![img](/开源框架/redis/.assert/redis数据结构原理/1120165-20180530075617307-666390009.png)



redis字符串相对C字符串的优点

1. 二进制安全。C字符串使用\0表示字符串结束，如果字符串用于保存二进制数据，中间可能会存现\0，导致读取字符串提前终止。而redis字符串使用len表示字符串长度
2. 修改字符串时，不用频繁分配内存。redis字符串使用一个buf数组保存数据，当有足够空间保存新数据时，可以不需要重新分配内存



缺点

1. 虽然redis字符串也会在最后插入\0，但是也只能使用部分C字符串的标准方法
2. 占用比较多内存。buf可能会有没有使用的内存空间

### RAW

raw类型编码就是数据类型为字符串，数据对象通过指针指向字符串对象

![img](/开源框架/redis/.assert/redis数据结构原理/1120165-20180530075617307-666390009.png)



### EMBSTR

embstr编码与raw编码本质是一样的，只是数据对象和字符串对象的内存是连续的。它的优点是：

1. 可以减少内存分配和释放的次数

![img](/开源框架/redis/.assert/redis数据结构原理/1120165-20180530075632624-696981013.png)



### ZipList

ziplist是一种存储多个字符串数据的数据结构，ziplist可以减少内存空间

```
<zlbytes> <zltail> <zllen> <entry> <entry> ... <entry> <zlend>
```

如果没有特别指出，ziplist所有整数采用小端存储

1. zlbytes：32位无符整数，占4个字节，表示整个ziplist的长度，包括zlbytes自己的4个字节。
2. zltail：32位无符整数，占用4个字节，表示最后一个entry的偏移量。
3. zllen：16位无符整数，占用2个字节，表示entry的个数。当entry个数小于2^16-2时，zllen值entry的个数，当entry个数大于等于2^16-1，zllen的值为2^16-1，此时，如果需要知道entry的个数，需要遍历整个ziplist
4. entry：每个元素
5. zlend：一个字节，表示ziplist的结尾，值为FF



entry的结构

```
<prevlen> <encoding> <entry-data>
# 当数据为0到12的整数时，数值放在encoding中表示，没有entry-data部分
<prevlen> <encoding> 

```

1. prevlen: 前一个entry的长度

   1. 当前一个entry长度小于253时，prevlen使用一个字节表示8位无符整数`<prevlen from 0 to 253> <encoding> <entry>`
   2. 当前一个entry长度大于等于254时，prevlen使用5个字节表示，第一个字节为FE，其他4个字节为32为无符整数,`0xFE <4 bytes unsigned little endian prevlen> <encoding> <entry>`

2. encoding：表示数据的编码和长度信息。编码分为整数和字符串两种。

   | encoding编码                                                 | 数据类型 | 数据长度                                                     |
   | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
   | \|00pppppp\| 1个字节                                         | 字符串   | `pppppp`表示字符串的长度。字符串长度不大于63                 |
   | \|01pppppp\|qqqqqqqq\| 2个字节                               | 字符创   | \|00pppppp\|qqqqqqqq\|表示字符串长度。长度不大于16383。大端数字 |
   | \|10000000\|qqqqqqqq\|rrrrrrrr\|ssssssss\|tttttttt\| 5个字节 | 字符串   | 后四个字节表示字符串长度。长度大于等于16384，小于等于2^32-1。大端数字 |
   | \|11000000\|                                                 | 整数     | 2字节整数                                                    |
   | \|11010000\|                                                 | 整数     | 4字节整数                                                    |
   | \|11100000\|                                                 | 整数     | 8字节整数                                                    |
   | \|11110000\|                                                 | 整数     | 3字节整数                                                    |
   | \|1111xxxx\|                                                 | 整数     | xxxx的编码范围为0001到1101，分别表示0到12（xxxx减1）         |

3. entry-data：存放数据，存放的数据与encoding有关，参考encoding编码



![img](/开源框架/redis/.assert/redis数据结构原理/format,png.png)

![img](/开源框架/redis/.assert/redis数据结构原理/format,png-20220210084733776.png)





### linkedList

已经不使用LinkedList编码

![img](/开源框架/redis/.assert/redis数据结构原理/format,png-20220210084844756.png)



### Skiplist

![img](/开源框架/redis/.assert/redis数据结构原理/format,png-20220210085135146.png)



```c
/* ZSETs use a specialized version of Skiplists */
typedef struct zskiplistNode {
    sds ele;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span; 
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

1. 每个节点保存前一个节点的指针，主要是方便反向遍历。在复制ziplist时，是从tail复制到header，会使用backward属性
2. span保存了指针指向的节点与当前节点的间隔大小，是用于快速查找节点的排名



### quicklist

一个双向链表，其中的元素是一个ziplist，或压缩过的zipList。

![img](/开源框架/redis/.assert/redis数据结构原理/QuickList.png)

```c
typedef struct quicklistNode {
    struct quicklistNode *prev;
    struct quicklistNode *next;
    unsigned char *zl;
    unsigned int sz;             /* ziplist size in bytes */
    unsigned int count : 16;     /* count of items in ziplist */
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    unsigned int recompress : 1; /* was this node previous compressed? */
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;

/* quicklistLZF is a 4+N byte struct holding 'sz' followed by 'compressed'.
 * 'sz' is byte length of 'compressed' field.
 * 'compressed' is LZF data with total (compressed) length 'sz'
 * NOTE: uncompressed length is stored in quicklistNode->sz.
 * When quicklistNode->zl is compressed, node->zl points to a quicklistLZF */
typedef struct quicklistLZF {
    unsigned int sz; /* LZF size in bytes*/
    char compressed[];
} quicklistLZF;
```





### intset

instset里面使用8位整数数组保存数据。整数的类型有16位，32位和64位，redis会自动增大整数的位数。在实际保存中，intset中的整数是从小到大保存，可以采用二分查找确定元素是否存在。





## 数据类型和底层数据编码的对应关系

每一种数据类型都有两种以上数据编码

### string

| 数据编码              | 使用场景                                             |
| --------------------- | ---------------------------------------------------- |
| REDIS_ENCODING_INT    | 当数据为整数，且范围在64位范围内                     |
| REDIS_ENCODING_EMBSTR | 第一次创建，数据不为整数，且长度小于或等于44个字节时 |
| REDIS_ENCODING_RAW    | 其他场景                                             |



### list

list使用REDIS_ENCODING_QUICKLIST编码保存。通过以下参数控制：

1. list-compress-depth：表示压缩的深度。0表示全部不压缩，N表示前N个和后N个ziplist节点不压缩。
2. list-max-ziplist-size：表示一个ziplist的最大大小，超过这个大小，ziplist将会拆分成两个节点或者创建一个新节点





```conf


# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0


```



### hash

hash使用REDIS_ENCODING_ZIPLIST和REDIS_ENCODING_HASH编码保存，使用以下参数控制：

1. hash-max-ziplist-entries 512。ziplist的个数不超过512个
2. hash-max-ziplist-value 64。ziplist每个元素的大小不超过64个字节



```
# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```



### set

使用REDIS_ENCODING_INTSET和REDIS_ENCODING_HASH编码保存，使用以下参数控制：

1. Set-max-intset-entries：表示intset保存的最大元素个数



```
# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512
```



### zset

zset使用skiplist和hashtable保存

1. Zset-max-ziplist-entries：ziplist保存的最大元素个数
2. zset-max-ziplist-value：ziplist元素最大长度

```

# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```





## 总结

1. redis有5中数据类型，底层使用8中编码实现。
2. 每种redis数据类型会对应多种数据结构。ziplist数据结构是使用最多的一种（list，hash）
3. string类型由（int，raw，embed）三种编码实现
4. list由quicklist实现，quicklist其实是一个双向链表，使用ziplist作为元素。
5. hash使用ziplist和hash表实现
6. set使用intset和hash表
7. zset使用ziplist和（skiplist+hash）表实现





## 疑问

1. embstr在修改时，redisObject会不会重新分配？
2. zset-max-ziplist-value表示的是整个entry的长度，还是只是字符串的长度？字符串长度

