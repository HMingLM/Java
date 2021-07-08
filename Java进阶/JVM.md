[toc]
# JVM： java virtual Machine

jdk中包含了jvm和“屏蔽操作系统差异的组件”

- jvm各个操作系统之上是一致的
- “屏蔽操作系统差异的组件：在各个PC上各不相同（联想下载jdk，不同系统 需要下载不同版本的jdk）

![avatar](D:/学习/java/JVM/笔记/图片/jdk和jvm.png)


**JVM是面向操作系统的，负责把class字节码解释成系统所能识别的指令并执行，同时也负责程序运行时内存的管理。**

## 源码文件(.java)到代码执行

**编译 -- 加载 -- 解释 -- 执行**

- **编译**：将源码文件编译成JVM可以解释的class文件。

> 会对源代码作“语法分析”“语义分析”“注解处理”等。

- **加载**：把编译后的class文件加载到JVM中。

- **解释**：把字节码转换为操作系统可以识别的指令

- **执行**：操作系统调用系统的硬件执行最终的解释器解析出来的程序指令。


---

**加载阶段**细化三个步骤：**装载、连接、初始化**
- **装载**：查找并加载类的二进制数据，在JVM（堆）中创建一个java.lang.Class类的对象，并将类相关的数据存储在JVM（方法区）中。
    - **装载时机**：有需要时才进行装载（new、反射等），而非一次性装载所有类至JVM，节省了内存开销
    - **装载发生**：通过**类加载器**装载到JVM，为防止内存中出现多份同样的字节码，使用了**双亲委派机制**（不会自己尝试加载这个类，而是把请求委托给父加载器去完成，依次向上）
    - **装载规则**：JDK中的本地方法一般由**根加载器**装载，JDK中内部实现的扩展类一般由**扩展加载器**实现装载，而程序中的类文件则由**系统加载器**实现装载
- **连接**：对class的信息进行验证，为类变量分配内存空间并对其赋默认值
    - 连接细化三个步骤：**验证、准备、解析**
    - **验证**：验证类是否符合Java规范和JVM规范
    - **准备**：为类的静态变量分配内存，初始化为系统的初始值
    - **解析**：将符号引用转为直接引用的过程
- **初始化**：为类的静态变量赋予正确的初始值
    - 包括静态变量、静态代码块、静态方法
    - 如果是实例化对象，则会调用方法对实例变量进行初始化，并执行构造方法内的代码

---


- **解释阶段**有两种方式把字节码信息解释成机器指令码：**字节码解释器**、**即时编译器（JIT）**。JVM用**热点探测**来检测是否**热点代码**，非热点代码直接解释。
    - 热点探测两种方式：**计数器**、**抽样**。
    - HotSpot使用的是计数器的方式进行探测，为每个方法准备两类计数器：**方法调用计数器**、**回边计数器**。
    - 计数器都有一个确定的阈值，当计数器超过阈值，判断这部分代码执行频繁，为热点代码，触发JIT编译。
    - 即时编译器把热点代码的指令码保存起来，下次执行**无需重复解释**，直接执行缓存的机器语言。


    



## 类的生命周期

生命周期： 类的加载->连接->初始化->使用->卸载

-  类的加载

  查找并加载类的二进制数据（class文件）

  硬盘上的class文件 加载到jvm内存中

- 连接 ：确定类与类之间的关系  ； student.setAddress( address ); 

  - 验证

    .class 正确性校验

  - 准备

    static静态变量分配内存，并赋初始化默认值

    static int num =  10 ;  在准备阶段，会把num=0，之后（初始化阶段）再将0修改为10

    在准备阶段，JVM中只有类，没有对象。

    初始化顺序： static ->非static ->构造方法

    public class Student{

    ​	static int age ; //在准备阶段，将age = 0 ;

    ​	String name ; 

    }

    
  - 解析:把类中符号引用，转为直接引用

    前期阶段，还不知道类的具体内存地址，只能使用“com.yanqun.pojo.Student ”来替代Student类，“com.yanqun.pojo.Student ”就称为符号引用；

    在解析阶段，JVM就可以将 “com.yanqun.pojo.Student ”映射成实际的内存地址，会后就用 内存地址来代替Student，这种使用 内存地址来使用 类的方法 称为直接引用。

 - 初始化：给static变量 赋予正确的值

static int num =  10 ;  在连接的准备阶段，会把num=0，之后（初始化阶段）再将0修改为10

 - 使用： 对象的初始化、对象的垃圾回收、对象的销毁
 - 卸载



jvm结束生命周期的时机：

- 正常结束
- 异常结束/错误  
- System.exit()
- 操作系统异常



## JVM内存模型（Java Memoery Model，简称JMM）

JMM:用于定义（所有线程的共享变量， 不能是局部变量）变量的访问规则

JMM将内存划分为两个区： 主内存区、工作内存区

- 主内存区 ：真实存放变量
- 工作内存区：主内存中变量的副本，供各个线程所使用



注意：1.各个线程只能访问自己私有的工作内存（不能访问其他线程的工作内存，也不能访问主内存）

2.不同线程之间，可以通过主内存间接地访问其他线程的工作内存



完整的研究：不同线程之间交互数据时 经历的步骤：

1.Lock：将主内存中的变量，表示为一条线程的独占状态

2.Read：将主内存中的变量，读取到工作内存中

3.Load：将2中读取的变量拷贝到变量副本中

4.Use：把工作内存中的变量副本，传递给线程去使用

5.Assign:把线程正在使用的变量，传递给工作内存中的变量副本中

6.Store:将工作内存中变量副本的值，传递到主内存中

7.Write：将变量副本作为一个主内存中的变量进行存储

8.Unlock:解决线程的独占状态


![avatar](D:/学习/java/JVM/笔记/图片/jvm内存模型.png)

![avatar](D:/其他/图片/study/狂神JMM理解.png)

JVM要求以上的8个动作必须是原子性的;

但是对于64位的数据类型（long/ double）有些非原子性协议。

- 说明什么问题：
  - 在执行以上8个操作时，可能会出现 只读取（写入等）了半个long/double数据，因此出现错误。

- 如何避免？ 
  - 1.商用JVM已经充分考虑了此问题，无需我们操作
  - 2.可以通过volatile避免此类问题（读取半个数据的问题）   volatile double num ;

- 关于JMM的一些同步的约定
  - 线程解锁前，必须把工作内存中的变量**立刻**刷回主存
  - 线程加锁前，必须读取主存中的最新值到工作内存中
  - 加锁和解锁是同一把锁

![avatar](D:/其他/图片/study/JMM存在的问题.png)

## volatile 

概念：JVM提供的一个**轻量级同步机制**

作用：

