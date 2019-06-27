# BIO、NIO和AIO

## BIO

- 一种**阻塞同步**的通信模式
- **BIO模型中通过Socket和ServerSocket完成套接字通道的实现。阻塞，同步，建立连接耗时。**
- 优点
	- 模式简单，使用方便
- 缺点
	- 并发处理能力低，通信耗时，依赖网速

## NIO
- 一种**非阻塞同步**的通信模式
- **NIO模型中通过SocketChannel和ServerSocketChannel完成套接字通道的实现。非阻塞/阻塞，同步，避免TCP建立连接使用三次握手带来的开销**

## AIO（全场最佳）
- 一种**非阻塞异步**的通信模式
- **AIO模型中通过AsynchronousSocketChannel和AsynchronousServerSocketChannel完成套接字通道的实现。非阻塞，异步。**

## 小结

||BIO|NIO|AIO|
|:--:|:--:|:--:|:--:|
|是否阻塞|阻塞|非阻塞|非阻塞|
|同步/异步|同步|同步|异步|
|性能|低|一般|高|
|线程比|1:1|N:1|N:0|
|吞吐量|低|高|高|
