# pidstat

	pidstat是sysstat工具的一个命令，用于监控全部或指定进程的cpu、内存、线程、设备IO等系统资源的占用情况。
	
	pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
	
## pidstat -u -p ALL

|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| PID  | %usr| %system | %guest|    %CPU  | CPU  |Command|