1.防止JVM对long/double等64位的非原子性协议进行的误操作（读取半个数据）

2.**内存可见性**，可以使变量对所有的线程立即可见（某一个线程如果修改了 工作内存中的变量副本，若是被volatile修饰 ，则该变量就会【立刻】同步到其他线程的工作内存中）

3.禁止指令的“重排序”优化

**总结**

- ① **保证可见性**
- ② **不保证原子性**
- ③ **禁止指令重排**



原子性 ： num = 10 ;

非原子性： int num = 10 ; ->  int num ;   num =10 ;


**重排序**

源代码-->编译器优化的重排-->指令并行也可能重排-->内存系统也会重排-->执行

排序的对象就是 原子性操作，目的是为了提高执行效率，优化

处理器在进行指令重排时，考虑数据之间的依赖性

```java
int a  =10 ; //1    int a ; a = 10 ;
int b ;//2
b = 20 ;//3
int c = a * b ;//4
```

重排序“不会影响**单线程**的执行结果”，因此以上程序在经过重排序后，可能的执行结果：1,2,3,4 ；2,3,1,4

```java
//2 3 1 4
int b ;
b = 20 ;
int a  =10 ;
int c = a * b ;
```

懒汉式
```java
package com.yanqun;
//双重检查式的懒汉式单例模式【DCL懒汉式】
public class Singleton {
    private static Singleton instance = null ;//单例
    private Singleton(){}   
    public static Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class){
                if(instance == null){
                    instance = new Singleton() ;//不是一个原子性操作
                }
            }
        }
        return instance ;
    }
}
```


以上代码可能会出现问题，原因 instance = new Singleton() 不是一个原子性操作，会在执行时拆分成以下动作：

1.JVM会分配内存地址、内存空间

2.使用构造方法实例化对象

3.instance = 第1步分配好的内存地址

根据重排序的知识，可知，以上3个动作在真正执行时 可能1、2、3，也可能是1、3、2【单线程下结果一致】

如果在多线程环境下，使用1、3、2可能出现问题：

假设线程A刚刚执行完以下步骤（即刚执行 1、3，但还没有执行2）

1正常0x123 ,  ...

3instance=0x123

此时，线程B进入单例程序的if，直接会得到Instance对象（注意，此instance是刚才线程A并没有new的对象）,就去使用该对象，例如instance.xxx() 则必然报错。解决方案，就是 禁止此程序使用1 3 2 的重排序顺序。解决：

```
  private volatile static Singleton instance = null ;//单例
```

volatile是通过“内存屏障”防止重排序问题：

1.在volatile写操作前，插入StoreStore屏障

2.在volatile写操作后，插入StoreLoad屏障

3.在volatile读操作后，插入LoadLoad屏障

4.在volatile读操作后，插入LoadStore屏障



- ***volatile***是否能保证原子性、保证线程安全？不能！
  - 要想保证原子性/线程安全，可以使用原子类，**java.util.cocurrent.aotmic**中的类，
  - 能够保证原子性的核心，底层都直接和操作系统挂钩，在内存中修改值【unsafe类】
  - 提供了compareAndSet()方法【CAS的由来】， cas算法（无锁算法）。

> CAS算法保证了原子性


### 单例模式

思想：构造器私有，无法new实例，保证内存中只有一个对象


懒汉式
```java
package com.yanqun;
//双重检查式的懒汉式单例模式【DCL懒汉式】
public class Singleton {
    private static Singleton instance = null ;//单例
    private Singleton(){}   
    public static Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class){
                if(instance == null){
                    instance = new Singleton() ;//不是一个原子性操作
                }
            }
        }
        return instance ;
    }
}
```

饿汉式
```
class Hungry{
    private Hungry(){ }     //构造器私有

    private final static Hungry HUNGRY  = new Hungry();

    public static Hungry getInstance(){
        return HUNGRY;
    }
}
```

静态内部类

```
class Holder{
    private Holder(){}

    public static Holder getInstance() {
        return InnerClass.HOLDER;
    }

    public static class InnerClass{
        private static final Holder HOLDER = new Holder();
    }
}
```

- 以上所写单例模式都可能会被反射破坏，想办法解决，但，道高一尺魔高一丈，始终不安全
  - 解决：用枚举类

**enum**

**枚举本身也是一个class类**：class xx extends enum

```
public enum EnumSigle {
    INSTANCE;
    public EnumSigle getInstance(){
        return INSTANCE;
    }
}
```
安全，但无法延迟加载


## CAS算法

CAS：比较当前工作内存中的值和主内存中的值，如果这个值是期望值，则执行操作，否则就一直循环

- 缺点：
  - 循环会耗时
  - 一次性只能保证一个共享变量的原子性
  - ABA问题

![avatar](D:/其他/图片/study/Java后门unsafe类.png)

![avatar](D:/其他/图片/study/unsafe类的getAndAddInt方法.png)

![avatar](D:/其他/图片/study/自旋锁.png)

ABA问题：某个线程对共享变量做了两次修改，第二次修改将其改回原先的值，对另一线程来说该变量没有发生改变，这是存在问题的【乐观锁】

- 原子引用解决ABA问题，AtomicStampedReference


```
//AtomicInteger实现CAS
    public static void Demo01() {
        AtomicInteger atomicInteger = new AtomicInteger(2020);

        System.out.println(atomicInteger.compareAndSet(2020,2021));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2020,2021));
        System.out.println(atomicInteger.get());
    }

    // 原子引用解决ABA问题
    public static void Demo02() {
        //初始值1，初始版本1
        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1, 1);
        new Thread(()->{
            System.out.println("线程A....当前值-"+atomicStampedReference.getReference()+",版本-"+atomicStampedReference.getStamp());

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //将1赋值为2，版本+1【乐观锁】
            System.out.println("线程A....修改成功？ "+atomicStampedReference.compareAndSet(1, 2,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));
            System.out.println("线程A....当前值-"+atomicStampedReference.getReference()+",版本-"+atomicStampedReference.getStamp());

            //将2赋值为1，版本+1
            System.out.println("线程A....修改成功？ "+atomicStampedReference.compareAndSet(2, 1,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));
            System.out.println("线程A....当前值-"+atomicStampedReference.getReference()+",版本-"+atomicStampedReference.getStamp());
        },"A").start();

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();
            System.out.println("线程B....当前值-"+atomicStampedReference.getReference()+",版本-"+stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //将1赋值为2，版本+1【乐观锁】
            System.out.println("线程B....修改成功？ "+atomicStampedReference.compareAndSet(1, 2,
                    stamp, stamp + 1));
            System.out.println("线程B....当前值-"+atomicStampedReference.getReference()+",版本-"+atomicStampedReference.getStamp());
        },"B").start();
    }
```


