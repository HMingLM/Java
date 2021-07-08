[toc]
## 1.对象的序列化

- 操作对象的流：ObjectInputStream与ObjectOutputStream
- 被操作的对象需要实现**java.io.Serializable**(标记接口)以启用其序列化功能；
- **ObjectOutputStream(OutputStream out)**:创建写入指定OutputStream的ObjectOutputStream；
- **ObjectInputStream(InputStream out)**
- **writeObject(Object obj)**:将指定对象写入ObjectOutputStream; 
- **Object readObject()**:从ObjectInputStream读取对象；


```
public static void writeObj() throws IOException
{
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(obj.txt));
    oos.writeObject(new Person("lisi",39));
    oos.close();
}
public static void readObj() throws Exception
{
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.txt"));
    Person p = (Person)ois.readObject();
    System.out.println(p);
    ois.close();
}
class Person implements Serializable{...}
```

给对象类定义固定标识，序列号(UID)，根据成员获取，静态不能被序列化，加transisent使对应成员不被序列化；

## 2.管道流
- PipedInputStream和PipedOutputStream
- 输入输出可以直接进行连接，通过结合线程使用；

```
PipedInputStream in = new PipedInputStream();
PipedOutputStream out = new PipedOutputStream();
in.connect(out);
Read r = new Read(in);
Writer w = new Writer(out);
new Thread(r).start();
new Thread(w).start();
class Read implements Runnable
{
    private PipedInputStream in;
    Read(PipedInputStream in)
    {
        this.in = in;
    }
    public void run(){try{}catch(){}}
}
class Write implements Runnable
{
    private PipedOutputStream out;
    Read(PipedOutputStream out)
    {
        this.out = out;
    } 
    public void run(){try{}catch(){}}
}
```


## 3.RandomAccessFile
- 该类不算IO体系子类，而是直接继承自Object；是IO包中的成员，因为它具备读写功能；(读写：内部封装了字节输入流和输出流)
- 内部封装了一个数组，通过指针对数组的元素进行操作；
- 可通过getFilePointer获取指针位置，通过seek改变指针位置；
- 随机访问文件，自身具备读写的方法；通过skipBytes(int x)--->跳过指定字节数,**seek(int x)**--->调整对象中指针，来达到随机访问；
- 该类只能操作文件，操作文件有**模式**；(mode:r[只读]/rw[读写]/rws/rwd);
- 若模式为r,不会创建文件，会去读取一个已存在文件，如果该文件不存在则出现异常；
若模式为rw，构造函数要操作的文件不存在会自动创建，如果存在则不会覆盖；

```
public static void writeFile()throws IOException
{
    RandomAccessFile raf = new RandomAccessFile("ean.txt","rw");
    raf.write("李四".getBytes());
    raf.writeInt(97);//用write只存最低八位，可能导致数据不完整；
    raf.write("王五".getByte());
    raf.writeInt(99);
    //raf.seek(8*3);
    //raf.write("周七".getByte());
    //raf.writeInt(103);
    raf.close();
}
public static void readFile()throws IOException
{
    RandomAccessFile raf = new RandomAccessFile("ran.txt","r");
    //raf.skipBytes(8);
    raf.seek(8*1);
    byte[] buf = new byte[4];
    raf.read(buf);
    String name = new String(buf);
    int age = raf.readInt();
    System.out.println("name="+name);
    System.out.println("age="+age);
    raf.close();
}
```


## 3.操作基本数据类型的流对象
- DataInputStream与DataOutputStream
- DataOutputStream(OutputStreamout):创建一个新的数据输出流，将数据写入指定基础输出流；


```
public static void writeData() throws IOException
{
    DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));
    dos.writeInt(234);
    dos.writeBoolean(true);
    dos.writeDouble(987.654);
    dos.close();
}
public static void readData() throws IOException
{
    DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));
    int num = dis.readInt();
    boolean b = dis.readBoolean();
    double d = dis.readDouble();
    System.out.println(num);
    System.out.println(b);
    System.out.println(d);
    dis.close();
}
```


## 4.ByteArrayStream

用于操作字节数组的流对象(用流的思想操作数组)


- **ByteArrayInputStream**:在构造时需要接收数据源，而且数据源是一个字节数组；
- **ByteArrayOutputStream**:在构造时不用定义数据目的地，因为该对象中已经内部封装了可变长度的字节数组，即数据目的地；
- 这两个流对象都操作的是数组，并没有使用系统底层资源，所有不用进行close关闭；
toByteArray(),toString()获取数据，size()获取缓冲区大小；writeTo(OutputStream out)写入指定的输出流；
- 源设备：键盘System.in，硬盘FileStream，内存ArrayStream；
- 目的设备：控制台System.out，硬盘FileStream，内存ArrayStream；


```
ByteArrayInputStream bis = new ByteArrayInputStream("ABCDEFG".getByte());//数据源；
ByteArrayOutputStream bos = new ByteArrayOutputStream();//数据目的；
int by = 0;
while((by=bis.read())!=-1)
{
    bos.write(by);
}
System.out.println(bos.size());
System.out.println(bos.toString());
//bos.writeTo(new FileOutputStream("a.txt"));//要抛出异常；
```


- 操作字符数组:CharArrayReader,CharArrayWrite
- 操作字符串：StringReader,StringWriter


## 5.转换流的字符编码

- 字符编码：字符流的出现为了方便操作字符，更加入了编码转换，通过子类转换流InputStreamReader,OutputStreamWriter来完成,在两个对象进行构造的时候可加入字符集；

- 常见的编码表
> ASCII：美国标准信息交换码；用一个字节的7位可表示；
> 
> ISO8859-1：拉丁码表；用一个字节的8位表示；
> 
> GB2312：中国的中文编码表；
> 
> GBK：中国的中文编码表升级；
> 
> Unicode：国际标准码，融合多种文字；所有文字都用两个字节表示，Java使用的就是unicode；
> 
> UTF-8：最多用三个字节表示一个字符；


```
public static void writeText() throws IOException
{
    OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("utf.txt"),"UTF-8");
    osw.write("你好")；
    osw.close();
}
public static void readText() throws IOException
{
    InputStreamReader isr = new InputStreamReader(new FileInputStream("gbk.txt"),"GBK");
    char[] buf = new char[10];
    int len = isr.read(buf);
    String str = new String(buf,0,len); 
    System.out.println(str);
    isr.close();
}
```



- **编码**：字符串String变成字节数组byte[]，**str.getBytes(charseNmae)**;
- **解码**：字节数组byte[]变成字符串String，**new String(byte[],charseName)**;


```
public static void main(String[] args) throws Exception
{
    String s = "你好";
    byte[] b1 = s.getBytes("GBK");//编码
    System.out.println(Arrays.toString(b1));//打印
    String s1 = new String(b1,"ISO8859-1");//解码为????
    System.out.println("s1="+s1);//打印
    byte[] b2 = s1.getBytes("iso8859-1");//对s1进行iso8859-1编码
    System.out.println(Arrays.toString(b2));//打印
    String s2 = new String(b2,"gbk");//解码为你好
    System.out.println("s2="+s2);//打印
}
```
