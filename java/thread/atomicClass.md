# 原子类：无锁工具类的典范

前面我们多次提到一个累加器的例子，示例代码如下。在这个例子中，add10K()这个方法不是线程安全的，问题就出在变量count的可见性和count+=1的原子性上。可见性问题可以用volatile来解决，而原子性问题我们前面一直都是采用的互斥锁方案。

```java

public class Test {
  long count = 0;
  void add10K() {
    int idx = 0;
    while(idx++ < 10000) {
      count += 1;
    }
  }
}

```
其实对于简单的原子性问题，还有一种**无锁方案**。Java SDK并发包将这种无锁方案封装提炼之后，实现了一系列的原子类。不过，在深入介绍原子类的实现之前，我们先看看如何利用原子类解决累加器问题，这样你会对原子类有个初步的认识。

在下面的代码中，我们将原来的long型变量count替换为了原子类AtomicLong，原来的count += 1 替换成了count.getAndIncrement()，仅需要这两处的改动就能使add10()方法变成线程安全的，原子类的使用还是简单的。

```java

public class Test {
  AtomicLong count = 
    new AtomicLong(0);
  void add10K() {
    int idx = 0;
    while(idx++ < 10000) {
      count.getAndIncrement();
    }
  }
}

```

无锁方案相对互斥锁方案，最大的好处就是性能。互斥锁方案为了保证互斥性，需要执行加锁、解锁操作，而加锁、解锁操作本身就消耗性能；同事拿不到锁的线程还会进入阻塞状态，进而触发线程切换，线程切换对性能的消耗也很大。相比之下，无锁方案则完全没有加锁、解锁的性能消耗，同时还能保证互斥性，既解决了问题，又没有带来新的问题，可谓绝佳方案。那它是如何做到的呢？

## 无所方案的实现原理

其实原子类性能高的咪咪很简单，硬件支持而已。CPU为了解决并发问题，提供了CAS指令（CAS，全称是Compare And Swap，即"比较并交换"）。CAS指令包含3个参数：共享变量的内存地址A、用于比较的值B和共享变量的新值C；并且只有当内存中的地址A处的值等于B时，才能将内存中的地址A处的值更新为新值C。**作为一条CPU指令，CAS指令本身就是能够保证原子性的**。

你可以通过下面的CAS指令的模拟代码来理解CAS的工作原理。在下面的模拟程序中有两个参数，一个是期望值except，另一个是需要写入的新值newValue，**只有当目前count的值和期望值expect相等时，才会将count更新为newValue**。

```java

class SimulatedCAS{
  int count;
  synchronized int cas(
    int expect, int newValue){
    // 读目前 count 的值
    int curValue = count;
    // 比较目前 count 值是否 == 期望值
    if(curValue == expect){
      // 如果是，则更新 count 的值
      count = newValue;
    }
    // 返回写入前的值
    return curValue;
  }
}

```

你仔细地再次思考一下这句话，"只有当目前count的值和期望值except相等时，才会将count更新为newValue。"要怎么理解这句话呢？

对于前面提到的累加器的例子，count+=1的一个核心问题是：基于内存中的count的当前值A计算出来的count+=1为A+1，在将A+1写入内存的时候，很可能此时内存中count已经被其他线程更新过了，这样将会导致错误地覆盖其他线程写入的值。也就是说，只有当内存中count的值等于期望值A时，才能将内存中count的值更新为计算结果A+1，这不就是CAS的语义吗？！

使用CAS来解决并发问题，一般都会伴随着自旋，而所谓的自旋，其实就是循环尝试。例如，实现一个线程安全的count+=1操作，"CAS+自旋"的实现方案如下所示，首先计算newValue=count+1，如果cas(count,newValue)返回的值不等于count，则意味着线程在执行完代码①处之后，执行代码②处之前，count的值被其他线程更新过。那此时该怎么处理呢？可以采用自旋方案，就像下面代码中展示的，可以重新读count最新的值来计算newValue并尝试再次更新，直到成功。

