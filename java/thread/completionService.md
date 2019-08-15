# CompletionService：如何批量执行异步任务？

我们来看下下面这段代码：

```java

// 创建线程池
ExecutorService executor =
  Executors.newFixedThreadPool(3);
// 异步向电商 S1 询价
Future<Integer> f1 = 
  executor.submit(
    ()->getPriceByS1());
// 异步向电商 S2 询价
Future<Integer> f2 = 
  executor.submit(
    ()->getPriceByS2());
// 异步向电商 S3 询价
Future<Integer> f3 = 
  executor.submit(
    ()->getPriceByS3());
    
// 获取电商 S1 报价并保存
r=f1.get();
executor.execute(()->save(r));
  
// 获取电商 S2 报价并保存
r=f2.get();
executor.execute(()->save(r));
  
// 获取电商 S3 报价并保存  
r=f3.get();
executor.execute(()->save(r));

```

这段代码本身没什么问题，分别起了三个线程异步执行询价，通过三次调用Future的get()方法获取询价结果，之后将询价结果保存在数据库中。但有个地方的处理需要注意，如果S1报价的耗时很长，那么即便获取电商S2报价的耗时很短，也无法让保存S2报价的操作先执行，因为这个主线程都阻塞在f1.get()操作上。这点小瑕疵你应该如何解决呢？

我们可以增加一个阻塞队列，获取到S1、S2、S3的报价都进入阻塞队列，然后在主线程中消费阻塞队列，这样就能保证先获取到的报价先存到数据库了。

```java

// 创建阻塞队列
BlockingQueue<Integer> bq =
  new LinkedBlockingQueue<>();
// 电商 S1 报价异步进入阻塞队列  
executor.execute(()->
  bq.put(f1.get()));
// 电商 S2 报价异步进入阻塞队列  
executor.execute(()->
  bq.put(f2.get()));
// 电商 S3 报价异步进入阻塞队列  
executor.execute(()->
  bq.put(f3.get()));
// 异步保存所有报价  
for (int i=0; i<3; i++) {
  Integer r = bq.take();
  executor.execute(()->save(r));
}  

```

## 利用CompletionService实现询价系统

不过在实际项目中，并不建议大家这样做，因为Java SDK并发包里已经提供了设计精良的CompletionService。利用CompletionService不但能帮你解决先获取到的报价先保存到数据库的问题，而且还能让代码更简练。

CompletionService的实现原理也是内部维护了一个阻塞队列，当任务执行结束把任务的执行结果加入到阻塞队列中，不同的是CompletionService是把任务执行结果的Future对象加入到阻塞队列中，而上面的示例代码是把任务最重的之行结果放入阻塞队列中。

### 如何创建CompletionService呢？

CompletionService 接口的实现类是ExecutorCompletionService，这个实现类的构造方法有两个，分别是：

```java

ExecutorCompletionService(Executor executor);

ExecutorCompletionService(Executor executor,BlockingQueue<Future<V>> completionQueue);

```

这两个构造方法都需要传入一个线程池，如果不指定comletionQueue，那么默认会使用无界的LinkerBlockingQueue。之后通过CompletionService接口提供的submit()方法提交了三个询价操作，这个询价操作将会被CompletionService异步执行。最后，我们通过CompletionService接口提供的take()方法获取一个Future对象（前面我们提到过，加入到阻塞队列中的是任务执行结果的Future对象），调用Future对象的get()方法就能返回询价操作的之行结果了。

```java

// 创建线程池
ExecutorService executor = 
  Executors.newFixedThreadPool(3);
// 创建 CompletionService
CompletionService<Integer> cs = new 
  ExecutorCompletionService<>(executor);
// 异步向电商 S1 询价
cs.submit(()->getPriceByS1());
// 异步向电商 S2 询价
cs.submit(()->getPriceByS2());
// 异步向电商 S3 询价
cs.submit(()->getPriceByS3());
// 将询价结果异步保存到数据库
for (int i=0; i<3; i++) {
  Integer r = cs.take().get();
  executor.execute(()->save(r));
}

```

## CompletionService接口说明

下面我们详细介绍下CompletionService接口提供的方法，CompletionService接口提供的方法有5个，这5个方法的方法签名如下所示。

