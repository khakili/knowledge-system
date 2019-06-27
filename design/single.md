为什么要学单例模式
> - 单例模式特点
> - 单例模式应用场景
> - 如何安全有效使用单例模式
> - 单例模式优点和缺点

## 单例模式特点

1.单例模式介绍

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

2.单例模式特点

单例类中只能有一个实例

单例类必须自己创建自己唯一实例

单例类需要统一对外暴露获取实例的接口方法


## 单例模式使用场景

**1、支付过程中，生成唯一交易流水号。**

**2、与数据库交互过程中，频繁的获取数据库连接。**

```
/**
 * @author jiangtao
 * @version V1.0
 * @Description：单例模式获取唯一序列 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class SingleSequenceFacory {
    //全局唯一序列
    private volatile static int sequence;
    //1.私有构造器
    private SingleSequenceFacory(){};
    //2.私有的静态属性
    private static SingleSequenceFacory singleSequenceFacory = new SingleSequenceFacory();
    //3.公开的方法获取序列的实例
    public static SingleSequenceFacory getInstance(){
        return singleSequenceFacory;
    }
    //获取自增序列
    public int getSequence(){
        return sequence++;
    }

    public int get(){
        return sequence;
    }
}
//测试代码
public static void main(String[] args) throws InterruptedException {
        //1.单例流水工厂，是不可被实例化的，因为构造器私有化
        //SingleSequenceFacory SingleSequenceFacory = new SingleSequenceFacory();
        //2.模拟多线程场景获取流水号
        for (int i = 0; i <10 ; i++) {
            int finalI = i;
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                        //3.通过实例工程获取实例对象，调用自增方法获取流水号
                        SingleSequenceFacory singleSequenceFacory = SingleSequenceFacory.getInstance();
                        System.out.println("第【"+ finalI +"】个流水号="+singleSequenceFacory.getSequence());
                }
            });
            thread.start();
        }
        Thread.sleep(1000);
        //打印多线程执行完成后，最后一次出现的流水号
        System.out.println("=====执行多线程结束后，序列号结果="+SingleSequenceFacory.getInstance().get());
        System.out.println(SingleSequenceFacory.getInstance());
        System.out.println(SingleSequenceFacory.getInstance());
    }
```
这段代码的输出如下：
```
第【0】个流水号=0
第【2】个流水号=1
第【4】个流水号=2
第【6】个流水号=3
第【7】个流水号=4
第【8】个流水号=6
第【3】个流水号=5
第【1】个流水号=7
第【5】个流水号=8
第【9】个流水号=9
=====执行多线程结束后，序列号结果=10
com.example.factory_demo.singleton.SingleSequenceFacory@4a574795
com.example.factory_demo.singleton.SingleSequenceFacory@4a574795


```
从上面的例子中可以看出，《恶汉模式》无论多少个线程去获取流水实例，都能拿的到相同的实例
上面的演示demo中，序列类实例是提前实例化完成的。

咱们接下来在看看《懒汉模式》，相比《恶汉模式》懒汉模式有什么优势
>优势：节省内存空间，什么时候时候什么时候初始化
>实现难度：易
>是否线程安全：否

```
/**
 * @author jiangtao
 * @version V1.0
 * @Description：单例模式获取唯一序列 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class SingleSequenceFacory {
    //全局唯一序列
    private volatile static int sequence;
    //1.私有构造器
    private SingleSequenceFacory(){};
    //2.私有的静态属性
    private static SingleSequenceFacory singleSequenceFacory ;
    //3.公开的方法获取序列的实例
    public static SingleSequenceFacory getInstance(){
        if(null==singleSequenceFacory){
            singleSequenceFacory =  new SingleSequenceFacory();
        }
        return singleSequenceFacory;
    }
    //获取自增序列
    public int getSequence(){
        return sequence++;
    }

    public int get(){
        return sequence;
    }

}

```
很明显在执行3部时，存在线程安全问题
```
 //3.公开的方法获取序列的实例
    public static SingleSequenceFacory getInstance(){
        if(null==singleSequenceFacory){
            singleSequenceFacory =  new SingleSequenceFacory();
        }
        return singleSequenceFacory;
    }
```

## 如何安全有效的使用单例模式
>- 饿汉模式：推荐使用
>- 懒汉模式：推荐使用静态内部类

如何使用静态内部类来实现单例模式
```
/**
 * @author jiangtao
 * @version V1.0
 * @Description：单例模式获取唯一序列 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class SingleSequenceFacory {
    //全局唯一序列
    private volatile static int sequence;
    //1.私有构造器
    private SingleSequenceFacory(){};
    //2.静态内部类实例化
    private static class SingleHolder{private static SingleSequenceFacory instance = new SingleSequenceFacory();}
    //3.公开的方法获取序列的实例
    public static SingleSequenceFacory getInstance(){
        return SingleHolder.instance;
    }
    //获取自增序列
    public int getSequence(){
        return sequence++;
    }

    public int get(){
        return sequence;
    }

}
```
运行结果
```
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
com.example.factory_demo.singleton.SingleSequenceFacory@488a7f84
```
静态内部类是如何保证在多线程场景下，获取的实例是相同的？
静态内部类利用了类加载机制原理实现了高并发场景下只初始化一个实例，classloader保证类的instance方法只有一个线程在处理。


## 单例模式优点和缺点

主要优点如下：
> - 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例
> - 避免对资源的多重占用（比如写文件操作）。

主要缺点如下：
> - 没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。


## 知识扩展
是否静态内部类能保证jvm内存中永远只存在一个实例
利用反射机制破坏单例模式中jvm内存中只有一个实例

```
 public static void main(String[] args) throws InterruptedException {
        System.out.println(SingleSequenceFacory.getInstance());
        Class factory = SingleSequenceFacory.class;
        try {
            //获取当前Class所表示类中指定的一个的构造器,和访问权限无关
            Constructor<?> constructor = factory.getDeclaredConstructor();
            constructor.setAccessible(true);
            try {
                SingleSequenceFacory singleSequenceFacory = (SingleSequenceFacory) constructor.newInstance();
                System.out.println(singleSequenceFacory);
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }

    }
```

执行结果：
>com.example.factory_demo.singleton.SingleSequenceFacory@4a574795
com.example.factory_demo.singleton.SingleSequenceFacory@f6f4d33