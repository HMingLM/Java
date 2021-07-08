[toc]
# 一、流操作
### 1.流操作的基本规律

- i.明确源和目的
源：输入流。InputStream，Reader；
目的：输出流。OutputStream，Writer；
- ii.操作的数据是否是纯文本
是：字符流； 
不是：字节流；
- iii.当体系明确后，再明确要使用哪个具体的对象；
源设备：内存，硬盘，键盘；
目的设备：内存，硬盘，控制台；

### 2.改变标准输入输出设备

- System.setIn(new FileInputStream("XXX.java"));//输入设备；
- System.setOut(new PrintStream("YYY.txt"));//输出设备；

### 3.异常的日志信息

```
try
{
    Date d = new Date();
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String s = sdf.format(d);
    PrintStream ps = new PrintStream("exception.log");
    ps.println(s);
    System.setOut(ps);
}
catch(IOException ex)
{
    throw new RuntimeException("日志文件创建失败");
}
e.printStackTrace(System.out);
```

    
//建立java日志信息的工具包：log4j

### 4.系统信息

**list(PrintStream out)**:将属性列表输出到指定的输出流；

```
Properties prop = System.getProperties();
//System.out.println(prop);
//prop.list(System.out);
prop.list(new PrintStream("sysinfo.txt"));
```

    
# 二、File类
### 1.概述
- 用来将文件或者文件夹封装成对象;
- 方便对文件与文件夹的属性信息进行操作；
- File对象可以作为参数传递给流的构造函数


**separator**:与系统有关的默认名称分割符，被表示为一个字符串；(！跨平台的分隔符！)

将已有的或未有的文件或文件夹封装成File对象：

```
File f1 = new File("a.txt");

File f2 = new File("c:\\abc","b.txt");

File d = new File("c:\\abc");
File f3 = new File(d,"c.txt");

sop(f1);
sop(f2);
sop(f3);

File f4 = new File("c:"+File.separator+"abc"+File.separator+"zzz"+File.separator+"a.txt");
```

    
### 2.File对象功能

##### 2.1 创建

- **boolean createNewFile()**:在指定位置创建文件；如果该文件已经存在，则不创建，返回flase；

和输出流不一样，输出流对象一建立就创建文件，而且文件已存在会覆盖；

- **boolean mkdir()**:创建文件夹；
- **boolean mkdirs()**:创建多级文件夹；
##### 2.2 删除
- **bollean delete()**:删除失败返回flase；
- **void deleteOnExit()**:在程序退出时删除指定文件；

```
public static void method_1() throws IOException
{
    File f = new File("file.txt");
    //f.deleteOnExit();
    System.out.println(f.createNewFile());
    System.out.println(f.delete());
    File dir = new File("a\\b\\c\\d");//创建文件夹
    System.out.println(dir.mkdirs());
}
```


##### 2.3 判断

- **boolean exists()**:文件是否存在；
- **boolean canExecute()**:文件是否可执行；

判断文件对象是否是文件或者目录时，**必须**先判断该文件对象封装的内容是否存在；

- **isFile()**:是否是文件；
- **idDirectory()**:是否是目录；
- **isHidden()**:是否是隐藏文件；
- **isAbsolute()**:是否是绝对路径；(绝对路径就是加上父目录)

```
public static void method_2()
{
    File f = new File("file.txt");
    System.out.println(f.exists());
    System.out.println(f.canExecute());
    
    f.mkdir();
    System.out.println(f.isDirectory());
    System.out.println(f.isFile());
}
```


##### 2.4 获取
- **getName()**:获取名称；
- **getPath()**:获取路径；
- **getParent()**:获取父目录；(返回的是绝对路径下中的父目录，如果相对路径中有上一层目录，那么该目录就是返回结果)
- **getAbsolutePath()**:获取绝对路径；
- **long lastMofified()**:获取最后一次被修改的时间；
- **long length()**：获取文件大小；

##### 2.5 重命名
- **boolean renameTo(File test)**

```
File f1 = new File("c:\\Test.java");
File f2 = new File("d:\\haha.java");
System.out.println(f1.rename(f1));//到了d目录下
```


### 3. 文件列表
##### 3.1 列出文件列表
- **static File[] listRoots()**:列出可用的文件系统根；

```
public static void listRootsDemo()
{
    File[] files = File.listRoots();
    for(File f : files)
    {
        System.out.println(f);
    }
}
```

- **String[] list()**:返回字符串数组，字符串指定该目录下的文件(包括隐藏文件)和文件夹；
- **File[] listFiles()**:区别：返回的是文件对象；

调用list()的file对象必须是封装了一个目录，并且该目录必须存在；

```
public static void listDemo()
{
    File f = new File("c:\\abc.txt");
    String[] names = f.list();
    for(String name : names)
    {
        System.out.println(name);
    }
}
```


- **String[] list(FilenameFilter filter)**:满足指定过滤器的文件和文件夹；
- **File[] listFiles(FilenameFilter filter)**:区别：返回的是文件对象；

过滤时覆盖接口FilenameFilter中的accept方法；
- **boolean accept(File dir,String name)**:测试指定文件是否应该包含在某一文件列表中；

```
File dir = new File("d:\\java1223\\day18");
String[] arr = dir.list(new FilenameFilter()
{
    public boolean accept(File dir,String name)
    {
        return name.endsWith(".bmp");
    }
});
System.out.println("len:"+arr.length);
for(String name : arr)
{
    System.out.println(name);
}
```


##### 3.2 列出目录下所有内容-递归

(包含子目录中的内容)

递归要注意
- 限定条件；
- 要注意递归的次数，尽量避免内存溢出；

