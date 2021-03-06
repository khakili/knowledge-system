# TCP滑动窗口机制

## TCP窗口机制

- 固定窗口
- 滑动窗口

## 什么是滑动窗口

- 滑动窗口是固定窗口的一个特例，滑动窗口将固定窗口又细分为多个小的窗口。
- 滑动窗口最大的优势在于不需要对每个数据包进行确认，而是最后累计确认窗口内所有数据包的结果。

- 滑动窗口的原则
	- 对发送的数据做分段处理，给每段数据编号
	- 滑动窗口的大小是动态的，每次由接收端连同确认码一同返回
	- 接收端对接收到的连续数据进行确认，若出现乱序传输，则将先到达的编号较大的数据缓存，并等待数据到达

- 滑动窗口发送方的原则
	- 发送且已经确认窗口：这个窗口内的数据表示已经发送成功并已经被确认的数据。
	- 发送但未确认窗口 : 这个窗口内的数据表示已经发送成功但并没有被确认的数据。
	- 未发送但允许发送的窗口 : 这个窗口内的数据表示尚未发送但是允许被发送的数据。
	- 不允许发送窗口 : 这个窗口内的数据属于接收端不允许发送的数据，因为这些数据已经超出了发送端所接收的范围

- 滑动窗口接收方的原则
	- 接收但未处理窗口 : 这个窗口内数据属于接收了但是还没有被上层的应用程序处理，被缓存在窗口内
	- 接收但未确认窗口 : 这个窗口内数据属于接收了但是还没有来得及确认的数据
	- 有空位但还未接收数据窗口 : 表示可以接收数据，但是数据尚未发送过来的窗口

## 滑动窗口示例
- 假设有33-39这些数据待发送，TCP协议将它们分成四个报文段分别为seg1 33-34，seg2 35-36，seg3 37，seg4 38-39这四个段，并依次发给接收端。
- 假设接收端之接收到了seg1，seg2，seg4。则此时接收端回复一个ACK表示33-36已经成功接收，由于seg3尚未到达，所以将seg4缓存起来。
- 发送端接收到ACK后将33-36这部分数据从“发送但未确认窗口”移到“发送且已确认窗口”，即将“发送但未确认窗口”的左边界右移4个单位。
- 假设接收端发送给窗口大小不变，则“发送但未确认窗口”的右边界继续向右移动4个单位，只要不越进“不允许发送窗口”即可。
- 对于seg3，若一段时间后，接收端接收到了seg3，则发送确认码表示seg3和seg4接收成功。若没有收到seg3，则丢弃seg4。对应的，如果发送端一段时间没有接收到seg3的确认码，则重传seg3。