注意：

> **Integer使用了对象缓存机制，默认范围是-128~127，推荐使用静态工厂方法valueOf获取对象实例，而不是new，因为valueOf使用缓存，而new一定会创建新的对象分配新的内存空间**

> 阿里开发手册：所有相同类型的包装类对象之间值的比较，全部使用equals方法。
> 
> 对于-128~127之间的赋值，Integer对象事在IntegerCahce.cache产生，会服用已有对象，这个区间可以直接使用==进行判断，但这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象。



CAS算法是计算机硬件对并发操作共享数据的支持，CAS包含3个操作数：

内存值V

预估值A

更新值B

当且仅当V==A时，才会把B的值赋给V，即V = B，否则不做任何操作。

就i++问题【非原子性操作，可能出现多线程安全问题】，CAS算法是这样处理的：

首先V是主存中的值0，然后预估值A也是0，因为此时还没有任何操作，这时V=B，所以进行自增，同时把主存中的值变为1。如果第二个线程读取到主存中的还是0也没关系，因为此时预估值已经变成1，V不等于A，所以不进行任何操作。


## JVM运行时的内存区域

将JVM在运行时的内存，划分为了5个部分，如图所示。

![avatar](D:/学习/java/JVM/笔记/图片/jvm运行时的内存区域.png)

## 程序计数器

程序计数器：行号指示器，指向当前线程所执行的字节码指令的地址

Test.java -> Test.class

```java
int num = 1;   //1
int num2 = 2 ; //2
if(num1>num2){//3
...//4-10
}else //11
{
    ...
}
while(...)
{
    
}
```

简单的可以理解为：class文件中的行号

> 注意：
>
> 1.一般情况下，程序计数器 是行号；但如果正在执行的方法是native方法，则程序计数器的值 undefined。
>
> 2.程序计数器 是唯一一个 不会 产生 “内存溢出”的区域。

goto的本质就是改变的 程序计数器的值（java中没有goto，goto是java中唯一的保留字）



## 虚拟机栈

定义：描述 方法执行的内存模型

- 方法在执行的同时，会在虚拟机栈中创建一个栈帧
- 栈帧中包含：方法的局部变量表，操作数据栈、动态链接、方法出口信息等



当方法太多时，就可能发生 栈溢出异常StackOverflowError，或者内存溢出异常OutOfMemoryError

```java
public static void main(String[] args) {
    main(new String[]{"abc","abc"});
}
```

## 本地方法栈

原理和结构与虚拟机栈一致，不同点： 虚拟机栈中存放的 jdk或我们自己编写的方法，而本地方法栈调用的 操作系统底层的方法。

JVM中有本地方法接口【JNI】，调用本地方法库【C语言】

![avatar](D:/学习/java/JVM/笔记/图片/虚拟机栈.png)

![avatar](D:/学习/java/JVM/笔记/图片/栈帧-局部变量表+操作数据栈.png)

![avatar](D:/学习/java/JVM/笔记/图片/栈帧-动态链接.png)

## 堆

![avatar](D:/学习/java/JVM/笔记/图片/Heap.png)

- 存放对象实例（数组、对象）

- 堆是jvm区域中最大的一块，在jvm启动时就已经创建完毕

- GC主要管理的区域

- 堆本身是线程共享，但在堆内部可以划分出多个线程私有的缓冲区

- 堆允许物理空间不连续，只要逻辑连续即可

- 堆可以分 新生代、老生代 。大小比例，新生代：老生代= 1:2  

- 新生代中 包含eden、s0、s1 = 8:1:1  

- 新生代的使用率一般在90%。 在使用时，只能使用 一个eden和一块s区间(s0或s1)

- 新生代：存放 1.生命周期比较短的对象  2.小的对象；反之，存放在老生代中。对象的大小，可以通过参数设置 -XX：PretenureSizeThredshold 。一般而言，大对象一般是 集合、数组、字符串。生命周期： -XX:MaxTenuringThredshold

  新生代、老生代中年龄：MinorGC回收新生代中的对象。如果Eden区中的对象在一次回收后仍然存活，就会被转移到 s区中；之后，如果MinorGC再次回收，已经在s区中的对象仍然存活，则年龄+1。如果年龄增长一定的数字，则对象会被转移到 老生代中。简言之：在新生代中的对象，每经过一次MinorGC，有三种可能：1从eden -》s区   2.（已经在s区中）年龄+1  3.转移到老生代中

   ![avatar](D:/学习/java/JVM/笔记/图片/新生代老生代.png)

  新生代在使用时，只能同时使用一个s区：底层采用的是复制算法，为了避免碎片产生，弊端是浪费了内存空间
  【另有标记清除压缩算法】

  

  老生代： 1.生命周期比较长的对象  2.大的对象； 使用的回收器 MajorGC\FullGC

  

  新生代特点： 

  - 大部分对象都存在于新生代
  - 新生代的回收频率高、效率高

  老生代特点：

  - 空间大、
  - 增长速度慢
  - 频率低

  意义：可以根据项目中 对象大小的数量，设置新生代或老生代的空间容量，从提高GC的性能。

  

如果对象太多，也可能导致内存异常。



虚拟机参数：

-Xms128m ：JVM启动时的大小

 -Xmn32m：新生代大小

 -Xmx128：总大小

jvm总大小= 新生代 + 老生代



堆内存溢出的示例：java.lang.OutOfMemoryError: Java heap space

```java
package com.yanqun;

import java.util.ArrayList;
import java.util.List;

public class TestHeap {
    public static void main(String[] args) {

        List list = new ArrayList() ;
        while(true){
            list.add(  new int[1024*1024]) ;
        }

    }
}

```



## 方法区

存放：类的元数据（描述类的信息）、常量池、方法信息（方法数据【静态变量】、方法代码）

gc：类的元数据（描述类的信息）、常量池

方法区中数据如果太多，也会抛异常OutOfMemory异常



图：方法区与其他区域的调用关系

   ![avatar](D:/学习/java/JVM/笔记/图片/方法区.png)

常量池：存放编译期间产生的 字面量("abc")、符号引用



注意： 导致内存溢出的异常OutOfMemoryError，除了虚拟机中的4个区域以外，还可能是直接内存。在NIO技术中会使用到直接内存。



## 类的使用方式

类的初始化：JVM只会在**“首次主动使用”**一个类/接口时，才会初始化它们 。

## 主动使用

1.new 构造类的使用 

```java
package init;

public class Test1 {

    static{
        System.out.println("Test1...");
    }

    public static void main(String[] args) {
        new Test1();//首次主动使用
        new Test1();
    }
}

```

结果：Test1...



2.访问类/接口的 静态成员（属性、方法）