其中，submit()相关的方法有两个。一个方法参数是Callable<V> task，前面利用CompletionService实现询价系统的示例代码中，我们提交任务就用的它。另外一个方法有两个参数，分别是Runable task和V result，这两个方法类似于ThreadPoolExecutor的<T> Future<T> submit(Runnable task,T result)。

CompletionService接口其余的3个方法，都是和阻塞队列相关的，take()、poll()都是从阻塞队列中获取并移除一个元素；它们的区别在于如果阻塞队列时空的，那么调用take()方法的线程会被阻塞，而poll()方法会返回null值。poll(long timeout,TimeUnit unit)方法支持以超时的方式获取并移除阻塞队列头部的一个元素，如果等待了timeout unit时间，阻塞队列还是空的，那么该方法会返回null值。

```java

Future<V> submit(Callable<V> task);
Future<V> submit(Runnable task, V result);
Future<V> take() 
  throws InterruptedException;
Future<V> poll();
Future<V> poll(long timeout, TimeUnit unit) 
  throws InterruptedException;

```

## 利用CompletionService实现Dubbo中的Forking Cluster

Dubbo 中有一种叫做**Forking的集群模式**，这种集群模式下，支持**并行地调用多个查询服务，只要有一个成功返回结果，整个服务就可以返回了**。例如你需要提供一个地址转坐标的服务，为了保证该服务的高可用和性能，你可以并行的调用3个地图服务商的API，然后只要有一个正确返回了结果r，那么地址转坐标这个服务就可以直接返回r了。这种集群模式可以容忍2个地图服务商服务异常，但是缺点是消耗的资源偏多。

```java

geocoder(addr) {
  // 并行执行以下 3 个查询服务， 
  r1=geocoderByS1(addr);
  r2=geocoderByS2(addr);
  r3=geocoderByS3(addr);
  // 只要 r1,r2,r3 有一个返回
  // 则返回
  return r1|r2|r3;
}

```

利用CompletionService可以快速实现Forking这种集群模式，比如下面的示例代码就展示了具体是如何实现的。首先我们创建了一个线程池excutor、一个CompletionService对象cs和一个Future<Integer>类型的列表futures，每次通过调用CompletionService的submit()方法提交一个异步任务，会返回一个Future对象，我们把这些Future对象保存在列表futures中。通过调用cs.take().get()，我们把这些Future对象保存在列表futures中。通过调用cs.take().get()，我们能够拿到最快返回的任务执行结果，只要我们拿到一个正确返回的结果，就可以取消所有任务并且返回最终结果了。

```java

// 创建线程池
ExecutorService executor =
  Executors.newFixedThreadPool(3);
// 创建 CompletionService
CompletionService<Integer> cs =
  new ExecutorCompletionService<>(executor);
// 用于保存 Future 对象
List<Future<Integer>> futures =
  new ArrayList<>(3);
// 提交异步任务，并保存 future 到 futures 
futures.add(
  cs.submit(()->geocoderByS1()));
futures.add(
  cs.submit(()->geocoderByS2()));
futures.add(
  cs.submit(()->geocoderByS3()));
// 获取最快返回的任务执行结果
Integer r = 0;
try {
  // 只要有一个成功返回，则 break
  for (int i = 0; i < 3; ++i) {
    r = cs.take().get();
    // 简单地通过判空来检查是否成功返回
    if (r != null) {
      break;
    }
  }
} finally {
  // 取消所有任务
  for(Future<Integer> f : futures)
    f.cancel(true);
}
// 返回结果
return r;

```

## 总结

当需要批量提交异步任务的时候建议你使用CompletionService。ComletionService将线程池Executor和阻塞队列BlockingQueue的功能融合在了一起，能够让批量异步任务的管理更简单。除此之外，CompletionService能够让异步任务的之行结果有序化，先执行完的先进入阻塞队列，利用这个特性，你可以轻松实现后续处理的有序性，避免无谓的等待，同时还可以快速实现诸如Forking Cluster这样的需求。

CompletionService的实现类ExecutorCompletionService，需要你自己创建线程池，虽然看上去有些啰嗦，但好处是你可以让多个ExecutorCompletionService的线程池隔离，这种隔离性能避免几个特别耗时的任务拖垮整个应用的风险。



