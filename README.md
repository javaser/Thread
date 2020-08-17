# Thread
一、线程同步
  1、线程接口、竞争条件同步
     1.1 实现多线程接口 Class Task implements Runable{} 
     1.2竞争是：处理器或者多个线程有共同依赖的数据时发生的。
     1.3使用 synchronized 关键字可以实现多线程同步。
  2、内部锁和Synchronization
     2.1内部锁我们常称之为 monitor lock 它是与每个对象实体内在关联的隐藏锁，它会强制对象进行独占访问。线程对 所有加上synchronized 的方法调用时必须获取其内部锁。
     2.2锁释放：方法执行完毕、没有被处理或者捕获的抛出异常。
     2.3当一个线程获取到对象锁是，其他线程需要获取相同锁时将会被阻塞，直到当前线程释放锁。
     2.4内部锁相关特点：唯一性、Thread.sleep()不释放锁、内部锁是可重入的（当调用一个方法的时候获取了锁，再调用其他方法就可以不需要重新获取锁）、构造函数不可以使用synchronized（因为只有创建对象的线程才会调用构造函数）、对于静态方法使用的是Class对象锁。
     注意： 内部锁是在对象上 不是在方法上。
  3、使用同步块
     3.1我们尽量使用synchronized(anyInstance){} 这种方式，它比在方法上使用会提升我们的程序性能。尽量减少同步块的大小（保证线程安全的前提下）。
     3.2 同步中加锁的方式实现：
           public class LazyInitDemo{
               private List<String> list;
               public static void main (String[] args) throws InterruptedException {
               LazyInitBlockDemo obj = new LazyInitBlockDemo();
              new Thread(()->{
              System.out.println("thread1 : " + System.identityHashCode(obj.getList()));
    }).start();

             new Thread(()->{ 
             System.out.println("thread2 : " + System.identityHashCode(obj.getList()));
             
    }).start();
    
     private synchronized  List<String> getList () {
        if (list == null) {
            list = new ArrayList<>();
        }
        return list;
    }
    
               }
  
  }
  =======================================================================================================================================
  对于加锁我们开始减少加锁区域：
   private List<String> getList(){
      if(list==null){
      synchronized(this){
        if(list==null){
      list=new ArrayList();
     }
    }
  }
  return list;
}
我们还可以使用不同锁实现：
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
     private List<String> getList2(){
      if(list2==null){
      synchronized(lock2){
        if(list2==null){
      list2=new ArrayList();
     }
    }
  }
  return list2;
}
     private List<String> getList1(){
      if(list1==null){
      synchronized(lock1){
        if(list1==null){
      list1=new ArrayList();
     }
    }
  }
  return list1;
}
我们也可以使用String 作为锁对象（简写代码）
  private Map<String,Object> lock=new HashMap();
  private Object getLock(String path){
    if(!lock.containsKey(path)){
      lock.put(path,new Object);
    }
  }
  
 4、死锁（例如A、B两个线程，A持有一个B需要获取的锁，B持有A需要获取的锁，此时死锁产生）
  public static void main (String[] args) throws InterruptedException {
        List<Integer> list1 = new ArrayList<>(Arrays.asList(2, 4, 6, 8, 10));
        List<Integer> list2 = new ArrayList<>(Arrays.asList(1, 3, 7, 9, 11));
        Thread thread1 = new Thread(() -> {
            moveListItem(list1, list2, 2);
        });
        Thread thread2 = new Thread(() -> {
            moveListItem(list2, list1, 9);
        });

        thread1.start();
        thread2.start();
    }
    private static void moveListItem (List<Integer> from, List<Integer> to, Integer item) {
        log("attempting lock for list", from);
        synchronized (from) {
            log("lock acquired for list", from);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log("attempting lock for list ", to);
            synchronized (to) {
                log("lock acquired for list", to);
                if(from.remove(item)){
                  to.add(item);
                }
                log("moved item to list ", to);
            }
        }
    }
    private static void log (String msg, Object target) {
        System.out.println(Thread.currentThread().getName() +
                                           ": " + msg + " " +
                             System.identityHashCode(target));
    }
}
 如何防止死锁发生： java在编译期间不提供任何死锁的检查，我们在开发是尽量避免直接或间接递归调用持有锁的同一方法、使用更细粒度的锁、使用同步.
 二、Java内存模型
 1、为什么需要内存模型？
    对于程序员来说 很难推断出具体的 硬件内存架构、处理器优化和JIT编译器优化。为了便于使用Java内存模型指定了Java编程构造和设计准则，让开发者不必去推断底层永健和JIT编译器的优化细节。
 2、变量可见性
    在多处理器/多核心的环境中，为了数据的性能会存在多级缓存，所以对于共享数据需要特殊的处理器级执行来刷新主内存数据，从而时其他线程看到这个线程的写操作。
 3、代码重排序
    Java环境中代码可能会通过 JIT编译优化，编译是内嵌的是在底层实现的。该优化在单线程下是不受影响的，但是在多线程环境下对一些共享数据执行读/写操作，会出现不安全的情况。
 4、顺序一致性
    顺序一致性是一个非常强大的保证，它保证了程序执行中的可见性和顺序。在顺序一致的执行中，所有单个操作(如读和写)的总顺序与程序的顺序一致。每个单独的操作都是原子性的，对每个线程都是立即可见的。
 5、究竟什么是Java内存模型
  5.1、程序必须正确同步，以避免重新排序和可见性问题。
  5.2、如果程序 按照之前发生原则、避免数据竞争（如果访问中至少有一个是写操作，那么使用相同共享字段或数组变量的两个操作没有遵循 happens-before 原则的冲突）。
 6、synchronized 阻塞块
    6.1、两个或多个线程之间共享的阻塞代码的互斥。这意味着它们在同步块中实现了与多线程访问相关的复合操作的原子性
    6.2、上锁和解锁都会是本地内存进行刷新，从而解决内存可见问题。
 7、 JSR-133 对每个变量的默认值(0、false或null)的写入与每个线程中的第一个操作同步。
 三、线程池
  
     