```java
package init;
class A{
    static int i  = 10;

    static{
        System.out.println("A...");
    }

     static void method(){
        System.out.println("A method...");
    }
}

public class Test2 {
    public static void main(String[] args) {

//        A.i = 1 ;
//        A.i = 1 ;
//        System.out.println(A.i);
        A.method();

    }
}

```

注：main()本身也是一个静态方法，也此main()的所在类 也会在执行被初始化

特殊情况：

- 如果成员变量既是static，又是final ，即常量，则不会被初始化
- 上一种情况中，如果常量的值 是一个随机值，则会被初始化 (为了安全)



3. 使用Class.forName("init.B")执行反射时使用的类（B类）
4. 初始化一个子类时，该子类的父类也会被初始化

```java
public class Son extends  Father {
    public static void main(String[] args) {
        new Son();
    }
}

```

5.动态语言在执行所涉及的类 也会被初始化（动态代理）



## 被动使用

除了主动以外，其他都是被动使用。

```java
package init;
class BD
{
    static {
        System.out.println("BD...");
    }
}

public class BeiDong {
    public static void main(String[] args) {
        BD[] bds = new BD[3];

    }
}

```

以上代码，不属于主动使用类，因此不会被初始化。



## 助记符



反编译： cd到class目录中， javap -c class文件名





Test.java  ->                    javap -c Test.java

javap反编译的是class文件

应该：xx.java -> xx.class ->javap



aload_0: 装载了一个引用类型

 Invokespecial:  init,  private  , super.method() :  \<init\>存放的是初始化代码的位置

getstatic ：获取静态成员

bipush ： 整数范围 -128  -- 127之内  (8位带符号的整数),放到栈顶

sipush:    >127   (16个带符号的整数),放到栈顶

注意：无论是定义int或short 等，只要在 -128 --127以内 都是bipush，否则是sipush.

注意：特殊：-1  -- 5不是bipush

​		 iconst_m1（-1）  iconst_0   iconst_1 ....  iconst_5



ldc  : int  float String 常量 ,放到栈顶

ldc2_w :long  double常量,放到栈顶



## JVM四种引用级别

如果一个对象存在着指向它的引用，那么这个对象就不会被GC回收？  -- 局限性

Object obj = new Object() ; --强引用



根据引用的强弱关系： 强引用>软引用>弱引用>虚引用



### 强引用

Object obj = new Object() ;

约定： 引用 obj，引用对象new Object()

强引用对象什么失效？

1.生命周期结束（作用域失效）

```java
public void method(){
  
  Object obj = new Object() ;
}//当方法执行完毕后，强引用指向的 引用而对象new Object()就会等待被GC回收
```

2.引用被置为null，引用对象被GC回收

```
obj = null ;//此时，没有任何引用指向new Object() 因此，new Object() 就会等待被GC回收
```

除了以上两个情况以外，其他任何时候GC都不会回收强引用对象。

### 软引用

根据JVM内存情况： 如果内存充足，GC不会随便的回收软引用对象；如果JVM内存不足，则GC就会主动的回收软引用对象。



各种引用的出处：

强引用:new

软引用 弱引用 虚引用 （最终引用）：Reference

软引用：java.lang.ref.SoftReference

Reference中有一个get()方法，用于返回 所引用的对象

```
SoftReference<SoftObject> softRef = new SoftReference<>(new SoftObject() );
```

softRef.get()  -->返回引用所指向的SoftObject对象本身

```java
package ref;

import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;



//软引用对象
class SoftObject{}
public class SoftReferenceDemo {
    public static void main(String[] args) throws Exception {
        //softRef  -->SoftObject  设计模式中的：装饰模式
        SoftReference<SoftObject> softRef = new SoftReference<>(new SoftObject() );
        List<byte[]> list = new ArrayList<>();

        //开启一个线程，监听 是否有软引用已经被回收
        new Thread(  ()->{
        while(true) {
            if (softRef.get() == null) //软引用对象
            {
                System.out.println("软引用对象已被回收..");
                System.exit(0);
            }
        }
        }  ,"线程A" ) .start();     //lambda


        //不断的往集合中 存放数据，模拟内存不足
        while(true){
//          Thread.sleep(10);
            if(softRef.get() != null)
                list.add(new byte[1024*1024]) ;//每次向list中增加1m内容
        }


    }
}

```



### 弱引用

回收的时机：只要GC执行，就会将弱引用对象进行回收。

java.lang.ref.WeakReference\<T\>

```java
package ref;

import java.lang.ref.WeakReference;

public class WeakReferenceDemo {
    public static void main(String[] args) throws Exception {

        WeakReference<Object> weakRef = new WeakReference<>(new Object());
        //weakRef->Object

        System.out.println( weakRef.get()==null   ? "已被回收":"没被回收"  );

        System.gc();//建议GC执行一次回收（存在概率）
        Thread.sleep(100);

        System.out.println( weakRef.get()==null   ? "已被回收":"没被回收"  );

    }
}


```

### 虚引用（幻影引用或者幽灵引用）

java.lang.ref.PhantomReference\<T\>

是否使用虚引用，和引用对象本身 没有任何关系； 无法通过虚引用来获取对象本身.

引用get() -> 引用对象

虚引用get() -> null

虚引用不会单独使用，一般会和 引用队列（java.lang.ref.ReferenceQueue）一起使用。

价值： 当gc回收一个对象，如果gc发现 此对象还有一个虚引用，就会将虚引用放入到 引用队列中，之后（当虚引用出队之后）再去回收该对象。因此，我们可以使用 虚引用+引用对象 实现：在对象被gc之前，进行一些额外的其他操作。

 GC ->如果有虚引用->虚引用入队->虚引用出队-> 回收对象

```java
package ref;

import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

class MyObject {

}

public class PhantomReferenceDemo {


    public static void main(String[] args) throws Exception {
        MyObject obj = new MyObject();
        //引用队列
        ReferenceQueue queue = new ReferenceQueue();

        //虚引用+引用队列
        PhantomReference<MyObject> phantomRef = new PhantomReference<>(obj, queue);

        //让gc执行一次回收操作
        obj = null;
        System.gc();
        Thread.sleep(30);
        System.out.println("GC执行...");

        //GC-> 虚引用->入队->出队->     obj
        System.out.println(queue.poll());


    }
}


```

特殊情况：如果虚引用对象重写了finalize()，那么JVM会延迟 虚引用的入队时间。

