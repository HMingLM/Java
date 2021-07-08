[toc]
# IO流
## 一、概述
#### 1.理解
- IO流用来处理设备之间的数据传输；
- Java对数据的操作是通过流的方式；
- Java用于操作流的对象都在IO包中；
- 流按操作数据分为两种：字节流，字符流；
- 流按流向分为：输入流，输出流；

#### 2.IO流常用基类
- 字节流的抽象基类：InputStream，OutputStream；
- 字符流的抽象基类：Reader，Writer；
- 注：由这四个类派生出来的子类名称都是以其父类名作为子类名的后缀；(如FileInputStream,FileReader)；前缀名是该流对象的功能；

## 二、FileWriter
#### 1.写入数据
FileWriter:专门用于操作文件的Writer子类对象;

- **FileWriter(String fileName)**:构造方法，根据给定的文件名构造一个FileWriter对象；
> 该对象一被初始化就必须要明确被操作的文件；
> 
> 该文件会被创建到指定目录下，如果该目录下已有同名文件，将覆盖；
> 
> 其实就是在明确数据要存放的目的地；
- **write(String str)**:将字符串写入流中；(内存中)
> write(String str,int off,int len):写入字符串的某一部分；
> 
> write():还可以写入单个字符，字符数组，字符数组的某一部分；
- **flush()**:刷新该流对象的缓冲中的数据；将数据刷到目的地中；
- **close()**:刷新一次内部缓冲中的数据,将数据刷到目的地中(flush)，并关闭流资源；(一定要做)

```
import java.io.*;
class FileWriterDemo
{
    public static void main(String[] args) throw IOException
    FileWriter fw = new FileWriter("demo.txt");
    fw.writer("abcde");
    //fw.flush():
    fw.close():
}
```

    
#### 2.IO异常的处理方式

```
import java.io.*;
class FileWriterDemo
{
    public static void main(String[] args)
    {
        FileWriter fw = null;//在try的外边引用，finally中才能执行；
        try
        {
            fw = new FileWriter("demo.txt");
            fw.write("abcdef");
        }
        catch(IOException e)
        {
            System.out.println(e.toString());
        }
        finally
        {
            try
            {
                if(fw!=null)    //若初始化时异常，fw为null;
                    fw.close();
            }
            catch(IOException e)
            {
                System.out.println(e.toString());
            }
        }
    }
}
```

    
    
#### 3.文件的续写

```
FileWriter fw = new FIleWriter("demo.txt",true);
```

- 传递一个**true**参数，代表不覆盖已有的文件，并在已有文件的末尾处进行数据续写；


```
fw.write("ni\r\nhao");
```


- \r\n：回车(单独的\n在window的记事本不识别)

## 三、FileReader
#### 1.文本文件的读取方式

###### 1.1方式一

```
FileReader fr = new FileReader("demo.txt");
```

- 创建一个文件读取流对象，和指定名称的文件相关联；
- 要保证该文件是存在的，否则会发生异常FileNotFoundException


```
int ch = 0;
while((ch=fr.read())!=-1)
{
    System.out.println((char)ch);
}
fr.close():
```

- **read()**:调用读取流对象的方法，一次读一个字符(返回整数)，而且会自动往下读，读到流的末尾返回-1；


###### 1.2方式二

通过字符数组进行读取；
- **int read(char[] cbuf)**:将字符读入数组；返回读取的字符数，读到流的末尾返回-1；

```
    FileReader fr = new FileReader("demo.txt");
    char[] buf = new char[1024];//可以是1024的整数倍；
    int num = 0;
    while((num=fr.read(buf))!=-1)
    {
        System.out.println(new String(buf,0,num));
    }
    fr.close():
```


#### 2.拷贝文本文件

步骤：
- 在D盘创建一个文件，用于存储C盘文件中的数据；
- 定义读取流和C盘文件关联；
- 通过不断地读写完成数据存储；
- 关闭资源；

```
    import java.io.*;
    class CopyText
    {
        public static void main(String[] args) throws IOException
        {
            copy();
        }
        public stattic void copy()
        {
            FileWriter fw = null;
            FileReader fr = null;
            try
            {
                fw = new FileWriter("SystemDemo_copy.txt");
                fr = new FileReader("SystemDemo.java");
                char[] buf = new char[1024];
                int len = 0;
                while((len=fr.read(buf)!=-1)
                {
                    fw.write(buf,0,len);
                }
            }
            catch(IOException e)
            {
                throw new RuntimeException("读写失败");
            }
            finally
            {
                if(fr!=null)
                try
                {
                    fr.close();
                }
                catch(IOException e)
                {
                }
                if(fw!=null)
                try
                {
                    fw.close():
                }
                catch(IOException e)
                {
                }
            }
        }
    }
```

    
## 四、字符流的缓冲区

