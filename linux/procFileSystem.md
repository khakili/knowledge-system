# proc文件系统

> /proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。
>
> 它以文件系统的方式为内核与进程提供通信的接口。
> 
> 用户和应用程序可以通过/proc得到系统的信息，并可以改变内核的某些参数。
> 
> 由于系统的信息，如进程，是动态改变的，
> 
> 所以用户或应用程序读取/proc目录中的文件时，
> 
> proc文件系统是动态从系统内核读出所需信息并提交的。

## /proc/cpuinfo

/proc/cpuinfo文件
         该文件中存放了有关 cpu的相关信息(型号，缓存大小等)。

```linux
[zhengangen@buick ~]$ cat /proc/cpuinfo

processor       : 0

vendor_id       : GenuineIntel

cpu family      : 15

model           : 4

model name      : Intel(R)Xeon(TM) CPU 3.00GHz

stepping        : 10

cpu MHz         : 3001.177

cache size      : 2048 KB

physical id     : 0

siblings        : 2

core id         : 0

cpu cores       : 1

fdiv_bug        : no

hlt_bug         : no

f00f_bug        : no

coma_bug        : no

fpu             : yes

fpu_exception   : yes

cpuid level     : 5

wp              : yes

flags           : fpu vme de psetsc msr pae mce cx8 apic mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsrsse sse2 ss ht tm pbe lm pni monitor ds_cpl cid xtpr

bogomips        : 6004.52


//说明：以下只解释对我们计算Cpu使用率有用的相关参数。
 //参数					解释
// processor (0)   cpu的一个物理标识
//结论1：可以通过该文件根据processor出现的次数统计cpu的逻辑个数(包括多核、超线程)。

 //总核数 = 物理CPU个数 X 每颗物理CPU的核数 
 //总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

 //查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

 //查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

 //查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

## /proc/pid

> /proc目录中有一些以数字命名的目录，它们是进程目录。
>
> 系统中当前运行的每一个进程在/proc下都对应一个以进程号为目录名的目录/proc/pid，它们是读取进程信息的接口。
>
> 此外，在Linux2.6.0-test6以上的版本中/proc/pid目录中有一个task目录，/proc/pid/task目录中也有一些以该进程所拥有的线程的线程号命名的目录/proc/pid/task/tid，它们是读取线程信息的接口。


## /proc/stat

> /proc/stat文件
>
> 该文件包含了所有CPU活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。
> 不同内核版本中该文件的格式可能不大一致，以下通过实例来说明数据该文件中各字段的含义。

**实例数据：2.6.24-24版本上的**

```linux
cat /proc/stat

cpu  38082 627 27594 89390812256 581 895 0 0

cpu022880 472 16855 430287 10617 576 661 0 0

cpu115202 154 10739 463620 1639 4 234 0 0

intr120053 222 2686 0 1 1 0 5 0 3 0 0 0 47302 0 0 34194 29775 0 5019 845 0 0 0 0 00 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 00 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 00 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 00 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 00 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0

ctxt1434984

btime1252028243

processes8113

procs_running1

procs_blocked0
```

第一行的数值表示的是CPU总的使用情况，所以我们只要用第一行的数字计算就可以了。下面解析第一行各数值的含义：

- 参数解析（单位：jiffies）

>(jiffies是内核中的一个全局变量，用来记录自系统启动一来产生的节拍数，在linux中，一个节拍大致可理解为操作系统进程调度的最小时间片，不同linux内核可能值有不同，通常在1ms到10ms之间)

	user (38082)    从系统启动开始累计到当前时刻，处于用户态的运行时间，不包含 nice值为负进程。

	nice (627)      从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间

	system (27594)  从系统启动开始累计到当前时刻，处于核心态的运行时间

	idle (893908)   从系统启动开始累计到当前时刻，除IO等待时间以外的其它等待时间iowait (12256) 从系统启动开始累计到当前时刻，IO等待时间(since 2.5.41)

	irq (581)           从系统启动开始累计到当前时刻，硬中断时间(since 2.6.0-test4)

	softirq (895)	从系统启动开始累计到当前时刻，软中断时间(since2.6.0-test4)stealstolen(0)                   which is the time spent in otheroperating systems when running in a virtualized environment(since 2.6.11)

	guest(0)	whichis the time spent running a virtual CPU  for guest operating systems under the control ofthe Linux kernel(since 2.6.24)
	
## /proc/&lt;pid&gt;/stat
- /proc/&lt;pid&gt;/stat文件                                          

>该文件包含了某一进程所有的活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。

**以下通过实例数据来说明该文件中各字段的含义。**

```linux
cat/proc/6873/stat
6873 (a.out) R 6723 6873 6723 34819 6873 8388608 77 0 0 0 41958 31 0 0 25 0 3 05882654 1409024 56 4294967295 134512640 134513720 3215579040 0 2097798 0 0 0 00 0 0 17 0 0 0
```
说明：以下只解释对我们计算Cpu使用率有用相关参数

	pid=6873                            进程号

	utime=1587                       该任务在用户态运行的时间，单位为jiffies

	stime=41958                      该任务在核心态运行的时间，单位为jiffies

	cutime=0                         所有已死线程在用户态运行的时间，单位为jiffies

	cstime=0                         所有已死在核心态运行的时间，单位为jiffies
	
## 软连接与硬链接

- 软链接：ln -s 源文件 目标文件 （可以理解成快捷方式。它和windows下的快捷方式的作用是一样的。 ）
- 硬链接：ln 源文件 目标文件 （等于cp -p 加 同步更新。）

> 源文件：即你要对谁建立链接

**i节点是文件和目录的唯一标识，每个文件和目录必有i节点**

> 理解inode，要从文件储存说起。
文件储存在硬盘上，硬盘的最小存储单位叫做"扇区"（Sector）。每个扇区储存512字节（相当于0.5KB）。
操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个"块"（block）。这种由多个扇区组成的"块"，是文件存取的最小单位。"块"的大小，最常见的是4KB，即连续八个 sector组成一个 block。
文件数据都储存在"块"中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做inode，中文译名为"索引节点"。
每一个文件对应一个inode，硬盘上有多少文件，就有多少个inode。


		          --->  test.hard
	inode(i节点)	--->  test
		          --->  test.soft


	   
**删除源文件后软链接失效。此时在软链接写入数据会生成一个新的源文件, 硬链接正常,生成新的源文件也不会对硬链接所在的文件有影响**
			
	作用: 
	软链接: 快捷方式, 常用于日志等输出信息跨分区存储.
	硬链接: 拷贝并实时同步, 重要内容的备份.

 