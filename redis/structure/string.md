# 字符串

> redis中的string并不是C中标准的string对象

## 为什么redis不使用c中的string

> redis中的string全名叫做Simple dynamic string（SDS）

[redis官方对SDS的解释](https://redis.io/topics/internals-sds)

- redis中SDS的结构（伪代码）

```c
//src/sds.h
	struct sds{
		//sds的长度
		int len;
		//sds未使用的长度
		int free;
		//实际存储的字符串
		char[] buf;
	
	}
```

- redis的SDS的优点

	1. 操作SDS的时间复杂度比原生string低很多
		1. 获取长度时间复杂度为O(1)
		2. 创建和追加的时间复杂度为O(n)
		3. 删除的时间复杂度为O(1) 
	2. SDS是二进制安全的，可以存储字符流
	3. 不会出现内存溢出和泄露
	4. 减少内存分配的操作，提高性能