```java
package ref;

import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

class MyObject3 {
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("即将被回收之前...");
    }
}

public class PhantomReferenceDemo2 {


    public static void main(String[] args) throws Exception {
        MyObject3 obj = new MyObject3();
        //引用队列
        ReferenceQueue queue = new ReferenceQueue();

        //虚引用+引用队列
        PhantomReference<MyObject3> phantomRef = new PhantomReference<>(obj, queue);

        //让gc执行一次回收操作
        obj = null;

        System.gc();
//        Thread.sleep(30);
        System.out.println("GC执行...");

        //GC-> 虚引用->入队->出队->     obj
        System.out.println(queue.poll());//虚引用并没有入队

        System.gc();
//        Thread.sleep(30);
        System.out.println("GC执行...");

        //GC-> 虚引用->入队->出队->     obj
        System.out.println(queue.poll());//虚引用延迟到了第二次gc时入队

        System.gc();
//        Thread.sleep(30);
        System.out.println("GC执行...");

        //GC-> 虚引用->入队->出队->     obj
        System.out.println(queue.poll());

        System.gc();
        Thread.sleep(30);
        System.out.println("GC执行...");

        //GC-> 虚引用->入队->出队->     obj
        System.out.println(queue.poll());


    }
}

```



final class Finalizer extends FinalReference：最终引用

构造方法()  -> 析构函数()，在java中存在Finalizer  可以帮我们自动的回收一些不需要的对象，因此不需要写析构函数。   

jvm能够直接操作的是：非直接内存 

直接内存：native （操作系统中的内存，而不是jvm内存）

jvm不能操作 直接内存（非jvm操作的内容）时，而恰好 此区域的内容 又忘了关闭，此时Finalizer就会将这些内存进行回收。



### 使用软引用实现缓存的淘汰策略

java ->缓存( 90% ->60%)  ->   db(iphone)

LRU

一般的淘汰策略：

根据容量/缓存个数 + LRU 进行淘汰。

在java中 还可以用引用实现 淘汰策略。

MyObject obj = new MyObject();//强引用，不会被GC回收

map.put( id   ,   obj ) ;



Map.put(id,  软引用(obj) )；//当jvm内存不足时，会主动回收。



```java
package ref;

import java.lang.ref.SoftReference;
import java.util.HashMap;
import java.util.Map;

class MyObject10{}

public class SoftDemo {

    //map: key:id  ,value:对象的软引用  （拿对象： 对象的软引用 .get() ）
    Map<String, SoftReference<MyObject10>> caches = new HashMap<>();
    //java -> caches -> db
    //set: db->caches
    //get: java->cache


    void setCaches(String id,MyObject10 obj){
        caches.put( id,   new SoftReference<MyObject10>(obj) );

    }

    MyObject10 getCache(String id){
        SoftReference<MyObject10> softRef = caches.get(id) ;
       return  softRef == null ?  null : softRef.get()  ;
    }
    //优势：当jvm内存不足时，gc会自动回收软引用。因此本程序 无需考虑 OOM问题。



}

```



## 双亲委派

前置：类的加载

```java
package com.yanqun.parents;
class MyClass{
    static int num1 = 100 ;

    static MyClass myClass = new MyClass();
    public MyClass(){
        num1 = 200 ;
        num2 = 200 ;
    }
    static int num2 = 100 ;
    public static MyClass getMyClass(){
        return myClass ;
    }

    @Override
    public String toString() {
        return this.num1 + "\t" + this.num2 ;
    }
}


public class MyClassLoader {
    public static void main(String[] args) {
        MyClass myc =  MyClass.getMyClass() ;
        System.out.println(myc);

    }
}

```



分析

```java
    static int num1 = 100 ;     【 0 】-> 【100】->【200】

    static MyClass myClass = new MyClass();【null】 ->【引用地址0x112231】
    public MyClass(){
        num1 = 200 ;
        num2 = 200 ;
    }
    static int num2 = 100 ;  【0】->【200】->【100】

连接：static静态变量并赋默认值

初始化：给static变量 赋予正确的值
```

总结：在类中 给静态变量的初始化值问题，一定要注意顺序问题（静态变量 和构造方法的顺序问题）

 

双亲委派： JVM自带的加载器（在JVM的内部所包含，C++）、用户自定义的加载器（独立于JVM之外的加载器,Java）



-  JVM自带的加载器

  - 根加载器,Bootstrap   : 加载 jre\lib\rt.jar （包含了平时编写代码时 大部分jdk api）；指定加载某一个jar（ -Xbootclasspath=a.jar）
  - 扩展类加载器，Extension：C:\Java\jdk1.8.0_101\jre\lib\ext\\\*.jar ;指定加载某一个jar(-Djava.ext.dirs= ....)
  - AppClassLoader/SystemClassLoader，系统加载器（应用加载器）：加载classpath；指定加载（-Djava.class.path= 类/jar）

- 用户自定义的加载器

  - 都是抽象类java.lang.ClassLoader的子类

  

![avatar](D:/学习/java/JVM/笔记/图片/双亲委派.png)

双亲委派：当一个加载器要加载类的时候，自己先不加载，而是逐层向上交由双亲去加载；当双亲中的某一个加载器 加载成功后，再向下返回成功。如果所有的双亲和自己都无法加载，则报异常。



```java
package com.yanqun.parents;
//classpath: .; ..lib，其中“.”代表当前（自己写的类）
class MyClass2{
}

public class TestParentsClassLoader {


    public static void main(String[] args) throws Exception {
       Class myClass1 =  Class.forName("java.lang.Math") ;
        ClassLoader classLoader1 = myClass1.getClassLoader();
        System.out.println(classLoader1);
        /* JDK中的官方说明：
            Some implementations may use null to represent the bootstrap class loader
         */
       Class myClass2 =  Class.forName("com.yanqun.parents.MyClass2") ;
        ClassLoader classLoader2 = myClass2.getClassLoader();
        System.out.println(classLoader2);
    }
}

```

![avatar](D:/学习/java/JVM/笔记/图片/1571542429810.png)

null: bootstrap class loader



小结：如果类是 rt.jar中的，则该类是被 bootstrap（根加载器）加载；如果是classpath中的类（自己编写的类），则该类是被AppClassLoader加载。



定义类加载：最终实际加载类的 加载器  

初始化类加载类：直接面对加载任务的类





```java
package com.yanqun.parents;

import java.net.URL;
import java.util.Enumeration;

class MyCL{

}
public class JVMParentsCL {
    public static void main(String[] args) throws Exception {
        Class<?> myCL = Class.forName("com.yanqun.parents.MyCL");
        ClassLoader classLoader = myCL.getClassLoader();
        System.out.println(classLoader);
        System.out.println("---");
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);

        ClassLoader parent1 = systemClassLoader.getParent();
        System.out.println(parent1);
        ClassLoader parent2 = parent1.getParent();
        System.out.println(parent2);

        System.out.println("----");

        ClassLoader appClassLoader = ClassLoader.getSystemClassLoader();
        Enumeration<URL> resources = appClassLoader.getResources("com/yanqun/parents/MyCL.class");// a/b/c.txt
        while(resources.hasMoreElements()){
            URL url = resources.nextElement();
            System.out.println(url);
        }

    }
}

```