- 缓冲区的出现提高了对数据的读写**效率**；
- 对应类：BufferedWriter,BufferedReader；
- 缓冲区要**结合流**才可以使用；(是为了提高流的操作效率而出现的，所以创建缓冲区之前要先有流对象)
- 在流的基础上对流的功能进行了增强；
- 只要用到缓冲区，就要记得**刷新**；
- 关闭缓冲区，就是在**关闭**缓冲区中的流对象；
#### 1.BufferedWriter

```
    FileWriter fw = new FileWriter("buf.txt");//创建字符写入流对象；
    BufferedWriter bufw = new BufferedWriter(fw);//将需要被提高效率的流对象作为参数传递给缓冲区的构造函数；
    bufw.write("abcde");
    //bufw.flush();//缓冲区记得刷新；
    bufw.close();//刷新，关闭缓冲区(关闭流对象)；
```



- **newLine()**:该缓冲区提供的跨平台的换行符；(windows中\r\n,linux中\n)

#### 2.BufferedReader

- **String readLine()**:该缓冲区提供的方法；读取一个文本行；返回包含该行内容的字符串，不含任何行终止符(如回车符)；到达流的末尾返回null；

```
    FileReader fr = new FileReader("buf.txt");//创建读取流对象与文件相关联；
    BufferedReader bufr = new BufferReader(fr);//将字符读取流对象作为参数传递给缓冲对象的构造函数；
    String line = null;
    while((line=bufr.readLine())!=null)
    {
        System.out.println(line);
    }
    bufr.close();
```

    
#### 3.通过缓冲区复制文本文件


```
    BufferedReader bufr = null;
    BufferedWriter bufw = null;
    try
    {
        bufr = new BufferedReader(new FileReader("XXX.java"));
        bufw = new BufferedWriter(new FileWriter("XXX_Copy.txt"));
        String line = null;
        while((line=bufr.readLine())!=null)
        {
            bufw.write(line);
            bufw.newLine();
            bufw.flush();
        }
    }
    catch(IOException e)
    {
        throw new RuntimeException("读写失败")；
    }
    finally
    {
        try
        {
            if(bufr!=null)
                bufr.close();
        }
        catch(IOException e)
        {
            throw new RuntimeException("读取关闭失败")；
        }
        try
        {
            if(bufw!=null)
                bufw.close();
        }
        catch(IOException e)
        {
            throw new RuntimeException("写入关闭失败")；
        }
    }
```

    
## 五、装饰设计模式
#### 1.装饰
- 当想要对已有的对象进行功能增强时(如readLine()对于read())，可以定义类，将已有对象传入，基于已有的功能，并提供加强功能；
- 自定义的该类称为装饰类；
- 装饰类通常会通过构造方法接收被装饰的对象，并基于被装饰对象的功能，提供更强的功能；

#### 2.装饰和继承的区别

- 装饰模式比继承更灵活，避免了继承体系臃肿，而且降低了类与类之间的关系；
- 装饰类因为增强已有对象，具备的功能和已有的是相同的，只不过提供了更强功能；
所以装饰类和被装饰类通常都属于一个体系中的；

#### 3.装饰类LineNumberReader


```
    FileReader fr = new FileReader("XXX.java");
    LineNumberReader lnr = new LineNumberReader(fr);
    String Line = null;
    //lnr.setLineNumber(100);//从101开始排行号;
    while((line=lnr.readLine())!=null)
    {
        System.out.println(lnr.getLineNumber()+":"+line);//每行数据前都有行号；
    }
    lnr.close();
```

    
## 六、字节流File读写操作

没有使用具体指定缓冲区时，**不需要刷新数据**，因为是对字节最小单位的操作，不需要转换
#### 1.FileOutputStream


```
    public static void writeFile() throws IOException
    {
        FileOutputStream fos = new FileOutputStream("fos.txt");
        fos.write("abcde".getBytes());//字符串转成字节数组；
        fos.close();//关闭资源；
    }
```

    
#### 2.FileInputStream


```
    public static void readFile_1() throws IOException
    {
        FileInputStream fis = new FileInputStream("fos.txt");
        byte[] buf = new byte[1024];
        int len = 0;
        while((len=fis.read(buf))!=-1)
        {
            System.out.println(new String(buf,0,len));
        }
        fis.close();
    }
```

    
使用available():【数据太大时不能用】


