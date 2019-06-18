# vmstat

> vmstat(Virtual Memory Statistics 虚拟内存统计) 命令用来显示Linux系统虚拟内存状态，也可以报告关于进程、内存、I/O等系统整体运行状态。

<table>
	<tr>
		<td>类别</td>
		<td>项目</td>
		<td>含义</td>
		<td>说明</td>
	</tr>
	<tr>
		<td rowspan=2>Procs（进程）</td>
		<td>r</td>
		<td>等待执行的任务数</td>
		<td>展示了正在执行和等待cpu资源的任务个数。当这个值超过了cpu个数，就会出现cpu瓶颈。</td>
	</tr>
	<tr>
		<td>b</td>
		<td>等待IO的进程数量</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan=6>Memory(内存)</td>
		<td>swpd</td>
		<td>正在使用虚拟的内存大小，单位k</td>
		<td></td>
	</tr>
	<tr>
		<td>free</td>
		<td>空闲内存大小</td>
		<td></td>
	</tr>
	<tr>
		<td>buff</td>
		<td>已用的buff大小，对块设备的读写进行缓冲</td>
		<td></td>
	</tr>
	<tr>
		<td>cache</td>
		<td>已用的cache大小，文件系统的cache</td>
		<td></td>
	</tr>
	<tr>
		<td>inact</td>
		<td>非活跃内存大小，即被标明可回收的内存，区别于free和active</td>
		<td>添加-a时显示</td>
	</tr>
	<tr>
		<td>active</td>
		<td>活跃的内存大小</td>
		<td>添加-a时显示</td>
	</tr>
	<tr>
		<td rowspan=2>Swap</td>
		<td>si</td>
		<td>每秒从交换区写入内存的大小（单位：kb/s）</td>
		<td></td>
	</tr>
	<tr>
		<td>so</td>
		<td>每秒从内存写到交换区的大小</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan=2>IO</td>
		<td>bi</td>
		<td>每秒读取的块数（读磁盘）</td>
		<td></td>
	</tr>
	<tr>
		<td>bo</td>
		<td>每秒写入的块数（写磁盘）</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan=2>System</td>
		<td>in</td>
		<td>每秒中断数，包括时钟中断</td>
		<td rowspan=2>这两个值越大，会看到由内核消耗的cpu时间会越多</td>
	</tr>
	<tr>
		<td>cs</td>
		<td>每秒上下文切换数</td>
	</tr>
	<tr>
		<td rowspan=4>CPU（以百分比表示）</td>
		<td>us</td>
		<td>用户进程执行消耗cpu时间(user time)</td>
		<td>us的值比较高时，说明用户进程消耗的cpu时间多，但是如果长期超过50%的使用，那么我们就该考虑优化程序算法或其他措施了</td>
	</tr>	
	<tr>
		<td>sy</td>
		<td>每秒中断数，包括时钟中断</td>
		<td></td>
	</tr>	
	<tr>
		<td>id</td>
		<td>空闲时间(包括IO等待时间)</td>
		<td></td>
	</tr>
	<tr>
		<td>wa</td>
		<td>等待IO时间</td>
		<td>Wa过高时，说明io等待比较严重，这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。
		</td>
	</tr>
</table>

- 常见问题

	- 如果r经常大于4，且id经常少于40，表示cpu的负荷很重。
	- 如果pi，po长期不等于0，表示内存不足。
	- 如果disk经常不等于0，且在b中的队列大于3，表示io性能不好。
		- 如果在processes中运行的序列(process r)是连续的大于在系统中的CPU的个数表示系统现在运行比较慢,有多数的进程等待CPU。
		- 如果r的输出数大于系统中可用CPU个数的4倍的话,则系统面临着CPU短缺的问题,或者是CPU的速率过低,系统中有多数的进程在等待CPU,造成系统中进程运行过慢。
		- 如果空闲时间(cpu id)持续为0并且系统时间(cpu sy)是用户时间的两倍(cpu us)系统则面临着CPU资源的短缺。

- 解决办法

	- 当发生以上问题的时候请先调整应用程序对CPU的占用情况.使得应用程序能够更有效的使用CPU.同时可以考虑增加更多的CPU.  关于CPU的使用情况还可以结合mpstat,  ps aux top  prstat –a等等一些相应的命令来综合考虑关于具体的CPU的使用情况,和那些进程在占用大量的CPU时间.一般情况下，应用程序的问题会比较大一些.比如一些sql语句不合理等等都会造成这样的现象.

	- 内存问题现象:

		- 内存的瓶颈是由scan rate (sr)来决定的.scan rate是通过每秒的始终算法来进行页扫描的.如果scan rate(sr)连续的大于每秒200页则表示可能存在内存缺陷.同样的如果page项中的pi和po这两栏表示每秒页面的调入的页数和每秒调出的页数.如果该值经常为非零值,也有可能存在内存的瓶颈,当然,如果个别的时候不为0的话,属于正常的页面调度这个是虚拟内存的主要原理.

	- 解决办法: 
		- 1.调节applications & servers使得对内存和cache的使用更加有效.

		- 2.增加系统的内存.

		- 3. Implement priority paging in s in pre solaris 8 versions by adding line "set priority paging=1" in /etc/system. Remove this line if upgrading from Solaris 7 to 8 & retaining old /etc/system file.

> 关于内存的使用情况还可以结ps aux top  prstat –a等等一些相应的命令来综合考虑关于具体的内存的使用情况,和那些进程在占用大量的内存.一般情况下，如果内存的占用率比较高,但是,CPU的占用很低的时候,可以考虑是有很多的应用程序占用了内存没有释放,但是,并没有占用CPU时间,可以考虑应用程序,对于未占用CPU时间和一些后台的程序,释放内存的占用。
