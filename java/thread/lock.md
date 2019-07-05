# Lock&Condition:并发包中的管程

在并发编程领域，两大核心问题：
> - 一个是互斥，即同一时刻只允许一个线程访问共享资源；
> - 另一个是同步，即线程之间如何通信、协作。

Java SDK并发包通过Lock和Condition两个接口来实现管程，其中的Lock用于解决互斥问题，Condition用于解决同步问题。

本节重点介绍Lock的使用，在这之前，我们先思考一个问题：Java语言本身提供的synchronized也是管程的一种实现，既然Java从语言层面已经实现了管程，为什么SDK还要提供另一种实现呢？

** 再造管程的理由

我们在解决死锁问题的时候，提出了一个**破坏不可抢占条件**方案，但是这个方案synchroized没有办法解决。原因是synchroized在申请资源时，如果申请不到，线程直接阻塞，而我们希望的是：
> 对于"不可抢占"这个条件，占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它战友的资源，这样不可抢占条件就破坏掉了。

我们可以有三种方案解决这个问题：
1. **能够响应中断**。synchroized的问题是，持有锁A后，如果尝试获取锁B失败，那么线程就进入阻塞状态，一旦发生死锁，就没有任何机会来唤醒阻塞的线程。但如果阻塞状态的线程能响应中断信号，也就是说当我们给阻塞的线程发送中断信号的时候，能够唤醒它，那它就有机会释放曾经持有的锁A。这样就破坏了不可抢占条件了。
2. **支持超时**。如果线程在一段时间之内没有获取到锁，不是进入阻塞状态，而是返回一个错误，那这个线程也有机会释放曾经持有的锁。
3. **非阻塞地获取锁**。如果尝试获取锁失败，并不进入阻塞状态，而是直接返回，那它就有机会释放曾经持有的锁。

这三种方可以全面弥补synchroized的问题，体现在API上，就是Lock的三个接口方法：

```java
    void lockInterruptibly() throws InterruptedException;    
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    boolean tryLock();
```

** 如何保证可见性

Java多线程里的可见性是通过Happens-Before规则保证的，其中有一条synchroized相关的规则：synchroized的解锁Happens-Before于后续对这个锁的加锁。那Lock靠什么保证可见性呢？

Lock**利用了volatile相关的Happens-Before规则**。

Java SDK里面的ReentrantLock，内部持有一个volatile的成员变量state，获取锁的时候，会重写state的值；解锁的时候，也会读写state的值。

** 什么是可重入锁

ReentrantLock，翻译过来叫**可重入锁**。所谓可重入锁，顾名思义，指的是**线程可以重复获取同一把锁**。

```java

class X {
  private final Lock rtl =
  new ReentrantLock();
  int value;
  public int get() {
    // 获取锁
    rtl.lock();         ②
    try {
      return value;
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
  public void addOne() {
    // 获取锁
    rtl.lock();  
    try {
      value = 1 + get(); ①
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
}

```
上面代码中，当线程T1执行到①处，已经获取到了锁rtl，当在①处调用get()方法时，会在②再次对锁rtl进行加锁操作。此时，如果锁rtl是可重入的，那么线程T1可以再次加锁成功；如果rtl是不可重入的，那么线程T1此时会被阻塞。

除了可重入锁，可能你还听说过**可重入函数**。所谓**可重入函数，指的是多个线程可以同时调用该函数**，每个线程都能得到正确结果；同时在一个线程内支持线程切换，无论被切换多少次，结果都是正确的。所以，可重入函数是线程安全的。