```
File dir = new File("d:\\java1223");
public static void showDir(File dir)
{
    System.out.println(dir);
    File[] files = dir.listFiles();
    for(int x=0; x<files.length; x++)
    {
        if(files[x].isSirectory())
            showDirir(files[x]);
        else
            System.out.println(files[x]);
    }
}
```


##### 3.3 删除带内容的目录

```
File dir = new File("d:\\testdir");
public static void removeDir(File dir)
{
    File[] files = dir.listFiles();
    for(int x=0; x<files.length; x++)
    {
        if(files[x].isDirectory())
            removeDir(files[x]);
        else
            System.out.println(files[x].toString()+"-file-"+files[x].delete());
    }
    System.out.println(dir+"-dir-"+dir.delete());
}
```


##### 3.4 创建文件列表文件
将一个指定目录下的Java文件的绝对路径，存储到一个文本文件中；

步骤
- 对指定的目录进行递归；
- 获取递归过程所有的Java文件的路径；
- 将这些路径存储到集合中；
- 将集合中的数据写入到一个文件中；

# 三、properties类
### 1.简述

- properties是hashtable的子类，具备map集合的特点，存储的键值对都是字符串；
- 是集合中和IO技术相结合的集合容器；
- 该对象的特点：可以用于键值对形式的**配置文件**；(配置文件：程序关闭、重启后仍保留原来的信息)
- 加载数据时需要数据由固定格式：键=值；

### 2.properties存取

##### 2.1 基本存取

```
Properties prop = new Properties();
prop.setProperty("zsan","20");//设置元素；
prop.setProperty("lsi","30");
//System.out.println(prop);//直接打印；
Set<String> names = prop.stringPropertyNames();//返回所有键；
for(String s : names)
{
    System.out.println(s+":"+prop.getProperty(s));//输出键及其对应值；
}
```


##### 2.2 存取配置文件

- 将流中的数据存储到集合中(**load(InputStream in)方法**,字符流对应的参数是Reader)；
- setProperty方法改变的时内存结果，**store(OutputStream out,String comments)方法**将内存结果写入输出流，并存到文件上；
- **list(PrintStream out)**：将属性列表输出到指定输出流；

```
Properties prop = new Properties();
FileInputStream fis = new FileInputStream("info.txt");
prop.load(fis);//将流中的数据加载进集合；
prop.setProperty("wangwu","39");
FileOutputStream fos = new FileOutputStream("info.txt");
prop.store(fos,"");//""里为备注信息；
prop.list(System.out);
fos.close();
fis.close();
```


# 四、PrintWriter

- IO包中的其他类-**打印流**对象:可以直接操作输入流和文件；
- 该流提供了打印方法，可以将各种数据类型的数据都原样打印；


- 字节打印流：PrintStream
> 构造函数可接收的类型：
> 
> file对象
> 
> 字符串路径；String
> 
> 字节输出流；OutputStream

- 字符打印流：PrintWriter
> 构造函数可接收的类型：
> 
> file对象
> 
> 字符串路径；String
> 
> 字节输出流；OutputStream
> 
> 字符输出流;Writer

```
public static void main(String[] args) throws IOException
{
    BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
    PrintWriter out = new PrintWriter(System.out,true);//加了true，在识别到打印标记ln时自动刷新;
    //若想把数据存到文件，则传入(new FileWriter("a.txt"),true),把文件封装流中可以刷新；
    String line = null;
    while((line=bufr.readLine())!=null)
    {
        if("over".equals(line))
            break;
        out.println(line.toUpperCase());
        //out.flush();
    }
    out.close();
    bufr.close();
}
```


# 五、序列流(合并流)
- SequenceInputStream:对多个流进行合并;(多个源，一个目的地)


```
public static void main(String[] args)
{
    Vector<FileInputStream> v = new Vector<FileInputStream>();
    v.add(new FileInputStream("c:\\1.txt"));
    v.add(new FileInputStream("c:\\2.txt"));
    v.add(new FileInputStream("c:\\3.txt"));
    Enumeration<FileInputStream> en = v.elements();
    SequenceInputStream sis = new SequenceInputStream(en);
    FileOutputStream fos = new FileOutputStream("c:\\4.txt");
    byte[] buf = new byte[1024];
    int len = 0;
    while((len=sis.read(buf))!=-1)
    {
        fos.write(buf,0,len);
    }
    fos.close();
    sis.close();
}
```


- 切割文件


```
public static void splitFile() throws IOException
{
    FileInputStream fis = new FileInputStream("c:\\1.bmp");
    FileOutputStream fos = null;
    byte[] buf = new byte[1024*1024];
    int len = 0;
    int count = 1;
    while((len=fis.read(buf))!=-1)
    {
        fos = new FileOutputStream("c:\\splitfiles\\"+(count++)+".part");
        fos.write(buf,0,len);
        fos.close();
    }
    fis.close();
}
```


- 然后合并


```
public static void merge() throws IOException
{
    ArrayList<FileInputStream> al = new ArrayList<FileInputStream>();
    for(int x=1; x<=3; x++) 
    {
        al.add(new FileInputStream("c:\\splitfiles\\"+x+".part"));
    }
    final Iterator<FileInputStream> it = al.iterator();
    Enumeration<FileInputStream> en = new Enumeration<FileInputStream>()
    {
        public boolean hasMoreElements()
        {
            return it.hasNext();
        }
        public FileInputStream nextElement()
        {
            return it.next();
        }
    };
    SequenceInputStream sis = new SequenceInputStream(en);
    FileOutputStream fos = new FileOutputStream("c:\\splitfiles\\0.bmp");
    byte[] buf = new byte[1024];
    int len = 0;
    while((len=sis.read(buf)!=-1)
    {
        fos.write(buf,0,len);
    }
    fos.close();
    sis.close();
}
```

    