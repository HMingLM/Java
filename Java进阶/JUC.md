[toc]

# 回顾多线程

### 1.Java真的可以开启线程吗？

不可以，Java无法直接操作硬件

new Thread().start，此处的start会执行 private void native start0() 方法，调用的是本地操作系统的方法


### 2.线程有几个状态

- new 新生
- runnable 运行
- blocked 阻塞
- waiting 等待
- timed_waiting 超时等待 
- terminated 终止


### 3.wait()和sleep()的区别

- 来自不同的类
  - wait- -》 Object
  - sleep- -》Thread
- 关于锁的释放
  - wait- -》 释放锁
  - sleep- -》不释放锁
- 使用范围不同
  - wait- -》 必须在同步代码块中
  - sleep- -》任何地方
- 捕获异常
  - wait- -》 不需捕获异常【好像也要的】
  - sleep- -》需要捕获异常

**实际开发中用JUC下的TimeUnit来让线程睡眠**


### 4.Lock 与 synchronized 的区别

JUC.Locks.Lock   、  JUC.Locks.condition


> 公平锁：先来后到

> 非公平锁：可以插队

> Lock默认使用非公平锁【查看reentrantLock源码可知】，也可以设置为公平锁


- synchronized 是内置关键字
  - Lock是接口
- synchronized 无法判断是否获取了锁
  - 而Lock可以
- synchronized 自动释放锁
  - Lock需手动释放
- synchronized 若获得锁的线程阻塞，其他线程傻等
  - 而Lock不一定【tryLock尝试获取锁】
- synchronized 可重入锁，不可判断，非公平
  - Lock可重入锁，可判断，非公平【可设置】
- synchronized 适合少量代码
  - Lock适合大量代码


> 可重入：就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁。

### 5.锁的对象

解决这类问题的根本：

- 非静态方法：synchronized(this) 锁的对象是方法的调用者
- 静态方法：  synchronized(xx.class) 


# 生产者与消费者问题

传统synchronized--》Lock--》用Condition精准唤醒

> 可见idea代码JUCStudy--ProducerAndConsumer，代码如下


```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
//生产者和消费者问题【synchronized / Lock】
//Lock替换synchronized方法和语句的使用， Condition取代了对象监视器方法的使用。 可精准通知唤醒线程
//注意防止多个线程时出现【虚假唤醒】：等待应该总是出现在循环中 while
public class ProducerAndConsumer {
    public static void main(String[] args) {
        Data data = new Data() ;
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    data.increate();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },"A").start();
        }
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    data.decreate();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },"B").start();
        }
    }

}

//资源类【synchronized，wait，notifyAll】
class Data {
    private int num = 0 ;

    //+1
    public synchronized void increate() throws InterruptedException {
        while (num!=0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        this.notifyAll();
    }

    //-1
    public synchronized void decreate() throws InterruptedException {
        while (num==0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        this.notifyAll();
    }
}

//资源类【Lock，await，signal/signalAll】
class Data2 {
    private int num = 0 ;
    Lock lock = new ReentrantLock();
    Condition condition1 = lock.newCondition();
    Condition condition2 = lock.newCondition();

    public void increate() {
        try {
            lock.lock();
            while (num!=0) {
                condition1.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName()+"=>"+num);
            condition2.signal();    //唤醒指定线程-2
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decreate() {
        try {
            lock.lock();
            while (num==0) {
                condition2.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName()+"=>"+num);
            condition1.signal();     //唤醒指定线程-1
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> java.util.ConcurrentModificationException  并发修改异常



# 并发场景

### 1.安全的集合类

CopyOnWriteArrayList

CopyOnWriteArraySet

ConcurrentHashMap


```
    public static void main(String[] args) {
        //ArrayList<String> list = new ArrayList<>();   java.util.ConcurrentModificationException  并发修改异常
        /*
         * 并发下ArrayList不安全，此处有三种解决方案 ：
         * 1.List<String> list = new Vector<>();     Vector底层用的是synchronized，效率低
         * 2.List<String> list = Collections.synchronizedList(new ArrayList<>());
         *  ---------------------------------
         * 3.List<String> list = new CopyOnWriteArrayList<>();      底层用的是Lock
         *      写入时复制【COW，计算机程序设计领域的一种优化策略】
         *      在写入时避免覆盖，造成数据问题
         */
        //  CopyOnWriteArraySet同理！
        //  探究 ConcurrentHashMap

        List<String> list = new CopyOnWriteArrayList<>();

        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5)) ;
                System.out.println(list);
            },String.valueOf(i)).start();
        }

        //=========================题外话=================================
        //**新的打印方式**
        List<String> strings = Arrays.asList("1", "2", "3");
        strings.forEach((s)-> System.out.println(s));
        strings.forEach(System.out::println);

        PrintStream out = System.out;
        Consumer<String> fun = out::println ;   //::表示引用println方法
        fun.accept("Hello::");
    }
