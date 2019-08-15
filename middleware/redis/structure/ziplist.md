# 压缩列表

- 压缩列表（ziplist）是列表和哈希表的底层实现之一
	- 在列表里包含少量元素，且列表项是小整数或者段字符串的时候会用压缩列表作为底层实现
	- 哈希表同理
- 压缩列表是为了节约内存而开发的顺序性数据结构
- 压缩列表可以包含多个个节点，每个节点可以保存一个字节数组或整数值
- 添加新节点或删除节点可能会引发连锁更新操作，但是几率很低
- 连锁更新

每个节点的previous_entry_length 属性都记录了前一个节点的长度： 

1. 如果前一节点的长度小于254 字节，那么previ ous_ entry_length 属性需要用 1字节长的空间来保存这个长度值。 
2. 如果前一节点的长度大于等于254 字节，那么previous entry length 属性需要用5 字节长的空间来保存这个长度值。 

	如果我们将一个长度大于等于 254 字节的新节点 new 设置为压缩列表的表头节点，那么麻烦的事情来了，由于previous entry length大小不够用(1->5B)，后面所有的节点可能都要重新分配内存大小。因为连锁更新在最坏情况下需要对压缩列表执行 N 次空间重分配操作， 而每次空间重分配的最坏复杂度为 O(N) ， 所以连锁更新的最坏复杂度为 O(N^2) 。

	但是呢，尽管连锁更新的复杂度较高，但它真正造成性能问题的几率是很低的。 
	1. 首先，压缩列表里要恰好有多个连续的、长度介于250 字节至253 宇节之间的节点，连锁更新才有可能被引发，在实际中，这种情况并不多见； 
	2. 其次，即使出现连锁更新，但只要被更新的节点数量不多，就不会对性能造成任何影响：比如说，对三五个节点进行连锁更新是绝对不会影响性能的； 


```c
typedef struct zlentry {
    unsigned int prevrawlensize; /* Bytes used to encode the previous entry len*/
    unsigned int prevrawlen;     /* Previous entry len. */
    unsigned int lensize;        /* Bytes used to encode this entry type/len.
                                    For example strings have a 1, 2 or 5 bytes
                                    header. Integers always use a single byte.*/
    unsigned int len;            /* Bytes used to represent the actual entry.
                                    For strings this is just the string length
                                    while for integers it is 1, 2, 3, 4, 8 or
                                    0 (for 4 bit immediate) depending on the
                                    number range. */
    unsigned int headersize;     /* prevrawlensize + lensize. */
    unsigned char encoding;      /* Set to ZIP_STR_* or ZIP_INT_* depending on
                                    the entry encoding. However for 4 bits
                                    immediate integers this can assume a range
                                    of values and must be range-checked. */
    unsigned char *p;            /* Pointer to the very start of the entry, that
                                    is, this points to prev-entry-len field. */
} zlentry;
```