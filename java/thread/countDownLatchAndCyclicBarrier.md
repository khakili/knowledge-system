# CountDownLatch和CyclicBarrier：如何让多线程步调一致

前几天老板突然过来说，对账系统最近越来越慢，能不能快速优化一下。对账系统的业务是，用户通过商城下单，保存在订单库，之后物流会生成派送单给用户发货，派送单保存在派送单库。为了防止漏派送或者重复派送，对账系统每天还会检验是否存在异常订单。

对账系统的逻辑很简单，可以参照下面的对账系统流程图。目前对账系统的处理逻辑是首先查询订单，然后查询派送单，之后对比订单和派送单，将差异写入差异库。

![](images/countDownLatchAndCyclicBarrier/flowchart.png)

对账系统的代码抽象之后，也很简单，核心代码如下，就是在一个单线程里面循环查询订单、派送单，然后执行对账，最后将差异写入差异库。

```java
while(存在未对账订单){
  // 查询未对账订单
  pos = getPOrders();
  // 查询派送单
  dos = getDOrders();
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
} 
```
## 利用并行优化对账系统

我们要优化这段代码，得先找到对账系统的瓶颈所在。

目前的对账系统，由于订单量和派送单量巨大，所以查询未对账订单getPOrder()和查询派送单getDOrder()先对较慢，那有没有办法快速优化一下呢？目前对账系统是单线程执行的，图形化后是下图这个样子。对于串行化的系统，优化性能首先想到的是**能否利用多线程并行处理**。

![](images/countDownLatchAndCyclicBarrier/origin.png)

所以，这里你应该能够看出来这个对账系统里的瓶颈：查询未对账订单getPOrder()和查询派送单getDOrder()是否可以并行处理呢？显然是可以的，因为这两个操作**并没有先后顺序的依赖**。这两个最耗时的操作并行之后，执行过程如下图所示。对比一下单线程的执行示意图，你会发现同等时间里，并行执行的吞吐量近乎单线程的2倍，优化效果还是相对明显的。

![](images/countDownLatchAndCyclicBarrier/optimize.png)

思路有了，下面我们再来看看如何用代码实现。在下面的代码中，我们创建了两个线程T1和T2，并行执行查询未对账订单getPOrder()和查询派送单getDOrder()这两个操作。在主线程中执行对账操作check()和差异写入save()两个操作。不过需要注意的是：主线程需要等待线程T1和T2执行完才能执行check()和save()这两个操作，为此我们通过调用T1.join()和T2.join()来实现等待，当T1和T2线程退出时，调用T1.join()和T2.join()的主线程就会从阻塞态被唤醒，从而执行之后的check()和save()。

```java
while(存在未对账订单){
  // 查询未对账订单
  Thread T1 = new Thread(()->{
    pos = getPOrders();
  });
  T1.start();
  // 查询派送单
  Thread T2 = new Thread(()->{
    dos = getDOrders();
  });
  T2.start();
  // 等待 T1、T2 结束
  T1.join();
  T2.join();
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
} 

```
## 用CountDownLatch实现线程等待

经过上面的优化，基本可以收工了，但是美中不足的是，while循环里面每次都会创建新的线程，而创建线程可是个耗时的操作。所以最好是创建出来的线程能够循环利用，估计这时你已经想到线程池了，是的，线程池就能解决这个问题。

我们定义一个线程池，固定大小为2，之后在while里面循环利用。一切看上去都很顺利，但是有个问题好像无解，那就是主线程如何知道getPOrder()和getDOrder()这两个操作什么时候执行完。前面主线程通过调用线程T1和线程T2的join()方法来等待线程T1和T2退出，但是在线程池的方案里，线程根本就不会退出，所以join()方法已经失效了。

```java
// 创建 2 个线程的线程池
Executor executor = 
  Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
  });
  
  /* ？？如何实现等待？？*/
  
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}   

```

**CountDownLatch**可以帮我们解决这个问题。在while循环里面，我们首先创建一个CountDownLatch，计数器的初始值等于2，之后在 pos = getPOrder();和 dos = getDOrder();两条语句后面对计数器进行减1操作，这个对计数器减1的操作是通过调用latch.countDown();来实现的。在主线程中，我们通过调用 latch.await()来实现对计数器等于0的等待。

```java
// 创建 2 个线程的线程池
Executor executor = 
  Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 计数器初始化为 2
  CountDownLatch latch = 
    new CountDownLatch(2);
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
    latch.countDown();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
    latch.countDown();
  });
  
  // 等待两个查询操作结束
  latch.await();
  
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}

```

## 进一步优化性能

我们再来看看有没有优化的余地，前面我们将getPOrders() 和 getDOrders()这两个查询操作并行了，但这两个查询操作和对账操作check()、save()之前还是串行的。很显然，这两个查询操作和对账操作也是可以并行的，也就是说，在执行对账操作的时候，可以同时去执行下一轮的查询操作，这个过程可以形象化的标书为下面这幅示意图。

