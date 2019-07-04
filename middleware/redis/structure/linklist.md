# 链表

```c
//src/adlist.h
//链表节点定义
typedef struct listNode {
	//上一个节点
    struct listNode *prev;
    //下一个节点
    struct listNode *next;
    //当前节点值的指针
    void *value;
} listNode;
//迭代器
typedef struct listIter {
	//下一个节点
    listNode *next;
    //方向
    int direction;
} listIter;

//链表定义
typedef struct list {
	//头节点
    listNode *head;
    //尾节点
    listNode *tail;
    //复制节点函数
    void *(*dup)(void *ptr);
    //释放节点函数
    void (*free)(void *ptr);
    //节点比较函数
    int (*match)(void *ptr, void *key);
    //链表长度
    unsigned long len;
} list;

```

````c
//创建一个空链表
list *listCreate(void)
{
    struct list *list;

    if ((list = zmalloc(sizeof(*list))) == NULL)
        return NULL;
    //（5）
    list->head = list->tail = NULL;
    list->len = 0;
    list->dup = NULL;
    list->free = NULL;
    list->match = NULL;
    return list;
}

```

- 总结
	1.	由于结构特性，使list修改很快（时间复杂度O(1))，查找复杂度O(n)
	2.	获取链表长度时间复杂度O(1)
	3. 由于链表中节点不直接存储数据，所以链表可以保存不同类型的值
	4. redis链表是双向链表
	5. 因为链表表头节点的前置节点和表尾节点的后置节点都指向null, 所以Redis的链表实现是无环链表。
	