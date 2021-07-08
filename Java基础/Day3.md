[toc]
## 一.常用函数

**1.键盘输入**
new Scanner(System.in)

**输出**：System.out.println(binstr)

**2.十进制转化为二进制**
Integer.toBinaryString(int i)

**3.十进制转八进制**
Integer.toOctalString(int i)

**4.十进制转十六进制**
Integer.toHexString(int i)

**5.十六进制转十进制**
- Integer.valueOf("FFFF",16).toString

> Integer i =  integer.parseInt("00001509",16);
> 
> System.out.println(i.intValue());

**6.八进制转十进制**
Integer.valueOf("5",8).toString()

**7.二进制转十进制**
Integer.valueOf("0101",2).toString()

**8.string转int**
Integer.valueOf("12")

**9.int转string**
String.valueOf(12)

**10.char转int**
**先将char转string**
String.valueOf('2')

**再转int:**
Integer.valueof(str)  *返回的是Integer对象*

Integer.PaseInt(str)  *返回的是int*

**11.舍掉小数取整**
:Math.floor(3.5)=3

**12.四舍五入取整**
:Math.rint(3.5)=4

**13.进位取整**
:Math.ceil(3.1)=4

**14.取绝对值**
:Math.abs(-3.5)=3.5

**15.字符串“2019-7-8”转换成Date型**（要引入包）

> String dateStr="2019-07-08";
> 
> SimpleDateFormat spf=new
> SimpleDateFormat("yyyy-MM-dd");
> 
> Date date=spf.parse(dateStr);
> 
> System.out.println(date.toString());
	
## 二.语句
 
#### 1.while;do while

后者先执行循环体，再判断条件，无论条件是否满足，循环体至少执行一次

#### 2.for

- 无限循环的最简单表达形式for(;;){} 或 while(true){}
- 对于for，若将用于控制循环的增量定义在for语句中，那么该变量只在for语句内有效（优化：只作循环增量）

**累加思想，计数器思想，嵌套语句**

#### 3.break,continue
应用范围：break选择结构和循环结构；continue循环结构

## 三.函数 

#### 1.函数：
定义在类中的具有特定功能的一段独立小程序；

#### 2.格式：

```
修饰符 返回值类型 函数名（参数类型 形式参数1， 参数类型 形式参数2， ...)
{
    执行语句；
    return 返回值；
}
```


#### 3.特点

可将功能代码进行封装，提高了代码的复用性，对于函数没有具体返回值的情况，返回值类型用关键字void表示，且该函数中的return语句如果在最后一行可以省略不写；

#### 4.注意：
函数中只能调用函数，不可以在函数内部定义函数；

#### 5.函数的应用：
明确函数的返回值类型和函数的参数列表（参数的类型和参数的个数）

#### 6.函数的重载

- **概念**：在同一个类中，允许存在一个以上的同名函数，只要它们的参数个数或者参数类型不同即可；
- **特点**：与返回值类型，参数名称无关，只看参数列表；
- **好处**：方便阅读，优化了程序设计；

## 四.数组

#### 1.概念：
同一种类型数据的集合，（数组就是一个容器）；

#### 2.好处：
可自动给数组中的元素从0开始编号，方便操作这些元素；

#### 3.初始化格式：

- 元素类型【】 数组名 = new 元素类型【元素个数或数组长度】；
- 元素类型【】 数组名 = new 元素类型【】{元素，元素，...}；(数据明确时)

#### 4.内存

- **栈内存**：数据使用完毕，内存自动释放（如等号左边部分）；
- **堆内存**（如等号右边部分）


#### 5.直接获取数组元素个数：

**数组名称.length** = 

获取数组元素通常会用到遍历；

#### 6.排序：
**Arrays.sort(arr)**;

- 将排序中用到的位置置换功能抽取为函数；

区别选择排序与冒泡排序；

- **选择排序**：内循环结束一次，最值出现在头角标位置；
- **冒泡排序**：相邻的两个元素进行比较，若符合条件换位，（第一圈，最大值出现在最后位）；

#### 7.折半查找：
提高效率，但必须要保证该数组是**有序的**数组；

- while(arr[mid]!=key)
- while(min<=max)
- ***应用***：将一个元素插入到该数组中，并保证该数组是有序的，获取该元素在数组中的位置；

## 五.进制转化

（！！**见任务一题4==jinzhizh.java==**,参考==Day4视频12==！！）

- **查表法**：（可通过数组的形式）将所有的元素（0~F）临时存储起来，建立对应关系，每一次&15后的值作为索引去查建立好的表，就可找到对应的元素，这样比 -10+'a' 简单得多
- 结果是反着的，想要正过来，可通过StringBuffer reverse功能来完成，可使用学过的容器：数组来完成存储；
- **扩展（例子为十进制转二进制**）：


```
StringBuffer sb = new StringBuffer();
while(num>0)
{
    sb.append(num%2);  // 直接添加
    num=num/2;
}
System.out println(sb.reverse());  // 反向输出
```


## 六.二维数组

#### 格式

- int [ ] [ ] arr = new int[3][2];(二维数组中有三个一维数组)
- int [ ] [ ] arr = new int[3] [ ](每个一维数组都是默认初始化值null)

> 可以对这三个一维数组分别进行初始化:
>
> arr[0] = new int[3];
> 
> arr[1] = new int[1];
> 
> arr[2] = new int[2];

- **int[ ] x,y[ ]****;** --> **即** int [ ] x;   int[ ] y[ ];