![](images/countDownLatchAndCyclicBarrier/cyclicBarrier.png)

两次查询操作能够和对账操作并行，**对账操作还依赖查询操作的结果**，这明显有点生产者-消费者的意思，两次查询操作是生产者，对账操作是消费者。既然是生产-消费者模型，那就需要有个队列，来保存生产者生产的数据，而消费者则从这个队列消费数据。

针对对账，我们设计两个队列，两个队列的元素之间还有对应关系。具体如下图所示，订单查询操作将订单查询结果插入订单队列，派送单查询操作将派送单插入派送单队列，这两个队列的元素之间是有一一对应的关系的。两个队列的好处是，对账操作可以每次从订单队列出一个元素，从派送单队列出一个元素，然后对着两个元素执行对账操作，这样数据一定不会乱掉。

![](images/countDownLatchAndCyclicBarrier/queue.png)

下面再来看如何用双队列实现完全的并行。一个最直接的想法是：一个线程T1执行订单查询工作，一个线程T2执行派送单的查询工作，当线程T1和T2都各自生产完1条数据的时候，通知线程T3执行对账操作。这个想法虽然看上去简单，但其实还隐藏着一个条件，那就是线程T1和线程T2的工作要步调一致，不能一个跑的太快，一个跑的太慢，只有这样才能做到各自生产完1条数据的时候，通知线程T3。

下面这幅图形象的描述了上面的意图：线程T1和线程T2只有都生产完1条数据的时候，才能一起想下执行，也就是说，线程T1和线程T2要相互等待，步调要一直；同事当线程T1和T2都生产完一条数据的时候，还要能够通知线程T3执行对账操作。

![](images/countDownLatchAndCyclicBarrier/sync.png)

## 用CyclicBarrier实现线程同步

下面我们来实现上面提到的方案。这个方案的难点有两个：一个是线程T1和T2要做到步调一致，另一个是要能够通知到线程T3。

你依然可以利用一个计数器来解决这两个难点，计数器初始化为2，线程T1和T2生产完一条数据都将计数器减1，如果计数器大于0则线程T1或者线程T2等待。如果计数器等于0，则通知线程T3，并唤醒等待的线程T1或者T2，与此同时，将计数器重置为2，这样线程T1和线程T2生产下一条数据的时候就可以继续使用这个计数器了。

但我们不建议在项目中这么做，因为Java并发包里也已经提供了相关的工具类：CyclicBarrier。在下面的代码中，我们首先创建了一个计数器初始值为2的CyclicBarrier，你需要注意的是创建CyclicBarrier的时候，我们还要传入一个回调函数，当计数器减到0的时候，会调用这个回调函数。

线程T1负责查询订单，当查出一条时，调用barrier.await()来将计数器减1，同时等待计数器变成0；当T1和T2都调用barrier.await()的时候，计数器会减到0，此时T1和T2就可以执行下一条语句了，同时会调用barrier的回调函数来执行对账操作。

非常值得一提的是，**CyclicBarrier的计数器有自动重置的功能**，当减到0的时候，会自动重置你设置的初始值。这个功能用起来太方便了。

```java
// 订单队列
Vector<P> pos;
// 派送单队列
Vector<D> dos;
// 执行回调的线程池 
Executor executor = 
  Executors.newFixedThreadPool(1);
final CyclicBarrier barrier =
  new CyclicBarrier(2, ()->{
    executor.execute(()->check());
  });
  
void check(){
  P p = pos.remove(0);
  D d = dos.remove(0);
  // 执行对账操作
  diff = check(p, d);
  // 差异写入差异库
  save(diff);
}
  
void checkAll(){
  // 循环查询订单库
  Thread T1 = new Thread(()->{
    while(存在未对账订单){
      // 查询订单库
      pos.add(getPOrders());
      // 等待
      barrier.await();
    }
  });
  T1.start();  
  // 循环查询运单库
  Thread T2 = new Thread(()->{
    while(存在未对账订单){
      // 查询运单库
      dos.add(getDOrders());
      // 等待
      barrier.await();
    }
  });
  T2.start();
}

```

## 总结

CountDownLatch和CyclicBarrier是Java并发包提供的两个非常易用的线程同步工具类，这两个工具类用法的区别在这里还是有必要再强调一下的：**CountDownLatch主要用来解决一个线程等待多个线程的场景**，可以类比旅游团团长要等待所有的游客到齐才能去下一个景点；而**CyclicBarrier是一组线程之间的相互等待**，更想是几个驴友之间不离不弃。除此之外CountDownLatch的计数器是不能循环利用的，也就是说一旦计数器减到0，再有线程调用await()，该线程会直接通过。但**CyclicBarrier的计数器是可以循环利用的**，而且具备自动重置的功能，一旦计数器减到0会自动重置到你设置的初始值。除此之外，CyclicBarrier还可以设置回调函数，可以说功能丰富。