```
    public static void readFile_2() throws IOException
    {
        FileInputStream fis = new FileInputStream("fos.txt");
        byte[] buf = new byte[fis.available()];//定义一个刚刚好的缓冲区；
        fis.read(buf);
        System.out.println(new String(buf);
        fis.close();
    }
```

#### 3.拷贝图片

步骤：
- 用字节读取流对象和图片关联；
- 用字节写入流对象创建一个图片文件，用于存储获取到的图片数据；
- 通过循环读写，完成数据的存储；
- 关闭资源；


```
    import java.io.*;
    calss CopyPic
    {
        public static void main(String[] args)
        {
            FileOutputStream fos = null;
            FileInputStream fis = null;
            try
            {
                fos = new FileOutputStream("c:\\2.bmp");
                fis = new FileInputStream("c:\\1.bmp");
                byte[] buf = new bute[1024];
                int len = 0;
                while((len=fis.read(buf))!=-1)
                {
                    fos.write(buf,0,len);
                }
                
            }
            catch(IOException e)
            {
                throw new RuntimeException("复制文件失败")；
            }
            finally
            {
                try
                {
                    if(fis!=null)
                       fis.close();
                }
                catch(IOException e)
                {
                    throw new RuntimeException("读取关闭失败")；
                }
                try
                {
                    if(fos!=null)
                       fos.close();
                }
                catch(IOException e)
                {
                    throw new RuntimeException("写入关闭失败")；
                }
            }
        }
    }
```

## 七、字节流的缓冲区
#### 1.通过缓冲区复制mp3
BufferedOutputStream,BufferedInputStream


```
    public static void copy() throws IOException
    {
        BufferedInputStream bufis = new BufferedInputStream(new FileInputStream("c:\\0.mp3");
        BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream("c:\\1.mp3");
        int by = 0;
        while((by=bufis.read())!=-1)
        {
            bufos.write(by);
        }
        bufos.close();
        bufis.close();
    }
```

    
#### 2.read和write的特点

- **read()**：将byte提升为int；
- **write()**：将int强转为byte；

保持原字节数据不变的同时，避免了-1的出现，这就是为什么read()返回的是int而不是byte();

如read()将11111111(即-1)转化为int时在它前面补0，此时的int数据不是-1，写入时write()将其转为byte型，去掉前面的0；

## 八、读取键盘录入

- System.out:对应的是标准输出设备：控制台；
- System.in:对应的是标准输入设备：键盘；

**题**：键盘录入一行数据并打印出其大写


```
    public static void main(String[] args) throws IOException
    {
        InputStream in = System.in;
        StringBuilder sb = new StringBuilder();
        while(true)
        {
            int ch = in.read();
            if(ch=='\r')
                continue;
            if(ch=='\n')
            {
                String s = sb.toString();
                if("over".equals(s))
                    break;
                System.out.println(s.toUpperCase());
                sb.delete(0,sb.length());
                else
                    sb.append((char)ch);
            }
        }
    }
```


## 九、转换流
#### 1.读取转换流InputStreamReader
- 字节流--->字符流

(针对八-题)
> 想使用readLine方法来完成键盘录入的一行数据的读取 ；
> 
> readLine方法是字符流BufferedReader类中的方法；
> 
> 而键盘录入的read方法是字节流InputStream的方法；
> 
> 使用转换流，将字节流转成字符流



```
    public static void main(String[] args) throws IOException
    {
        //InoutStream in = System.in;//获取键盘录入对象；
        //InputStreamReader isr = new InputStreamReader(in);//将字节流对象转成字符流对象；
        //BufferedReader bufr = new BufferedReader(isr);
        //以上三句写为：
        BufferedReader bufr = new BufferedReadere(new InputStreamReader(System.in));
        String line = null;
        while((line=bufr.readLine())!=null)
        {
            if("over".equals(line))
                break;
            System.out.println(line.toUpperCase());
        }
        bufr.close();
    }
```

- **记住键盘录入的最常见写法：**

==BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));==
#### 2.写入转换流 OutputStreamWriter
- 字符流--->字节流



```
    BufferedWriter bufw = new BufferedWriter(new OutputStreamWriter(System.out));
    String line = null;
    while((line=bufr.readLine())!=null)
    {
        if("over".equals(line))
            break;
        bufw.write(line.toUpperCase());
        bufw.newLine();
        bufw.flush();
    }
    bufw.close();
```

    