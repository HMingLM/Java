[toc]
## 多线程

![avatar](D:/其他/图片/study/线程状态.png)

#### 1.理解
- **进程**：是一个正在执行中的程序；
- 每一个进程执行都有一个执行顺序。该顺序是一个执行路径，或者叫一个**控制单元**；
- **线程**：就是进程中的一个独立的控制单元；线程控制着进程的执行；
- 一个进程中至少有一个线程；

> jvm启动的时候会有一个进程java.exe；
> 
> 该进程中至少一个线程负责java程序的执行，
> 而且这个线程运行的代码存在于main方法中，
> 该线程称之为**主线程**；

#### 2.线程的创建方式一
java提供了对线程这类事物的描述，即**Thread类**

- 创建线程的第一种方式：==继承Thread类==
- **步骤**：

> 定义类继承Thread--->复写Thread类中的run方法：此run方法用于存储线程运行的自定义代码
> 
> --->调用线程的start方法：作用为启动线程和调用run方法


```
class Test extends Thread
{
    Test(String name)
    {
        super(name);
    }
    public void run()
    {
        System.out.println("demo run1");
    }
}

class ThreadTest
{
    public static void main(String[] args)
    {
        Test t1 = new Test("one"); //线程名称
        Test t2 = new Test("two"); 
        t1.start();
        t2.start();
        System.out.println(this(相当于Thread.currentThread()).getName()+"run");
    }
}
```


#### 3.线程的创建方式二
- 创建线程的第二种方式：==实现Runnable接口==

> 步骤

- 定义实现Runnable接口
- 覆盖Runnable接口中的run方法：将线程要运行的代码存放在该run方法中
- 通过Thread类建立线程对象
- 将Runnable接口的子类对象作为实际参数传递给Thread类的构造函数：因自定义的run方法所属的对象是Runnable接口的子类对象，让线程去执行指定对象的run方法
- 调用Thread类的start方法开启线程并调用Runnable接口子类的run方法


```
class Ticket implements Runnale
{
    public void run(){}  
}

class TicketDemo
{
    Ticket t = new Ticket();
    Thread t1 = new Thread(t);
    t1.start();
}
```

#### 4.实现方式和继承方式的区别

- 继承Thread：线程代码存放在Thread子类run方法中；
- 实现Runnable：线程代码存放在接口的子类run方法中；
- 实现方式的好处：避免了单继承的局限性（建议使用）

#### 5.多线程的一个特性：随机性

某一时刻只能由一个程序在运行（多核除外），cpu做快速切换，以达到看上去是同时运行的效果；

- 获取线程对象以及名称

线程都有自己默认的名称 ：Thread-编号，该编号从0开始；

- static Thread currentThread():获取当前线程对象；
- getName()：获取线程名称；
- 设置线程名称：setName或者构造函数；

#### 6.多线程的安全问题

- 问题的原因：当多条语句在操作同一个线程共享数据时，一个线程对多条语句只执行了一部分，还没有执行完，另一线程参与进来，导致了共享数据的错误；
- 解决办法：对多条操作共享数据的语句，只能让一个线程都执行完，执行过程中其他线程不可以参与执行；
- Java对多线程安全问题提供解决方式：==同步代码块==；

#### 7.同步代码块


```
synchronized(对象)
{
    需要被同步的代码
}
```


对象如同锁，持有锁的线程可以在同步中执行；

- ==前提==：

> 必须要有两个或以上的线程；

> 必须是多个线程使用同一个锁；

- 好处：解决了多线程的安全问题；
- 弊端：多个线程需要判断锁，较为消耗资源；

#### 8.同步函数
同步代码块


```
Object obj = new Object();
public void add(int n)
{
    synchronized(obj){}
}
```


**即为同步函数**:
**public synchronized void add(int n){}**

- 同步函数使用的锁是this；
- 静态同步函数的锁是Class对象（是该方法所在类的字节码文件对象，即类名.class）；

#### 9.单例设计模式-懒汉式
懒汉式延迟加载，在多线程访问时产生安全隐患，加了同步导致低效，为此使用双重判断，注意此时该同步使用的锁；


