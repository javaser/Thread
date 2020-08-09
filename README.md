# Thread
一、线程同步
  1、线程接口、竞争条件同步
     1.1 实现多线程接口 Class Task implements Runable{} 
     1.2竞争是：处理器或者多个线程有共同依赖的数据时发生的。
     1.3使用 synchronized 关键字可以实现多线程同步。
  2、内部锁和Synchronization
     2.1内部锁我们常称之为 monitor lock 它是与每个对象实体内在关联的隐藏锁，它会强制对象进行独占访问。线程对 所有加上synchronized 的方法调用时必须获取其内部锁。
     2.2锁释放：方法执行完毕、没有被处理或者捕获的抛出异常。
     
     
