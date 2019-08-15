# redis数据过期及淘汰策略

## 数据过期
	给key设置过期时间
	expire key time(以秒为单位)
	字符串独有的方式
	setex(String key, int seconds, String value)
如果设置了过期时间，之后又想让缓存永不过期，使用**persist key**（很少有这种操作）

**如果没有设置时间，那缓存就是永不过期**

### 数据过期策略
- 被动删除（惰性删除）
	- 当读/写一个已经过期的key时，会触发惰性删除策略，直接删除掉这个过期key。 
	- 惰性删除的优缺点
		1. 对CPU友好，不会检查全量的key
		2. 	对内存不友好，如果有key是过期的但是没有请求访问他，这个key就会一直占用内存，变相是一种内存泄露
- 主动删除
	-  基于src/server.c/serverCron方法
	-  优缺点
		-  优点：通过限制删除操作的时长和频率，来减少删除操作对CPU时间的占用–处理"定时删除"的缺点，定期删除过期key–处理"惰性删除"的缺点。
		-  缺点：在内存友好方面，不如"定时删除"，在CPU时间友好方面，不如"惰性删除"。
- 达到maxmemory，触发淘汰策略

## 淘汰策略
-	volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用 的数据淘汰
-	volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数 据淘汰
-	volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据 淘汰
-	allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
-	allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
-	no-enviction（驱逐）：（默认）禁止驱逐数据