```
class Single
{
    private static Single s = null;
    private Single(){}
    public static Single getInstance()
    {
        if(s==null)
        {
            synchronized(Single.class)
            {
                if(s==null)
                s = new Single();
            }
        }
    }
}
```


#### 10.死锁

死锁是这样一种情形：多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。

**死锁产生的四个必要条件**：

- **互斥使用**，即当资源被一个线程使用(占有)时，别的线程不能使用；
- **不可抢占**，资源请求者不能强制从资源占有者手中夺取资源，资源只能由资源占有者主动释放；
- **请求和保持**，即当资源请求者在请求其他的资源的同时保持对原有资源的占有；
- **循环等待**，即存在一个等待队列：P1占有P2的资源，P2占有P3的资源，P3占有P1的资源。这样就形成了一个等待环路。

当上述四个条件都成立的时候，便形成死锁。当然，死锁的情况下如果打破上述任何一个条件，便可让死锁消失。

#### 11.线程间通信

就是多个线程在操作同一资源，操作的动作不同；

**等待唤醒机制**
- 等待wait()；
- 唤醒notify()；
- 唤醒所有notifyAll();

都使用在同步中，因为要对持有监视器（锁）的线程操作；

- 使用这些方法时必须要标识所属的同步的锁，
**等待和唤醒必须是同一个锁**，而锁可以是任意对象，所以这些操作线程的方法（）定义在object类中；


```
public void run()
{
    while(true)
    {
        synchronized(r)
        {
            if(!r.flag)
            try{r.wait();}
            catch(Exception e){}
            System.out.println(r.name+r.sex);
            r.flag = flase;
            r.notifyAll();
        }
    }
}【优化前的代码】
```


- 生产者--->资源--->消费者

当生产者和消费者有多个时，要用**notifyAll()**：避免只唤醒本方线程导致程序中所有线程都等待的情况；
和**while（flag）**：让被唤醒的线程再一次判断标记;

#### 12.现实Lock代替同步synchronized
- Condition对象代替object中方法，该对象可通过Lock锁进行获取；
- await()---wait(),signal()---notify(),signalAll()---notifyAll();

> notifyAll和singleAll()唤醒了本方和对方；

**本方只唤醒对方：**

两个对象condition_pro和condition_con，唤醒时生产者唤醒消费者condition_con.signal();反之同理

**代码**


```
import java.util.concurrent.locks.*;

private Lock lock = new ReentrantLock();

private Condition condition_pro = lock.newCondition();

private Condition condition_con = lock.newCondition();

public void set (String name) throws InterruptedException
{
    lock.lock();
    try
    {
        while(flag)
        condition.await();
        this.name = name+"--"+couunt++;
        System.....
        flag=true;
        condition_con.signal();
    }
    finally{lock.unlock();}
}
```


#### 13.停止线程

- **定义循环结束标记**

因为线程运行代码一般都是循环，只要控制了循环，就可以让run方法结束，也就是线程结束；

**特殊情况**：当线程处于冻结状态，就不会读取标记，线程就不会结束；

- **使用interrupt(中断)方法**

当没有指定的方式让冻结状态的线程恢复到运行状态时，该方法结束线程的冻结状态，使线程回到运行状态中来；

- 注：stop方法已过时不再使用；

#### 14.守护线程
**setDemon()**;将该线程标记为守护线程或用户线程；该方法必须在启动线程前调用；  

#### 15.join方法
当A线程执行到了B线程的.join()
方法时，A就会等待；等B线程都执行完，A才会执行；

join可以用来临时加入线程执行；

**使用时需要抛出异常；**

- setPriority(Thread.MAX_PRIORITY);设置优先级（最高）
- yield()：暂停当前正在执行的线程对象，并执行其他线程；

#### 16.比较wait(),sleep(),yield()

- **wait**()方法则是指当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行;
- **sleep**()方法只让出了CPU，而并不会释放同步资源锁；可以使低优先级的线程得到执行的机会；  
- **yield**()：暂停当前正在执行的线程对象(不能由用户指定暂停多长时间)，并执行其他线程；只能让同优先级的线程有执行的机会。 
