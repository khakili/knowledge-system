# 字典

> Redis的Hash，就是在数组+链表的基础上，进行了一些rehash优化等。

```c
//src/dict.h

//hash表
typedef struct dictht {
	//哈希数组
    dictEntry **table;
    //哈希表大小（哈希数组长度）
    unsigned long size;
    //哈希表掩码（总是为size-1），用于计算索引
    unsigned long sizemask;
    //哈希表当前大小（当前长度）
    unsigned long used;
} dictht;

//hash节点
typedef struct dictEntry {
	//键
    void *key;
    //值
    union {
    	//值指针
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    //下一个节点，单链表
    struct dictEntry *next;
} dictEntry;

//字典
typedef struct dict {	
    dictType *type;
    //私有数据
    void *privdata;
    //ht[0]存放哈希表，ht[1]存放rehash用的哈希表
    dictht ht[2];
    //渐进式rehash次数
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    //迭代器计数标识
    unsigned long iterators; /* number of iterators currently running */
} dict;

```

- 字典存放k-v的过程
	1. 计算key的哈希值
	2. 计算key在table数组中的位置（第一步的结果&sizemask）
	3. 放入数组（ht[0]）

- 如何解决hash冲突

	使用链地址法解决hash冲突，会将冲突的entry放到链表的第一位（不会像java将链表转换为红黑树）

- rehash过程与渐进式rehash

	- 渐进式rehash
		**因为rehash操作需要将ht[0]的所有entry rehash到ht[1]，ht[0]可能会非常大，所以需要渐进式rehash**
		- 渐进式rehash过程
			1. 为ht[1]分配与ht[0]一样的空间
			2. rehashidx初始化为0，标记rehash开始
			3. 在rehash开始以后进行的增删改查操作，除了进行对应的操作，还要将ht[0]的对应entry rehash到ht[1]				上，当rehash完后后rehashidx+1（新增entry会直接保存到ht[1]，减少rehash次数）
			4. 在ht[0]的所有entry都移动到了ht[1]上后rehashidx设置为-1，标识rehash完成
		
	- rehash过程
		1. 初始化ht[1]到ht[0]大小的2的n次幂
		2. 执行渐进式rehash
		3. 在所有entry 都进行完rehash后释放ht[0]，将ht[1]改为ht[0]，并创建ht[1]

- 什么时候进行rehash操作

	1. 在没有执行BGSAVE或者BGREWRITEAOF并且负载因子大于1
	2. 正在执行这两个命令且负载因子大于5
	3. 负载因子小于0.1时进行收缩操作（将ht[1]=ht[0].used的2^n）

	> 负载因子=ht[0].used(已用大小)/ht[0].size(哈希表大小)
	
- 为什么要根据两个BG命令判断使用5还是1
	
	这两个BG命令需要使用子进程，当前大多数操作系统都是采用copy-on-write来优化子进程的使用效率，所以在执行这两个命令的时候提高了rehash的条件，减少内存读写，节约内存