![avatar](D:/学习/java/JVM/笔记/图片/1571556896861.png)


### 自定义类的加载器

二进制名binary names:

```
   "java.lang.String"
   "javax.swing.JSpinner$DefaultEditor"
   "java.security.KeyStore$Builder$FileBuilder$1"
   "java.net.URLClassLoader$3$1"
 
```

$代表内部类：

$数字：第几个匿名内部类

```
The class loader for an array class, as returned by {@link* Class#getClassLoader()} is the same as the class loader for its element* type; if the element type is a primitive type, then the array class has no* class loader.
```

1.数组的加载器类型  和数组元素的加载器类型 是相同

2.原声类型的数组 是没有类加载器的  

如果加载的结果是null：  可能是此类没有加载器(int[]) ， 也可能是 加载类型是“根加载器”

```
<p> However, some classes may not originate from a file; they may originate* from other sources, such as the network, or they could be constructed by an* application.  The method {@link #defineClass(String, byte[], int, int)* <tt>defineClass</tt>} converts an array of bytes into an instance of class* <tt>Class</tt>. Instances of this newly defined class can be created using* {@link Class#newInstance <tt>Class.newInstance</tt>}.
```

xxx.class文件可能是在本地存在，也可能是来自于网络 或者在运行时动态产生(jsp)

```java
<p> The network class loader subclass must define the methods {@link
 * #findClass <tt>findClass</tt>} and <tt>loadClassData</tt> to load a class
 * from the network.  Once it has downloaded the bytes that make up the class,
 * it should use the method {@link #defineClass <tt>defineClass</tt>} to
 * create a class instance.  A sample implementation is:
 *
 * <blockquote><pre>
 *     class NetworkClassLoader extends ClassLoader {
 *         String host;
 *         int port;
 *
 *         public Class findClass(String name) {
 *             byte[] b = loadClassData(name);
 *             return defineClass(name, b, 0, b.length);
 *         }
 *
 *         private byte[] loadClassData(String name) {
 *             // load the class data from the connection
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
```

如果class文件来自原Network，则加载器中必须重写findClas()和loadClassData().



自定义类加载器的实现

重写findClas()和loadClassData()

```java
package com.yanqun.parents;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

//public class MyException extends Exception{...}
public class MyClassLoaderImpl  extends ClassLoader{
        //优先使用的类加载器是：getSystemClassLoader()
        public MyClassLoaderImpl(){
            super();
        }

        public MyClassLoaderImpl(ClassLoader parent){//扩展类加载器
            super(parent);
        }
        //com.yq.xx.class
        public Class findClass(String name) {
            System.out.println(name);
              byte[] b = loadClassData(name);
              return defineClass(name, b, 0, b.length);
          }

          //“com/yq/xxx.class” ->  byte[]
          private byte[] loadClassData(String name)  {

              name =  dotToSplit("out.production.MyJVM."+name)+".class" ;
              byte[] result = null ;
              FileInputStream inputStream = null ;
              ByteArrayOutputStream output = null ;
              try {
                 inputStream = new FileInputStream( new File(  name)  );
                //inputStream -> byte[]
                 output = new ByteArrayOutputStream();

                byte[] buf = new byte[2];
                int len = -1;
                while ((len = inputStream.read(buf)) != -1) {
                    output.write(buf, 0, len);
                }
                result = output.toByteArray();
            }catch (Exception e){
                    e.printStackTrace(); ;
            }finally {
                  try {
                      if(inputStream != null )inputStream.close();
                      if(output != null ) output.close();
                  }catch (Exception e){
                      e.printStackTrace();
                  }
            }
            return result ;
          }

    public static void main(String[] args) throws Exception{
            //自定义加载器的对象
        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl();//默认在双亲委派时，会根据正规流程：系统——》扩展->根
//        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl();//直接指定某个 具体的的委派
        Class<?> aClass = myClassLoader.loadClass("com.yanqun.parents.MyDefineCL");
        System.out.println(aClass.getClassLoader());

        MyDefineCL myDefineCL =  (MyDefineCL)(aClass.newInstance() );
        myDefineCL.say();




    }

    public static String dotToSplit(String clssName){
        return clssName.replace(".","/") ;
    }

}


class MyDefineCL{
    public void say(){
        System.out.println("Say...");
    }
}
```

实现流程：

1.public class MyClassLoaderImpl  extends ClassLoader

2.findClass(String name){...} ：直接复制文档中的NetworkClassLoader中的即可

3.loadClassData(String name){...} ：name所代表的文件内容->byte[] 

4.细节：

loadClassData(String name)： 是文件形式的字符串a/b/c.class，并且开头out.production..

findClass(String name):是全类名的形式  a.b.c.class，并且开头 是： 包名.类名.class



操作思路：

要先将 .class文件从classpath中删除，之后才可能用到 自定义类加载器；否在classpath中的.class会被 APPClassLoader加载



```java
package com.yanqun.parents;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

//public class MyException extends Exception{...}
public class MyClassLoaderImpl  extends ClassLoader{
    private String path ; //null
        //优先使用的类加载器是：getSystemClassLoader()
        public MyClassLoaderImpl(){
            super();
        }

        public MyClassLoaderImpl(ClassLoader parent){//扩展类加载器
            super(parent);
        }
        //com.yq.xx.class
        public Class findClass(String name) {
            System.out.println("findClass...");
              byte[] b = loadClassData(name);
              return defineClass(name, b, 0, b.length);
          }

          //“com/yq/xxx.class” ->  byte[]
          private byte[] loadClassData(String name)  {
              System.out.println("加载loadClassData...");
              if(path != null){//name: com.yanqun.parents.MyDefineCL
                  name = path+ name.substring(  name.lastIndexOf(".")+1  )+".class" ;
              }else{
                  //classpath ->APPClassLoader
                  name =  dotToSplit("out.production.MyJVM."+name)+".class" ;
              }




              byte[] result = null ;
              FileInputStream inputStream = null ;
              ByteArrayOutputStream output = null ;
              try {
                 inputStream = new FileInputStream( new File(  name)  );
                //inputStream -> byte[]
                 output = new ByteArrayOutputStream();

                byte[] buf = new byte[2];
                int len = -1;
                while ((len = inputStream.read(buf)) != -1) {
                    output.write(buf, 0, len);
                }
                result = output.toByteArray();
            }catch (Exception e){
                    e.printStackTrace(); ;
            }finally {
                  try {
                      if(inputStream != null )inputStream.close();
                      if(output != null ) output.close();
                  }catch (Exception e){
                      e.printStackTrace();
                  }
            }
            return result ;
          }

    public static void main(String[] args) throws Exception {
        System.out.println("main...");
        //自定义加载器的对象
        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl();//默认在双亲委派时，会根据正规流程：系统——》扩展->根

        myClassLoader.path = "d:/" ;

        //MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl();//直接指定某个 具体的的委派
        Class<?> aClass = myClassLoader.loadClass("com.yanqun.parents.MyDefineCL");
        System.out.println(aClass.getClassLoader());
//        MyDefineCL myDefineCL = (MyDefineCL)( aClass.newInstance()) ;
    }

    public static String dotToSplit(String clssName){  return clssName.replace(".","/") ;  }

}


class MyDefineCL{
    public void say(){
        System.out.println("Say...");
    }
}
```

