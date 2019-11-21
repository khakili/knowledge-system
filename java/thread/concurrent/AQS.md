# ReentrantLock中的AQS

## 什么是AQS
	AQS - AbstractQueuedSynchronizer.
	它是JAVA SDK提供的一个抽象的队列同步器，它提供了一个抽象阻塞的先进先出队列。
	
## CLH队列结构
            +------+  prev +-----+       +-----+
       head |      | <---- |     | <---- |     |  tail
            +------+       +-----+       +-----+

		AQS中的CLH队列是CLH同步锁的一种变形，在结构上引入了头结点和尾节点，他们分别指向队列的头和尾，尝试获取
		锁、入队列、释放锁等实现都与头尾节点相关，并且每个节点都引入前驱节点和后后续节点的引用；在等待机制上由原
		来的自旋改成阻塞唤醒。

**AQS不关注如何获取/释放，它关注的是失败之后应该怎么做**

## Node对象分析

```java 

static final class Node {
		//共享节点
        static final Node SHARED = new Node();
		//独占节点
        static final Node EXCLUSIVE = null;

		//取消标志
        static final int CANCELLED =  1;
		//唤醒标志
        static final int SIGNAL    = -1;
		//条件标志，用于独占模式
        static final int CONDITION = -2;
       //传播标志，用于共享模式
        static final int PROPAGATE = -3;

        //等待状态
        volatile int waitStatus;
		//前置节点
        volatile Node prev;

       //后置节点
        volatile Node next;
        //当前节点持有的线程
        volatile Thread thread;

        //模式标志
        Node nextWaiter;

        /**
         * 返回是否为共享模式
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        /**
	      *返回前置节点
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```
- Node.waitStatus

|值|状态|解释|
|:--:|:--:|:--:|
|1| CANCELLED | 取消状态|
|-1| SIGNAL |等待触发状态|
|-2| CONDITION |等待条件状态|
|-3| PROPAGATE |状态需要向后传播|


## 独占模式

### acquire

获取独占锁流程

- 调用自定义同步器的tryAcquire()尝试直接去获取资源，如果成功则直接返回；
- 没成功，则addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
- acquireQueued()使线程在等待队列中休息，有机会时（轮到自己，会被unpark()）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
- 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。


```java 
//获取独占锁
public final void acquire(int arg) {
		//tryAcquire 获取方法忽略
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            //线程在等待时被中断过，则在获取资源后才再进行自我中断selfInterrupt()，将中断补上。
            selfInterrupt();
}
private Node addWaiter(Node mode) {
		 //构造节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        //如果尾节点不是空，则将该节点设置为尾节点
        if (pred != null) {
        	  //设置当前节点的前置节点为尾节点
            node.prev = pred;
            //CAS，防止尾节点被修改
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //入队
        enq(node);
        return node;
}
//入队操作（自旋直到可以入队尾）
private Node enq(final Node node) {
			//自旋
        for (;;) {
            Node t = tail;
            //如果尾节点空，则初始化(现在为空队列)
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                		//头尾为同一个node
                    tail = head;
            } else {
            		//实际入队操作
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
    //自旋等待获取锁
final boolean acquireQueued(final Node node, int arg) {
		 //失败标志
        boolean failed = true;
        try {
        //中断标志
            boolean interrupted = false;
            for (;;) {
            		//获取前置节点
                final Node p = node.predecessor();
                //如果前置节点是头节点，并获取锁成功
                if (p == head && tryAcquire(arg)) {
                		//设置头结点
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //判断是否需要在获取锁失败后挂起当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                		//挂起线程是否中断
                    parkAndCheckInterrupt())
                    //设置中断标志，后补用
                    interrupted = true;
            }
        } finally {
        	  //如果上边操作有失败，则将节点设为取消状态
            if (failed)
                cancelAcquire(node);
        }
}
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
		 //获取节点等待状态
        int ws = pred.waitStatus;
        //唤醒标志
        if (ws == Node.SIGNAL)
            return true;
        //ws >0 为 Node.CANCELLED
        if (ws > 0) {
           //自旋，直到节点状态小于等于0
            do {
            	   //设置当前节点
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
           	//cas,修改节点状态为Node.SIGNAL
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```




### release

```java
//释放锁
 public final boolean release(int arg) {
 		//尝试释放锁
        if (tryRelease(arg)) {
            Node h = head;
            //如果头结点不是空并且头结点的状态不是0
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    //唤醒节点
    private void unparkSuccessor(Node node) {
       
        int ws = node.waitStatus;
        //如果节点状态小于0 ，将其更新为0 
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        //后置节点状态大于0 
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
            		//从尾节点往前循环，取有效节点
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
        		//唤醒后置节点线程
            LockSupport.unpark(s.thread);
    }
```
## 共享模式

### acquireShared

获取共享锁流程

- tryAcquireShared，尝试获取锁，成功则操作结束
- 失败使用doAcquireShared进入队列，直到获取到锁才返回

```java 
//共享锁获取
 public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
    
 private void doAcquireShared(int arg) {
 		// 添加一个共享节点
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            //自旋
            for (;;) {
                final Node p = node.predecessor();
                //如果已经是头结点，尝试获取共享锁
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    //获取成功
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