```


### 2.Callable

FutureTask


```
        方式三：实现Callable接口    【它是JUC包下的接口，，有返回值，效率高！】
           和实现Runnable接口的区别就是，Callable带泛型，其call方法有返回值，且可以抛异常。
            使用的时候，需要用FutureTask来接收返回值。
              而且它也要等到线程执行完调用get方法才会执行，也可以用于闭锁操作。
          - 问：为什么是用FutureTask？
          - 答：启动线程只有一种方式： new Thread( Runnable )
                而FutureTask是Runnable的实现类，它将Callable实现类作为构造方法的参数，从而间接启动线程
          - Callable的细节：
                i、有缓存
                ii、结果可能需要等待，会阻塞【可以用异步通信解决】
                

CallableImpl callable = new CallableImpl();
        // 执行 Callable 方式,需要 FutureTask 实现类的支持,用于接收运算结果, FutureTask 是 Future 接口的实现类
        FutureTask<Integer> result = new FutureTask<>(callable);
        new Thread(result).start();
        try {
            //只有当 Thread 线程执行完成后,才会打印结果; 因此, FutureTask 也可用于闭锁
            System.out.println(result.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
```


### 3.三个并发辅助类

CountDownLatch


```
//常用的并发辅助类---CountDownLatch
/*
    【JUC】下的CountDownLatch，一个同步辅助类，理解为减法计数器
       在完成某些运算时，只有其他所有线程的运算全部完成，当前运算才继续执行。这就叫【闭锁】。
        方法：  countDown() 数量-1         await() 等待计数器归零，再向下执行

    另一案例：读 src\DemoCountDownLatch
 */
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"- Go Out!");
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        //主线程等待6个执行完
        countDownLatch.await();
        System.out.println("Close the door");
    }
}
```



CyclicBarrier


```
//常用的并发辅助类---CyclicBarrier （与CountDownLatch相对应，加法）
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> System.out.println("召唤神龙"));
        for (int i = 1; i <= 7; i++) {
            final int temp = i;     //改变变量的作用域
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集第"+temp+"个龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();

        }
    }
}
```



Semaphore


```
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

//常用的并发辅助类---Semaphore
/* 原理：
    semaphore.acquire();  获得，假如已经满了，则等待。
    semaphore.release();  释放，会将当前信号量释放（+1），任何换新等待的线程

   作用：多个共享资源互斥地使用，并发限流，控制最大的线程数
 */
public class SemaphoreDemo {
    public static void main(String[] args) {
        //线程数量  停车位     【限流】
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();    //得到
                    System.out.println(Thread.currentThread().getName() + "抢到车位");

                    TimeUnit.SECONDS.sleep(2);  //睡2s
                    System.out.println(Thread.currentThread().getName() + "离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();    //释放
                }
            },String.valueOf(i)).start();

        }
    }
}
```


### 4.ReadWriteLock

java.util.Locks.ReadWriteLock

> A ReadWriteLock维护一对关联的Locks，一个用于只读操作，一个用于写入【共享锁】

> 读可以被多个线程同时读，写的时候只能有一个线程去写【独占锁】


```
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyLockCache myLockCache = new MyLockCache();
        //写入
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(()->{
                myLockCache.put(temp+"",temp);
            },String.valueOf(i)).start();
        }

        //读取
        for (int i = 1; i <= 5; i++) {
            final int temp = i ;
            new Thread(()->{
                myLockCache.get(temp+"");
            },String.valueOf(i)).start();
        }
    }
}

class MyLockCache {
    private volatile HashMap<String,Object> hashMap = new HashMap<>();
    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    //写入
    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"写入"+key);
            hashMap.put(key,value);
            System.out.println(Thread.currentThread().getName()+"写入完成"+key);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    //读取
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"读取"+key);
            Object value = hashMap.get(key);
            System.out.println(Thread.currentThread().getName()+"读取完成"+value);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```


### 5.阻塞队列

![avatar](D:/其他/图片/study/BlockingQueue.png)

BlockingQueue

使用场景：多线程并发处理，线程池

**四组API**


方式 | 抛出异常 | 有返回值，不抛异常 | 阻塞 一直等待 | 阻塞 超时等待
---|---|---|---|---
添加  | add() | offer() | put() | offer(,,)
移除 | remove() | poll() | take() | poll(,)
判断队首元素 | element() | peek() | - | -


```
/**
     * 1.抛出异常
     */
    public static void test1() {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.add("a"));     //true
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        //System.out.println(blockingQueue.add("d"));     异常：IllegalStateException: Queue full

        System.out.println("队首:"+blockingQueue.element());    //查看队首元素    "a"

        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