代码流程：

```
loadClass() ->findClass()->loadClassData()
```

一般而言，启动类加载loadClass()；

**实现自定义加载器，只需要：**

 	**1.继承ClassLoader**

**2重写的 findClass()**



情况一：用APPClassLoader

classpath中的MyDefineCL.class文件：

1163157884
1163157884

d盘中的MyDefineCL.class文件：

356573597

说明，类加载器 只会把同一个类 加载一次； 同一个class文件  加载后的位置



结论：

自定义加载器 加载.class文件的流程：

先委托APPClassLoader加载，APPClassLoader会在classpath中寻找是否存在，如果存在 则直接加载；如果不存在，才有可能交给 自定义加载器加载。

```java
package com.yanqun.parents;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;

//public class MyException extends Exception{...}
public class MyClassLoaderImpl  extends ClassLoader{
    private String path ; //null
        //优先使用的类加载器是：getSystemClassLoader()
        public MyClassLoaderImpl(){
            super();
        }

        public MyClassLoaderImpl(ClassLoader parent){//扩展类加载器
            super(parent);
        }
        //com.yq.xx.class
        public Class findClass(String name) {
//            System.out.println("findClass...");
              byte[] b = loadClassData(name);
              return defineClass(name, b, 0, b.length);
          }

          //“com/yq/xxx.class” ->  byte[]
          private byte[] loadClassData(String name)  {
//              System.out.println("加载loadClassData...");
              if(path != null){//name: com.yanqun.parents.MyDefineCL
//                  System.out.println("去D盘加载;;");
                  name = path+ name.substring(  name.lastIndexOf(".")+1  )+".class" ;
              }

              byte[] result = null ;
              FileInputStream inputStream = null ;
              ByteArrayOutputStream output = null ;
              try {
                 inputStream = new FileInputStream( new File(  name)  );
                //inputStream -> byte[]
                 output = new ByteArrayOutputStream();

                byte[] buf = new byte[2];
                int len = -1;
                while ((len = inputStream.read(buf)) != -1) {
                    output.write(buf, 0, len);
                }
                result = output.toByteArray();
            }catch (Exception e){
                    e.printStackTrace(); ;
            }finally {
                  try {
                      if(inputStream != null )inputStream.close();
                      if(output != null ) output.close();
                  }catch (Exception e){
                      e.printStackTrace();
                  }
            }
            return result ;
          }

    public static void main(String[] args) throws Exception {
//        System.out.println("main...");
        //自定义加载器的对象
//        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl();//默认在双亲委派时，会根据正规流程：系统——》扩展->根
//        myClassLoader.path = "d:/" ;
//        Class<?> aClass = myClassLoader.loadClass("com.yanqun.parents.MyDefineCL");
//        System.out.println(aClass.hashCode());

        MyClassLoaderImpl myClassLoader2 = new MyClassLoaderImpl();//默认在双亲委派时，会根据正规流程：系统——》扩展->根
        Class<?> aClass2 = myClassLoader2.loadClass("com.yanqun.parents.MyDefineCL");
        System.out.println(aClass2.hashCode());


//        System.out.println(aClass.getClassLoader());
//        MyDefineCL myDefineCL = (MyDefineCL)( aClass.newInstance()) ;
    }

    public static String dotToSplit(String clssName){  return clssName.replace(".","/") ;  }

}


class MyDefineCL{
    public void say(){
        System.out.println("Say...");
    }
}
```

通过以下源码可知，在双亲委派体系中，“下面”的加载器 是通过parent引用 “上面”的加载器。即在双亲委派体系中，各个加载器之间不是继承关系。

```java
public abstract class ClassLoader {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    // The parent class loader for delegation
    // Note: VM hardcoded the offset of this field, thus all new fields
    // must be added *after* it.
    private final ClassLoader parent;
```

ClassLoader源码解读

```java
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                   //如果“父类”不为空，则委托“父类”加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        //如果“父类”为空，说明是双亲委派的顶层了，就调用顶层的加载器（BootstrapClassLoader）
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
				//如果“父类”加载失败，则只能自己加载（自定义加载器中的findClass()方法）
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

双亲委派机制优势： 可以防止用户自定义的类 和 rt.jar中的类重名，而造成的混乱

自定义一个java.lang.Math(和jdk中rt.jar中的类重名)

```java
package java.lang;

public class Math {
    public static void main(String[] args) {
        System.out.println("hello Math...");
    }
}

```

运行结果：

![avatar](D:/学习/java/JVM/笔记/图片/1571816579201.png)

![avatar](D:/学习/java/JVM/笔记/图片/1571816095595.png)

原因：根据双亲委派， 越上层的加载器越优先执行。最顶层的加载器是 根加载器，根加载器就会加载rt.jar中的类。因此rt.jar中的Math会被优先加载。 即程序最终加载的是不是我们自己写的Math，而是jdk/rt.jar中 内置的Math;而内置的Math根本没有提供main()方法，因此报 无法找到main()。





实验：将相关联的类A.class和B.class分别用 不同的类加载器加载

**A和B是继承关系**

```java
public class B{
    public B(){
        System.out.println("B被加载了，加载器是： "+ this.getClass().getClassLoader());//对象使用之前，必然先把此对象对应的类加载
    }
}

public class A extends  B
{
    public A(){
        super();
        System.out.println("A被加载了，加载器是： "+ this.getClass().getClassLoader());//对象使用之前，必然先把此对象对应的类加载
    }
}
//AppClassLoader.class : TestMyClassLoader2
//自定义加载器: A.class/B.class
public class TestMyClassLoader2 {
    public static void main(String[] args) throws Exception{
        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl() ;
        //自定义加载路径
        myClassLoader.path = "d:/" ;
        Class<?> aClass = myClassLoader.loadClass("com.yanqun.parents.A");
        Object aObject = aClass.newInstance();//newInstance()会调用 该类的构造方法(new 构造方法())
        System.out.println(aObject);
    }
}

