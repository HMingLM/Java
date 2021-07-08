[toc]
## 一、String类
### 1.理解
- 字符串是一个特殊的对象；
- 字符串一旦初始化就不可以被改变；

> String s1 = "abc"; 
> //s1是一个类类型变量，"abc"是一个对象；
> 
> String s2 = new String("abc");
> 
> String s3 = "abc";
> 
> String s4 = new String("abc");
- **s1和s2的区别**：s1在内存中有一个对象("abc")，存放于静态区；
- s2在内存中有两个对象("abc"一个，又new出了一个string对象)；存放于堆；

> (s1==s2)--->flase;
> 
> (s1.equals(s2))--->true;
> 
> (s1==s3)--->true;
> 
> (s2==s4)--->false;

**equals比较的是内容；==比较的是内存地址；**

### 2.常见功能

###### 2.1.获取
- **int length()**:获取字符串长度(字符串中包含的字符数)；
- **char charAt(int index)**:根据位置获取位置上的字符；
- **int indexOf(int ch)**:获取字符ch在字符串中第一次出现的位置；
- **int indexOf(int ch, int fromIndex)**:从fromIndex指定位置开始，获取ch在字符串中出现的位置；
- **int indexOf(String str)**:获取小字符串str在字符串中第一次出现的位置；
- **int indexOf(String str, int fromIndex)**:从fromIndex指定位置开始，获取str在字符串中出现的位置；
- **int lastIndexOf(XXX)**:反向索引;

注：若没有找到，返回-1；所以可用于对指定判断是否包含；
###### 2.2.判断
- **boolean contains(str)**:判断字符串中是否包含某一个子串；
- **boolean isEmpty()**:判断字符串中是否有内容(长度是否为0)；
- **boolean startsWith(str)**:判断字符串是否以指定内容开头；
- **boolean endsWith(str)**:判断字符串是否以指定内容结尾；
- **boolean equals(str)**:判断字符串内容是否相同；(复写了object类中的equals方法)
- **boolean equalsIgnoreCase(str)**:判断内容是否相同，并忽略大小写；

###### 2.3.转换

- **char[] toCharArray()**:将字符串转成字符数组;
- **String(byte[])**:将字节数组转成字符串；
- **String(byte[],offset,count)**:将字节数组中的一部分转成字符串；
- **byte[] getBytes()**:将字符串转成字节数组；
- **static String valueOf(int/double)**:将基本数据类型转成字符串；

**将字符数组转成字符串**

构造函数：
- **String(char[])**:将字符数组转成字符串；
- **String(char[],offset,count)**:将字符数组中的一部分转成字符串；

静态方法:
- **static String copyValueOf(char[])**;
- **static String copyValueOf(char[] date, int  offset, int count)**;
- **static String ValueOf(char[])**;

特殊：字符串和字节数组在转换过程中，是可以指定编码表的；

###### 2.4.替换

**String replace(oldchar,newchar)**;如果要替换的字符不存在，返回的还是原串；

###### 2.5.切割
**String[] split(regex)**;

> 例:String s = "zhangsan,lisi,wangwu";
> 
> String[] arr = s.split(",");
> 
> for(int x = 0; x<arr.length; x++){
>     System.out.println(arr[x]);
> }

###### 2.6.字串：获取字符串中的一部分

- **String substring(begin)**:从指定位置开始到结尾；如果角标不存在，会出现字符串角标越界异常；
- **String substring(begin,end)**:包含头，不包含尾；

###### 2.7.转换，去除空格，比较
- **String toUpperCase()**:将字符串转成大写；
- **String toLowerCase()**:将字符串转成小写；
- **String trim()**:将字符串两端的多个空格去除；
- **int compareTo(String)**:对两个字符串进行自然顺序的比较；

### 3.StringBuffer

- StringBuffer是字符串缓冲区；
是一个容器
###### 3.1.特点
- 长度是可变化的；
- 可以直接操作多个数据类型；
- 最终会通过toString方法变成字符串；
 
###### 3.2.常见功能-添加
- **StringBuffer append()**:将指定数据作为参数添加到已有数据结尾处；
- **StringBuffer insert(index,数据)**:可以将数据插入到指定index位置


###### 3.3.删除
- **StringBuffer delete(strat,end)**:删除缓冲区中的数据，包含strat，不包含end；
- **StringBuffer deleteCharAt(index)**:删除指定位置的字符；

###### 3.4.获取
- **char charAt(int index)**
- **int indexOf(String str)**
- **int lastIndexOf(String str)**
- **int length()**
- **String substring(int start,int end)**

###### 3.5.修改
- **StringBuffer replace(start,end,string)**;
- **void setCharAt(int index,char ch)**;
- **StringBuffer reverse()**:反转
- **void getChars(int srcBegin,int srcEnd,char[] dst,int dstBegin)**:将缓冲区中指定数据存储到指定字符数组中；

### 4.StringBuilder

- **StringBuffer**:线程同步，多线程时安全；
- **StringBuilder(建议使用)**:线程不同步，==效率高==；(提供了与StringBuffer兼容的API)

升级三个因素：
i.提高效率；
ii.简化书写；
iii.提高安全性；

## 二、基本数据类型对象包装类

####  1.基本数据类型(关键字)--->基本数据类型包装类

> byte--->Byte
> 
> short--->Short
> 
> int--->Integer
> 
> long--->Long
> 
> boolean--->Boolean
> 
> float--->Float
> 
> double--->Double
> 
> char--->Character

整数类型的最大值:MAX_VALUE

#### 2.类型转换
基本数据类型对象包装类的最常见作用，就是用于基本数据类型和字符串类型之间做转换；
###### 2.1.基本数据类型转成字符串：

**基本数据类型+""**

**基本数据类型.toString(基本数据类型值)**；

###### 2.2.字符串转成基本数据类型 

**xxx a = Xxx.parseXxx(String)**;

**int a = Integer.parseInt("123")**;

###### 2.3.十进制转成其他进制

**Integer.toBinaryString()**;

**Integer.toHexString()**;
 
**Integer.toOctalString()**;

###### 2.4.转成其他进制十进制

**Integer.parseInt(string,radix)**

#### 3.基本数据类型对象包装类新特性
**Integer x = 4;**//自动装箱；

**即为**Integer x = new Integer(4);
 
**x = x + 2;**//进行自动拆箱，变成int类型进行加法运算；再将运算结果进行装箱赋给x；