//        System.out.println(blockingQueue.remove());     异常：java.util.NoSuchElementException
    }
```


```
/**
     * 2.有返回值，不抛出异常
     */
    public static void test2() {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.offer("a"));   //true
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("d"));   //false 不抛出异常

        System.out.println("队首:"+blockingQueue.peek());   //查看队首元素

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());       //null 不抛出异常
    }
```


```
/**
     * 3.等待，一直阻塞
     */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        blockingQueue.put("d");     //没有位置了，一直阻塞

        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());   //没有元素了，一直阻塞
    }
```


```
/**
     * 4.等待，超时等待
     */
    public static void test4() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        blockingQueue.offer("a");
        blockingQueue.offer("b");
        blockingQueue.offer("c");
        blockingQueue.offer("d",2, TimeUnit.SECONDS);   //等待2秒就退出

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll(2,TimeUnit.SECONDS));
    }
```


### 6.同步队列

SynchronousQueue

和其他BlockingQueue不一样，SynchronousQueue不存储元素，put进去一个元素，必须等待take取出来，才能往里边再放一个元素


```
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+" put 1");
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+" put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+" put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+" get "+blockingQueue.take());
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+" get "+blockingQueue.take());
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+" get "+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
```

# 线程池

池化技术：线程池、连接池、内存池、常量池

好处：
- 1.降低资源消耗
- 2.提高响应速度
- 3.方便管理

线程复用，控制最大并发数、管理线程


### 1.三大方法【规约不允许】


```
/**
     * 1.三大方法
     *      Executors 工具类
     */
    public static void Demo01() {
//        ExecutorService executorService = Executors.newSingleThreadExecutor();  //单个线程的线程池
//        ExecutorService executorService = Executors.newFixedThreadPool(5);    //创建一个固定大小的线程池
        ExecutorService executorService = Executors.newCachedThreadPool();    //创建一个大小可伸缩的线程池【遇强则强】

        // 注意使用线程池来创建线程
        try {
            for (int i = 0; i < 10; i++) {
                executorService.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" ok");
                });
            }
        } finally {
            //关闭线程池
            executorService.shutdown();
        }
    }
```


### 2.七大参数

观察源码

```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,     
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    
  !!  Integer.MAX_VALUE  21亿，OOM  !!
```

发现创建线程池的**三大方法，本质都调用了ThreadPoolExecutor**

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

七大参数：
- int corePoolSize,         核心线程池大小
- int maximumPoolSize,      最大核心线程池大小
- long keepAliveTime,       超时了没有人调用就回释放
- TimeUnit unit,            超时的单位
- BlockingQueue<Runnable> workQueue,    阻塞队列    
- ThreadFactory threadFactory,          线程工厂，创建线程的，一般不用动
- RejectedExecutionHandler handler      拒绝策略

回看三大方法的参数内容，发现Executor创建线程池的方法存在风险【OOM】，阿里规约如下：

![avatar](D:/其他/图片/study/线程池规约.png)

理解七大参数

![avatar](D:/其他/图片/study/七大参数.png)

### 3.四种拒绝策略

![avatar](D:/其他/图片/study/四种拒绝策略.png)


```
四种拒绝策略：
     *      默认：new ThreadPoolExecutor.AbortPolicy()        //银行满了还有人来，不处理，【抛异常】
     *         new ThreadPoolExecutor.CallerRunsPolicy()   //哪来的去哪里 【执行结果：main ok】
     *         new ThreadPoolExecutor.DiscardPolicy()      //队列满了，【丢掉任务】，不会抛出异常
     *         new ThreadPoolExecutor.DiscardOldestPolicy()      //队列满了，尝试【去和最早的竞争】
