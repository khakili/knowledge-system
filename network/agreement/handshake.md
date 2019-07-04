# TCP的三次握手与四次挥手


## TCP的三次握手流程
![](images/handshake.png '握手流程')

（1）客户端发送SYN=1，seq=x

（2）服务端响应SYN=1，ACK=1，seq=y，ack=x+1

（3）客户端发送ACK=1，seq=x+1，ack=y+1

（4）连接建立成功，开始传输数据报文

- 为什么要三次握手

	**主要防止已经失效的连接请求报文突然又传送到了服务器，从而产生错误。**

第三次握手主要是为了前一次的服务器响应在网络不佳的情况下，导致客户端没有收到服务端的回应，重发了握手请求。如果是两次握手，在网络不佳的情况下有可能会浪费一次网络通信。

## TCP的四次挥手流程

![](images/wave.jpg '挥手流程')


（1）client向server发出释放tcp连接的请求（FIN=1，seq=u），server进入close-wait状态

（2）server响应client（ACK=1，seq=v，ack=u+1）

（3）server再次响应client（FIN=1，ACK=1，seq=w，ack=u+1），server进入last-ack状态

（4）client向server发送（ACK=1，seq=u+1，ack=w+1），server只要接到client的这个请求，立即关闭连接

（5）client等待2msl关闭连接

- 为什么客户端最后要等2msl才关闭连接
	- 什么是msl
		
		最长报文段寿命，它是任何报文在网络上存在的最长的最长时间，超过这个时间报文将被丢弃

	- 为什么要等2msl
		
		等待2msl是为了在网络不佳的时候，server没有收到client最后发的ACK报文，server会重发FIN-ACK报文，如果客户端不等待2msl直接关闭连接的话，server就无法进到closed状态了