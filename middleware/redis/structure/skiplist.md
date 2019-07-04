# 跳跃表

- 跳跃表是有序集合的底层实现之一，在Redis中只有两个地方使用了跳跃表，另一个是集群节点的内部数据结构
- 跳跃表（skiplist）支持平均O(logN)，最坏O(N)查找


- 跳跃表结构：主要由zskiplist和zskiplistNode组成
	- zskiplist用于保存跳跃表信息（头结点、尾节点，长度） 
	- zskiplistNode存储节点信息（实际存储需要保存的item）
	- 节点的层高都是1-32之间的随机数
	- 在同一个跳跃表中，多个节点可以包含相同的分值，但每个节点的成员对象必须是唯一的。
	- 跳跃表中的节点按照分值大小进行排序，当分值相同时，节点按照成员对象的大小进行排序。

```c
//src/server.h
typedef struct zskiplistNode {
	//节点存储的sds
    sds ele;
    //分值
    double score;
    //后一个节点
    struct zskiplistNode *backward;
    struct zskiplistLevel {
    	 //前一个节点的指针
        struct zskiplistNode *forward;
        //跨度
        unsigned long span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
	//头节点，尾节点
    struct zskiplistNode *header, *tail;
    //节点个数
    unsigned long length;
    //最高层数
    int level;
} zskiplist;
typedef struct zset {
	//字典
    dict *dict;
    //跳跃表
    zskiplist *zsl;
} zset;


```