```



### 4.自定义线程池【线程池的正确打开方式】


```
    /**
     * 自定义线程池【线程池的正确打开方式】
     */
    public static void Demo02() {
        ExecutorService executorService = new ThreadPoolExecutor(
                2,          //核心
                //5,                      //最大
                Runtime.getRuntime().availableProcessors(),          //最大
                3,                      //超时
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),       //阻塞队列【银行等候区】
                new ThreadPoolExecutor.DiscardOldestPolicy()      //队列满了，尝试去和最早的竞争
                );

        try {
            //注意，最大承载量：max+Deque=8 【银行全部窗口+等候区】
            for (int i = 1; i <= 9; i++) {      // 9 时----java.util.concurrent.RejectedExecutionException
                executorService.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" ok");
                });
            }
        } finally {
            //关闭线程池
            executorService.shutdown();
        }
    }
```

### 5.CPU密集型 && IO密集型

- 问：**线程池的最大大小该如何设置：**
- 答：两种类型【调优】
  - CPU密集型：计算机几核就是几，CPU效率最高
    - Runtime.getRuntime().availableProcessors()
  - IO密集型：判断，要大于你的程序中十分耗IO的线程数
    - 如程序中有15个大型任务，IO十分占用资源，可设置最大线程数30，保证有15个处理其他


# 函数式接口

函数式接口：**简化编程模型**

掌握新特性：Lambda表达式、链式编程、函数式接口、Stream流式计算

### 1. Function


```
//Function 函数型接口【转换型接口】
    public static void FunctionDemo(){
        /*Function<String,String> function = new Function<String,String>() {
            @Override
            public String apply(String str) {
                return str;
            }
        } ;*/
        Function<String,Integer> function = str->{return Integer.valueOf(str); } ;

        System.out.println(function.apply("123")); //adsf
    }
```

### 2.Predicate


```
//Predicate 断定型接口   一个参数，返回值是boolean
    public static void PredicateDemo() {
        /*Predicate<String> predicate = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.isEmpty();
            }
        } ;*/
        //判断字符串是否为空
        Predicate<String> predicate = s -> {return s.isEmpty();} ;

        System.out.println(predicate.test(""));     //true
    }
```

### 3.Consumer


```
//Consumer 消费型接口    只有输入，没有返回值
    public static void ConsumerDmeo() {
        /*Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        } ;*/
        Consumer<String> consumer = s-> System.out.println(s);

        consumer.accept("afds");    //afds
    }
```

### 4.Supplier


```
//Supplier 供给型接口      没有参数，只有返回值
    public static void SupplierDemo() {
        /*Supplier supplier = new Supplier<Integer>() {
            @Override
            public Integer get() {
                return 1024;
            }
        } ;*/
        Supplier supplier = ()->{return 1024;} ;

        System.out.println(supplier.get());
    }
```


# Stream流式计算


```
//获取流方式一   .stream()
        List<String> list1 = new ArrayList<>();
        Stream<String> stream1 = list1.stream();

        Set<String> set = new HashSet<>();
        Stream<String> stream2 = set.stream();

        Map<String,String> map = new HashMap<>();
        Set<String> keySet = map.keySet();
        Stream<String> stream3 = keySet.stream();
        Collection<String> values = map.values();
        Stream<String> stream4 = values.stream();

        Set<Map.Entry<String, String>> entries = map.entrySet();
        Stream<Map.Entry<String, String>> stream5 = entries.stream();

        //获取流方式二   Stream.of()
        Stream<Integer> stream6 = Stream.of(1, 2, 3, 4, 5);
        Integer[] arr1 = {1,2,3,4} ;
        Stream<Integer> stream7 = Stream.of(arr1);
        String[] arr2 = {"1","22","333"};
        Stream<String> stream8 = Stream.of(arr2);

        //常用方法--forEach  参数是Consumer 消费型的函数式接口 【forEach是“终结方法”，不再返回流自身】
        //常用方法--filter  参数是Predicate 判断型的函数式接口
        ArrayList<String> list = new ArrayList<>();
        list.add("猪八戒");
        list.add("兔兔");
        list.add("猪霸天");
        list.add("猪猪");
        //使用内部迭代，过滤、遍历
        list.stream()
                .filter(name->name.startsWith("猪"))
                .filter(name->name.length()==3)
                .forEach(name-> System.out.println(name));

        //常用方法--map，参数是Function 转换型函数式接口
        Stream<String> stream9 = Stream.of("1", "22", "333");
        Stream<Integer> stream10 = stream9.map(s->{
            return Integer.parseInt(s);
        });
        stream10.forEach(i-> System.out.println(i ));

        //常用方法--limit，截取前n个,返回流自身
        //常用方法--count，返回long类型个数【终结方法】
        Stream<String> stream11 = Stream.of("1", "22", "333","4444","55555");
        System.out.println("count:"+stream11.limit(2).count()); //终结方法count，使用后流关闭，无法再用【一次性】

        //常用方法--skip，跳过前n个，返回流自身
        Stream<String> stream12 = Stream.of("1", "22", "333","4444","55555").skip(2);
        stream12.forEach(s-> System.out.println("skip:"+s));

        //常用方法--concat，静态方法，组合流
        Stream.concat(Stream.of("猪"),Stream.of("兔兔")).forEach(s -> System.out.print(s));
