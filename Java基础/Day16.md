[toc]
# 四、其他对象 

### 1.System

描述系统一些信息，类中的方法和属性都是静态的

- out：标准输出，默认是控制台； 
- in：标准输入，默认是键盘；

获取系统所有属性信息：Properties **getProperties()**;

- **Properties**:是Hashtable的子类，也就是Map集合的一个子类对象，可通过map的方法取出该集合中的元素；该集合中存储的都是字符串，没有泛型定义； 


```
    Properties prop = System.getProperties();
    for(Object obj : prop.keySet())
    {
        String value = (String)prop.get(obj);
        System.out.println(obj+"::"+value);
    }
```



- setProperties(String key,String value):设置指定键指示的系统属性；  
- getProperties(String key):获取指定键指示的系统属性；

### 2.Runtime

Runtime没有提供构造函数，不可以new对象；

有非静态方法，提供了方法获取本类对象；(单例设计模式)

- **static Runtime getRuntime()**:返回与当前应用程序相关的运行时对象；
- **Process exec(String command)**:在单独的进程中执行指定的字符串命令；

```
    public static void main(String[] args)
    {
        Runtime r = Runtime.getRuntime();
    r.exec("winmine.exe");
    }
```

    
### 2.Date

```
    import java.util.*;
    import java.text.*;
    class DateDemo
    {
        punlic static void main(String[] args)
        {
            Date d = new Date();
            System.out.println(d);//打印的时间没有格式；
            //将模式封装到SimpleDateFormat对象中；
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日E hh:mm:ss");
            //调用format方法让模式格式化指定Date对象；
            String time = sdf.format(d);
            System.out.println(time);
        }
    }
```


### 3.Calendar

```
    Calendar c = Calendar.getInstance();
    System.out.println(c.get(Calendar.YEAR)+"年");
```


```
    c.set(2012,2,23);
    c.add(Calendar.YEAR,4);
```

    
### 3.Math

- **Math.ceil(double a)**:返回大于指定数据的最小整数；
- **Math.ceil(double a)**:返回小于指定数据的最大整数；
- **round(double a)**:返回最接近参数的long；(四舍五入)
- **Math.pow(double a,double b)**:返回第一个参数的第二个参数次幂的值；

**Random:随机数**
- 方法一：Random类

```
    import java.util.*;
    Random r = new Random();
    for(int x=0; x<10; x++)
    {
        int d = r.nextInt(10);
        S.o.p(d);
    }
```

- 方法二：  

返回大于等于0.0，且小于1.0的随机值；


```
    for(int x=0; x<10; x++)
    {
        int d = (int)(Math.random()*10);
        S.o.p(d);
    }
```
