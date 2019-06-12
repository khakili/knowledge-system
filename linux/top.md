# top -M

> top -M 以可读的方式展示单位

```linux
top - 15:04:26 up 94 days, 22:47,  1 user,  load average: 0.25, 0.15, 0.14
Tasks: 164 total,   1 running, 163 sleeping,   0 stopped,   0 zombie
Cpu0  :  0.0%us,  0.1%sy,  0.0%ni, 99.7%id,  0.2%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu1  :  0.0%us,  0.0%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu2  :  0.0%us,  0.0%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu3  :  0.0%us,  0.0%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:  7872.184M total, 4829.207M used, 3042.977M free,  357.102M buffers
Swap:    0.000k total,    0.000k used,    0.000k free, 1307.512M cached

PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND 
```

- 15:04:26 当前的系统时间
- 94 days, 22:47 已经运行了94天22小时47分钟
- 1 user 当前有一个用户正在登陆
- load average 负载情况 三个值分别为过去1分钟，5分钟，15分钟的负载情况。
**load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。**
- Tasks: 164 total 一共有164个进程
-  1 running, 163 sleeping,   0 stopped,   0 zombie 1个在运行中（running），163在休眠（sleeping），0个停止的（stopped）0个僵尸进程（zombie）
- Cpu0-Cpu3 4核CPU的负载情况（top -M进来会显示CPUS[所有用户进程占用整个cpu的平均值]，按1可以显示每个cpu的负载情况）
- 0.0%us 用户空间占用CPU的百分比。
- 0.1%sy 内核空间占用CPU的百分比。
- 0.0%ni 改变过优先级的进程占用CPU的百分比
- 99.7%id 空闲CPU百分比
- 0.2%wa IO等待占用CPU的百分比
- 0.0%hi 硬中断（Hardware IRQ）占用CPU的百分比
- 0.0%si 软中断（Software Interrupts）占用CPU的百分比
- 0.0%st stole time 的缩写，该项指标只对虚拟机有效，表示分配给当前虚拟机的 CPU 时间之中，被同一台物理机上的其他虚拟机偷走的时间百分比（表明你的系统花了百分之多少等待得到真正的cpu资源）
- Mem 
	- 第1段：物理内存总量，例如： 7872.184M total, 
	- 第2段：使用的物理内存总量，例如：4829.207M used,
	- 第3段：空闲内存总量，例如：Mem: 3042.977M free,
	- 第4段：用作内核缓存的内存量，例如：357.102M buffers
- Swap
	- 第1段：交换区总量，例如： 0.000k total,
	- 第2段：使用的交换区总量，例如： 0.000k used,
	- 第3段：空闲交换区总量，例如：0.000k free,
	- 第4段：缓冲的交换区总量，1307.512M cached
- PID 进程号
- USER 用户
- PR (Priority) 优先级
- NI (Nice value) nice值。负值表示高优先级，正值表示低优先级
- VIRT  (Virtual Image (kb)) 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES (Resident size (kb)) 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR  (Shared Mem size (kb)) 共享内存大小，单位kb
- S  (Process Status) 进程状态。D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程
- %CPU  (CPU usage) 上次更新到现在的CPU时间占用百分比
- %MEM (Memory usage (RES)) 进程使用的物理内存百分比
- TIME+  (CPU Time, hundredths) 进程使用的CPU时间总计，单位1/100秒
- COMMAND  (Command name/line) 命令名/命令行