```java

class SimulatedCAS{
  volatile int count;
  // 实现 count+=1
  addOne(){
    do {
      newValue = count+1; //①
    }while(count !=
      cas(count,newValue)); //②
  }
 

  // 模拟实现 CAS，仅用来帮助理解
  synchronized int cas(int expect, int newValue){
    // 读目前 count 的值
    int curValue = count;
    // 比较目前 count 值是否 == 期望值
    if(curValue == expect){
      // 如果是，则更新 count 的值
      count= newValue;
    }
    // 返回写入前的值
    return curValue;
  }
}

```

通过上面的示例代码，想必你已经发现了，CAS这种无锁方案，完全没有加锁、解锁操作，即便两个线程完全同事执行addOne()方法，也不会有线程被阻塞，所以相当于互斥锁方案来说，性能好了很多。

但是在CAS方案中，有一个问题可能会常被你忽略，那就是**ABA**的问题。什么是ABA问题呢？

前面我们提到"如果cas(count,newValue)返回的值**不等于**count，则意味着线程在执行完代码①处之后，执行代码②处之前，count的值被其他线程**更新过**"，那如果cas(count,newValue)返回的值**等于**count，是否就能够认为count的值没有被其他线程**更新过**呢？显然不是的，假设count原本是A，线程T1在执行完代码①处之后，执行代码②处之前，有可能count被线程T2更新成了B，之后又被T3更新会了A，这样线程T1虽然看到的一直是A，但其实已经被其他线程更新过了，这就是ABA问题。

可能大多数情况下我们并不关心ABA问题，例如数值的原子递增，但也不能说所有情况下都不关心，例如原子化的更新对象很可能就需要关心ABA问题，因为两个A虽然相当，但是第二个A的属性可能已经发生了变化，所以在使用CAS方案的时候，一定要先check一下。

## 看Java如何实现原子化的count += 1

在开始部分，我们使用原子类AtomicLong的getAndIncrement()方法替代了count += 1，从而实现了线程安全。原子类AtomicLong的getAndIncrement()方法内部就是基于CAS实现的，下面我们来看看Java是如何使用CAS来实现原子化的count += 1的。

在Java 1.8版本中，getAndIncrement()方法会转调unsafe.getAndAddLong()方法。这里this和valueOffset两个参数可以唯一确定共享变量的内存地址。

```java

final long getAndIncrement() {
  return unsafe.getAndAddLong(
    this, valueOffset, 1L);
}

```

unsafe.getAndAddLong()方法的源码如下，该方法首先会在内存中读取共享变量的值，之后循环调用compareAndSwapLong()方法来尝试设置共享变量的值，知道成功为止。compareAndSwapLong()是一个native方法，只有当内存中共享变量的值等于expected时，才会将共享变量的值更新为x，并且返回true；否则返回false。compareAndSwapLong的语义和CAS指令的语义的差别仅仅是返回值不同而已。

```java

public final long getAndAddLong(
  Object o, long offset, long delta){
  long v;
  do {
    // 读取内存中的值
    v = getLongVolatile(o, offset);
  } while (!compareAndSwapLong(
      o, offset, v, v + delta));
  return v;
}
// 原子性地将变量更新为 x
// 条件是内存中的值等于 expected
// 更新成功则返回 true
native boolean compareAndSwapLong(
  Object o, long offset, 
  long expected,
  long x);

```

另外，需要你注意的是，getAndAddLong()方法的实现，基本上就是CAS使用的经典范例。所以请你体会下面这段抽象后的代码片段，他在很多无锁程序中经常出现。Java提供的原子类里面CAS一般被实现为compareAndSet()，compareAndSet()的语义和CAS指令的语义的差别仅仅是返回值不同而已，compareAndSet()里面如果更新成功，则会返回true，否则返回false。

```java

do {
  // 获取当前值
  oldV = xxxx；
  // 根据当前值计算新值
  newV = ...oldV...
}while(!compareAndSet(oldV,newV);

```