# Redis对象

- Redis并没有直接使用上述数据结构，而是基于上述的数据结构，构造了一套对象系统，包含
	- 字符串对象
	- 列表对象
	- 哈希对象
	- 集合对象
	- 有序集合

这样就可以根据不同场景使用不同的数据结构，从而达到使用效率的优化

- Redis对象系统使用了引用计数，在某个对象没有引用的时候，这个对象所占的内存就会自动释放掉。
- 有了引用计数法，Redis还实现了对象共享机制，在适当的条件下，多个数据库会共享一个对象来节约内存。
- redisObject自身还带有一个记录访问时间的变量，在服务器使用了最大内存（maxmemory）的时候，如果这个key长时间没有被使用，并且达到了最大内存，这个key根据淘汰策略，就有可能被优先删除掉

- redisObject的定义

```c
typedef struct redisObject {
	//类型
    unsigned type:4;
    //编码
    unsigned encoding:4;
    //LRU或LFU的记录
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
	//引用次数
    int refcount;
    //底层数据结构的指针
    void *ptr;
} robj;
```

## 类型

|类型常量|名称|type值|
|:-:|:-:|:-:|
|REDIS_STRING|字符串对象|string|
|REDIS_LIST|列表对象|list|
|REDIS_HASH|哈希对象|hash|
|REDIS_SET|集合对象|set|
|REDIS_ZSET|有序集合对象|zset|

## 编码

|编码常量|类型|对象|
|:-:|:-:|:-:|
|REDIS_ENCODING_INT|REDIS_STRING|整数值实现字符串对象|
|REDIS_ENCODING_EMBSTR|REDIS_STRING|embstr编码的简单SDS|
|REDIS_ENCODING_RAW|REDIS_STRING|普通SDS|