```


# ForkJoin

【JDK1.7】并行执行任务，提高效率。大数据量

大数据：Map Reduce（把大任务拆分成小任务）

![avatar](D:/其他/图片/study/ForkJoin.png)


ForkJoin特点：**工作窃取**

![avatar](D:/其他/图片/study/工作窃取.png)

> 这个里面维护的都是双端队列


**如何使用ForkJoin？**

- 通过ForkJoinPool来执行
- 计算：ForkJoinPool.execute(ForkJoinTask task)
- 任务：ForkJoinTask，用其子类 RecursiveTask（递归任务），实现抽象方法compute即可

> Recursive:递归

![avatar](D:/其他/图片/study/ForkJoin的使用.png)


**大数据量求和计算**
```
/**
 * 大数据量求和计算
 *   i、  低级：for
 *   ii、 中级：ForkJoin
 *   iii、高级：Stream并行流
 */
public class ForkJoinDemo extends RecursiveTask<Long> {
    private Long start ;
    private Long end ;

    private Long temp = 10000L ;    //临界值

    public ForkJoinDemo(Long start,Long end) {
        this.start = start ;
        this.end = end ;
    }

    //计算方法
    @Override
    protected Long compute() {
        if ((end-start)<temp) {
            Long sum = 0L ;
            for (Long i = start; i <= end; i++) {
                sum += i ;
            }
            return sum ;
        } else {    //递归
            Long middle = (start+end)/2 ;
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();   //拆分任务，把任务压入线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle + 1, end);
            task2.fork();   //拆分任务，把任务压入线程队列

            return task1.join() + task2.join() ;
        }
    }
}

//测试
class test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        test1();    //9225
//        test2();    //9159
        test3();       //3847
    }

    //低等
    public static void test1() {
        Long sum = 0L ;
        long start = System.currentTimeMillis() ;

        for (Long i = 1L; i <= 10_0000_0000L; i++) {
            sum += i ;
        }

        long end = System.currentTimeMillis();
        System.out.println("sum="+sum+" 时间="+(end-start));
    }

    //中等
    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis() ;

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L,10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);
        Long sum = submit.get();

        long end = System.currentTimeMillis();
        System.out.println("sum="+sum+" 时间="+(end-start));
    }

    //高级
    public static void test3() {
        long start = System.currentTimeMillis() ;

        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);

        long end = System.currentTimeMillis();
        System.out.println("sum="+sum+" 时间="+(end-start));
    }
}
```


# 异步回调

Future设计的初衷：对将来的某个事件的结果进行建模

- 异步调用：**juc.CompletableFuture**
  - runAsync    无返回值
  - supplyAsync     有返回值

异步执行

成功回调

失败回调


```
//没有返回值的异步回调
    public static void demo01() throws ExecutionException, InterruptedException {
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"--runAsync--Void");
        }) ;
        System.out.println(1111);
        completableFuture.get();    //获取阻塞执行结果
    }

    //有返回值的supplyAsync异步回调
    public static void demo02() throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"--supplyAsync--Integer");
//            int i = 10/0;
            return 1024;
        });
        System.out.println(completableFuture.whenComplete((t,u)->{  //回调成功
                    System.out.println("t-->"+t);   //正常的返回结果
                    System.out.println("u-->"+u);   //错误信息  u-->java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
            }).exceptionally((e)->{         //回调失败
                    System.out.println(e.getMessage());     //java.lang.ArithmeticException: / by zero
                    return 404 ;            //获取错误的返回结果
                }).get()
        );
    }
```



> JMM、volatile、单例模式、CAS  补充在JVM笔记


# 各种锁的理解

### 1.公平锁、非公平锁

### 2.可重入锁

![avatar](D:/其他/图片/study/可重入锁.png)

### 3.自旋锁

### 4.死锁

排查：

- 使用**jps -l**定位进程号
- 使用**jstack 进程号**找到死锁问题


排查问题：
- 日志
- 堆栈













