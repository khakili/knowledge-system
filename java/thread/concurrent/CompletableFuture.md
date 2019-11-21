# CompletableFuture
> Java SDK提供的异步编程工具类

## CompletableFuture的优势
   - 无需手动维护线程
   - 语义更清晰，更容易描述各个任务的依赖关系
   - 可以使开发人员更专注于业务逻辑，而不是过多关注任务的流转

## 创建CompletableFuture对象
   - runAsync(Runnable runnable)
   - supplyAsync(Supplier<U> supplier)
   - runAsync(Runnable runnable,Executor executor)
   - supplyAsync(Supplier<U> supplier,Executor executor)
    
    runAsync与supplyAsync的区别是后者有返回值，后两个方法与前两个方法的区别是
    后两个方法可以指定执行任务的线程池。
    
   **推荐指定执行任务的线程池，避免不同的业务相互干扰**

## CompletionStage接口
      CompletableFuture对象实现了CompletionStage接口，用来描述任务的
      时序关系。
   - 串行关系
   - 并行关系
   - 汇聚关系 


