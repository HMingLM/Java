[toc]
## 一.内部类 
- 将一个类定义在另一个类里面，里面那个类成为内部类（内置类，嵌套类）；

#### 1.访问规则：
- 内部类可直接访问外部类中的成员，包括私有成员；
- 因为内部类中持有了一个外部类的引用，格式：==外部类名.this.成员==;
- 外部类要访问内部类中的成员必须建立内部类的对象；

#### 2.访问格式

##### 2.1.当内部类定义在外部类的==成员位置==上
###### 2.1.1.非私有时
- 可以在外部其他类中直接建立内部类对象；
- **格式**：外部类名.内部类名 变量名 = 外部类对象.内部类对象；
- Outer.Inner in = new Outer().new Inner();

###### 2.1.2.可以被成员修饰符所修饰
- 如**private**：将内部类在外部类中进行封装；
- **static**：内部类就具备了static的特性；（静态内部类）

##### 2.2.当内部类定义在局部时
- 不可以被成员修饰符修饰
- 可以直接访问外部类中的成员，因为还持有外部类中的引用；
- 但是不可以访问它所在的局部中的变量；只能访问被final修饰的局部中的变量；

##### 2.3.静态内部类
- 当内部类被static修饰后，只能直接访问外部类中的static成员；出现了访问局限；
- 在外部其他类中直接访问static内部类的非静态成员：
new Outer.Inner().function();
- 在外部其他类中直接访问static内部类的静态成员：
Outer.Inner.function();

##### 2.4.注意
- 当内部类中定义了静态成员，该内部类必须是static的；
当外部类中的静态方法访问内部类时，该内部类也必须是static的；

#### 3.定义原则
- 当描述事物时，事物的内部还有事物，该事物用内部类来描述；
- 因为内部事务在使用外部事物的内容

#### 4.匿名内部类
- 匿名内部类其实就是内部类的简写格式
- 定义匿名内部类的前提：
内部类必须是继承一个类或者实现接口
- 匿名内部类的格式：new 父类或者接口（）{定义子类的内容}
- 其实匿名内部类就是一个匿名子类对象，可以理解为带内容的对象；
- 匿名内部类中定义的方法最好不要超过3个；
- 示例


```
class Inner extends AbsDemo
{
    void show()
    {
        System.oout.println("show:"+x);
    }
}
```


==即为==


```
new AbsDemo()
{
    void show()
    {
        System.oout.println("show:"+x);
    }
}.show();
```


## 二.异常
#### 1.理解
- 异常：就是程序在运行时出现不正常情况；
- 异常由来：问题也是现实生活中一个具体的事物，也可以通过java的类的形式进行描述，并封装成对象；
- 其实就是java对不正常情况进行描述后的对象体现；

#### 2.体系
==Throwable==：
无论Error或者Exception都具有一些共性内容，（抽取父类Throwable）

##### 2.1.Error
- 通过Error类描述严重的问题如：运行的类不存在或者内存溢出等；
- 不编写针对代码对其处理；

##### 2.2.Exception
- 通过Exception类描述非严重的问题；
- 可以通过针对性的处理方式及逆行处理（try catch finally）；

Exception和Error的子类名基本都是以父类名作为后缀

#### 3.异常的处理
#####  3.1.java提供了特有的语句进行处理

==try== 

{
    需要被检测的代码；
}

==catch(异常类 变量)==

{
    处理异常的代码；（处理方式）
}

==finally==

{
    一定会执行的语句；
}

##### 3.2.对捕获到的异常对象进行常见方法操作；


```
catch (Exception e)//Exception e = new ArithmeticException();
{
    System.out.println(e.getMessage());//异常信息
    System.out.println(e.toString());//异常名称：异常信息
    e.printStackTrace();//异常名称，异常信息，异常出现的位置；
    //其实jvm默认的异常处理机制就是在调用printStackTrace方法。打印异常的堆栈的跟踪信息；
}
```


#### 4.异常声明==throws==

- 在功能上通过throws关键字声明了该功能有可能会出现问题；


```
int div(int a,int b)throws Exception
{
    return a/b;
}
```


#### 5.对多异常的处理
- 声明异常时，建议声明更为具体的异常，这样处理更具体；
- int div(int a,int b)throws ArithmeticException,ArrrayIndexOutOfBoundsException
- 对方声明几个异常，就对应有哦几个catch块；不要定义多余的catch块；
- 如果多个catch块中的异常出现继承关系，父类异常catch块放在最下面；
- 建议在进行catch处理时一定要定义具体处理方式；

#### 6.自定义异常

- 对项目中出现的特有问题（未被java描述并封装对象）进行自定义的异常封装；
- 自定义类继承Exception;(因为Throwable体系中的类和对象才可被throws和throw操作)


```
class FuShuException extends Exception
{
    FuShuException(String msg)
    {
        super(msg);
    }
}

class Demo
{
    int div(int a,int b)throws FuShuException
    {
        if(b<0)
        throw new FuShuException("自定义异常信息"); //手动通过throw关键字抛出一个自定义异常对象；
        return a/b;
    }
}
```


##### 6.1throws和throw的区别

==throws==:使用在函数上；

后面跟的是异常类，可跟多个（逗号隔开）；

==throw==:使用在函数中；

后面跟的是异常对象；

##### 6.2RuntimeException
- Exception中有一个特殊的子类异常RuntimeException运行时异常；
- 自定义异常时：如果该异常的发生，无法再继续运算，就让自定义异常继承==RuntimeException==;
- 如果在函数内抛出该异常，函数上可以不用声明，编译一样通过；
- 如果在函数上声明了该异常，调用者可以不用进行处理，编译一样通过；
因为不需要让调用者处理，希望停止程序，修正带代码；

> 异常分两种：
> 
> 1.编译时被检测的异常
> 
> 2.编译时不被检测的异常（运行时异常，RuntimeException以及其子类）

#### 7.finally
- finally中存放的是一定会被执行的代码（只有当执行到System.exit(0);finally
不执行），通常用于关闭资源； 
- 异常处理语句其他格式：
> try-catch
> 
> try-catch-finally
> 
> try-finally
- catch用于处理异常，如果没有catch就代表异常没有被处理过，如果该异常时检测时异常，那么必须声明；

#### 8.异常在子父类覆盖中的体现
- 子类在覆盖父类时，如果父类的方法抛出异常，那么子类的覆盖方法，只能抛出父类的异常或者该异常的子类（或不抛）；
- 如果父类方法抛出多个异常，那么子类在覆盖该方法时，只能抛出父类异常的子集；
- 如果父类或者接口的方法中没有异常抛出，那么子类在覆盖方法时，也不可以抛出异常；
- 如果子类方法发生了异常，必须进行try处理，绝对不能抛；
> 简单说：子类不能抛比父类多异常；