```

**A和B不是继承关系**

```java
public class Y {
    public Y(){
        System.out.println("Y被加载了，加载器是： "+ this.getClass().getClassLoader());//对象使用之前，必然先把此对象对应的类加载
    }
}
public class X {
    public X(){
        new Y() ;//加载Y（系统加载器）
        System.out.println("X被加载了，加载器是： "+ this.getClass().getClassLoader());//对象使用之前，必然先把此对象对应的类加载
    }
}

//AppClassLoader.class : TestMyClassLoader2
//自定义加载器: A.class/B.class
public class TestMyClassLoader3 {
    public static void main(String[] args) throws Exception{
        MyClassLoaderImpl myClassLoader = new MyClassLoaderImpl() ;
        //自定义加载路径
        myClassLoader.path = "d:/" ;
        //程序第一次加载时（X），使用的是  自定义加载器
        Class<?> aClass = myClassLoader.loadClass("com.yanqun.parents.X");



        Object aObject = aClass.newInstance();//newInstance()会调用 该类的构造方法(new 构造方法())
        System.out.println(aObject);
    }
}
```



```java
存在继承关系

A.class:  classpath
B.class:   classpath
原因
同一个AppClassLoader 会同时加载A.class和B.class

--
A.class:   d:\

B.class:   classpath
原因
A.class：自定义加载器加载
B.class：被AppClassLoader加载
因此，加载A.class和B.class的不是同一个加载器


IllegalAccess
---
A.class:    classpath

B.class:    d:\	
NoClassDefFoundError
原因:
A.class: 被AppClassLoader加载  
B.class: 自定义加载器加载
因此，加载A.class和B.class的不是同一个加载器

--
A.class	d:\
B.class d:\
TestMyClassLoader2 can not access a member of class com.yanqun.parents.A with modifiers "public"
A.class/B.class: 自定义加载器加载
原因是 main()方法所在类在 工程中（APPClassLoader），而A和B不在工程中（自定义加载器）。


造成这些异常的核心原因： 命名空间（不是由同一个类加载器所加载）


----
没有继承关系

X.class:  D:		自定义加载器
Y.class:  classpath	系统加载器

Y被加载了，加载器是： sun.misc.Launcher$AppClassLoader@18b4aac2
X被加载了，加载器是： com.yanqun.parents.MyClassLoaderImpl@74a14482


---

X.class:  classpath  系统加载器
Y.class:  D:	    自定义加载器

java.lang.NoClassDefFoundError: com/yanqun/parents/Y

--

```



![avatar](D:/学习/java/JVM/笔记/图片/1571824096839.png)

如果存在继承关系： 继承的双方（父类、子类）都必须是同一个加载器，否则出错；

如果不存在继承关系： 子类加载器可以访问父类加载器加载的类（自定义加载器，可以访问到 系统加载器加载的Y类）；反之不行（父类加载器 不能访问子类加载器）

核心： 双亲委派

如果都在同一个加载器 ，则不存在加载问题； 如果不是同一个，就需要双亲委派。

如果想实现各个加载器之间的自定义依赖，可以使用ogsi规范

![avatar](D:/学习/java/JVM/笔记/图片/OSGi.png)

OSGi：

1.网状结构的加载结构

2.屏蔽掉硬件的异构性。例如，可以将项目部署在网络上，可以在A节点上 远程操作B节点。在操作上，可以对硬件无感。也可以在A节点上 对B节点上的项目进行运维、部署，并且立项情况下  在维护的期间，不需要暂时、重启。



### 类的卸载

1.系统自带（系统加载器、扩展加载器、根加载器）：这些加载器加载的类  是不会被卸载。

2.用户自定义的加载器，会被GC卸载GC


## JVM监测工具

jps: 查看Java进程 （java命令）

jstat:只能查看当前时刻的内存情况；可以查看到 新生代、老年代中的内存使用情况

jmap：查看堆内存的占用情况；也可以执行dump操作

jconsole:图形的监控界面

​	例如：如果通过jconsole中的"执行gc"按钮发现 GC回收的内存太少，就说明当前进程是存在问题的（至少是可以被优化的）

jvisualvm:  监视 - 堆Dump -查找最大对象，从中可以发现 当前进程中是哪个对象 占据了最大的内存，从而对这个对象进行分析。

通过VM参数实现： 当内存溢出时，自动将溢出时刻的内存dump下来。

-Xmx100m
-Xms100m
-XX:+HeapDumpOnOutOfMemoryError



## GC调优

Java开发者为什么不把所有的参数调到最优？非得让我们手工去调？

取舍。

调优实际是是一种取舍，以xx换xx的策略。因此在调优之前，必须明确方向：低延迟？高吞吐量呢？

有两种情况需要考虑：

1.在已知条件相同的前提下， 牺牲低延迟 来换取 高吞吐量，或者反之。

2.随着软件硬件技术的发展，可能 二者都升高。



GC的发展：

JVM自身在GC上进行了很多次的改进升级：

JVM默认的GC:  CMS GC（在jdk9以后被逐步废弃） -> G1 GC(jdk9) -> Z GC(jdk11)

- Serial GC:

  串行GC，是一种在单核环境下的串行回收器。当GC回收的时刻，其他线程必须等待。一般不会使用。

- Parallel GC：

  在Serial 的基础上，使用 了多线程技术。 提高吞吐量。

- CMS GC

  CMS使用了多线程技术，使用“标记-清除”算法，可以极大提升效率 （尤其在低延迟上有很大的提升）。繁琐，参数太多，对开发者的经验要求太高。

- G1 GC

  jdk9开始使用的默认GC。特点：将堆内存划分为很多大小相等region，并会对这些区域的使用状态进行标记。以便GC在回收时，能够快速的定位出哪些region是空闲的，哪些是有垃圾对象，从而提升GC的效率。G1使用的算法是“标记-整理”算法。

- Z GC

  jdk11开始提供全新的GC。回收TB级别的垃圾 在毫秒范围。





如果从生命周期角度划分，GC也可以划分成：Minor GC，和Full GC

- Minor GC：回收新生代中的对象
- Full  GC：回收整个堆空间（新生代、老年代）

案例：

如果通过监测工具发现： Minor GC和Full GC都在频繁的回收，如何优化？

Minor GC为什么会频繁执行？因为 新生代中的对象太多了  Minor GC->短生命周期的对象太多了->造成逃逸到老年代中的对象越多->  新生代多+老年代多->Full GC 

Minor GC:可以尝试调大 新生代的最大空间

再调大 新生代晋升到老年代的阈值，从而降低  短生命周期的对象 从新生代转移到老年代的概率。

