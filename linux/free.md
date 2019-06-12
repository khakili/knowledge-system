# free -m

```linux
$ free -m
             total       used       free     shared    buffers     cached
Mem:          7872       4810       3061          0        357       1301
-/+ buffers/cache:       3152       4720
Swap:            0          0          0
```

- 从系统层面分析

	Mem:内存的使用情况总览表。

	totel:机器总的物理内存 单位为：M

	used：用掉的内存。

	free:空闲的物理内存。

> 物理内存(totel)=系统看到的用掉的内存(used)+系统看到空闲的内存(free)
 
- 从程序的角度分析

	shared:多个进程共享的内存总和，当前废弃不用。

	buffers：缓存内存数。

	cached:  缓存内存数。

> 程序预留的内存=buffers+cached

**buffers/cached可以分为两部分 + buffers/cached 和 - buffers/cached。**

**总的物理内存=| + buffers/cached |+| - buffers/cached |；**

> \- buffers/cached：程序角度上看已经使用的内存数，这才是程序实实在在用掉的内存数。
> 
> \+ buffers/cached：程序角度上看未使用、可用的内存数。