[toc]
# 网络编程

## 1. 网络模型
- OSI参考模型
- TCP/IP参考模型(应用层-传输层-网际层-主机至网络层)

## 2. 网络通讯三要素
- IP地址：找到对方IP地址； 
- 端口号：数据要发送到对方指定的应用程序上；为了标识这些应用程序，给它们都用数字进行标识，这个数字即端口；(逻辑端口)
- 传输协议：定义通信规则(协议)；(国际组织通用协议 TCP/IP)

#### 2.1 IP地址
封装成InetAddress对象

```
public static void main(String[] args) throws Exception
{
    //InetAddress i = InetAddress.getLocalHost();
    InetAddress ia = InetAddress.getByName("www.baidu.com");
    System.out.println(ia.getHostAddress());
    System.out.println(ia.getHostName());
}
```


#### 2.2 端口号
用于标识进程的逻辑地址，不同进程的标识;

有效端口：0 ~ 65535，其中0 ~ 1024系统使用或保留端口

#### 2.3 传输协议

常见协议：TCP，UDP

##### UDP
- 将数据及源和目的封装成数据包中，不需要建立连接；
- 每个数据包的大小都限制再64k内(分包)；
- 因无连接，是不可靠协议；
- 不需要建立连接，速度快；

##### TCP
- 建立连接，形成传输数据的通道；
- 在连接中进行大数据量传输；
- 通过三次握手完成连接，是可靠协议；
- 必须建立连接，效率会稍低；


## 3. Socket

- Socket就是为网络服务提供的一种机制；
- 通信的两端都有Socket；
- 网络通信其实就是Sockeet间的通信；
- 数据在两个Socket间通过IO传输；

## 4. UDP传输

- DatagramSocket与DatagramPacket
- udp传输的步骤：
> 建立发送端，接收端；
> 
定义udpSocket服务时，接收端通常会监听一个端口，其实就是给这个接收网络应用程序定义数字标识，便于明确哪些数据过来该程序可以处理；
> 
> 建立数据包；
> 
> 调用Socket的发送接收方法；
> 
> 关闭Socket；

发送端与接收端是两个**独立的运行程序**；

#### 4.1 Udp-发送端

- **DatagramSocket**：此类表示用来发送和接收数据报包的Socket；
- **send(DatagramPacket p)**:发送数据报包；
- **DatagramPacket**：此类表示数据报包，数据报包用来实现无连接包投递服务；
- **DatagramPacket(byte[] buf,int length,InetAddress address,int port)**:构造数据报包，用来将长度为length的包发送到指定主机的指定端口号；

通过udp传输方式，将一段文字数据发送出去
> 步骤：

> i.建立udpsocket服务；通过DatagramSocket对象；
> 
> ii.提供数据，并将数据封装到数据包中；
> 
> iii.通过socket服务的发送功能send，将数据包发出去；
> 
> iiii.关闭资源；

```
import java.net.*;

class UdpSend
{
    public static void main(String[] args) throws Exception
    {
        DatagramSocket ds = new DatagramSocket();//i
        
        byte[] buf = "udp lai le".getBytes();//ii
        DatagramPacket dp = new DatagramPacket(buf,buf.length,InetAddress.getByName("192.168.1.254"),10000);
        
        ds.send(dp);//iii
        
        ds.close();//iiii
    }
}
```

#### 4.2 Udp-接收端

- **receive(DatagramPacket p)**:接收数据报包；
- **DatagramPacket(byte[] buf,int length)**:构造DatagramPacket，用来接收长度为length的数据包；

数据包的方法
- **InetAddress getAddress()**:返回某台机器的Ip地址(对象)，此数据报将发往该主机或是从该主机接收到的;
- **byte[] getData()**:返回数据缓冲区；
- **int getLength()**:返回将发送的或接收到的数据的长度；
- **int getPort()**:返回某台远程主机的端口号，此数据报将发往该主机或是从该主机接收到的；

定义一个应用程序，用于接收udp协议传输的数据并处理
> 步骤：

> i.建立udpsocket服务；通过DatagramSocket对象建立端点；
> 
> ii.定义数据包，存储接收到的字节数据，因数据包对象中有更多功能可提取字节数据中的不同数据信息；
> 
> iii.通过socket服务receive方法将收到的数据存入定义好的数据包中；
> 
> iiii.通过数据包对象的方法获取其中的数据；打印在控制台上；
> 
> iiiii.关闭资源；

```
import java.net.*;

class UdpRece
{
    public static void main(String[] args) throws Exception
    {
        DatagramSocket ds = new DatagramSocket(10000);//i
        while(true)
        {
            byte[] buf = new byte[1024];//ii
            DatagramPacket dp = new DatagramPacket(buf,buf.length);
            
            ds.receive(dp);//iii//阻塞式方法；
            
            String ip = dp.getAddress().getHostAddress();//iiii
            String data = new String(dp.getData(),0,dp.getLength());
            int port = dp.getPort();
            System.out.println(ip+"::"+data+"::"+port);
        }
        //ds.close();//iiiii
    }
}
```
#### 4.3 键盘录入方式数据

```
import java.net.*;
import java.io.*;

class Udpsend2
{
    public static void main(String[] args) throws Exception
    {
        DatagramSocket ds = new DatagramSocket();
        BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
        String line = null;
        while((line=bufr.readLine())!=null)
        {
            if("886".equals(line))
                break;
            byte[] buf = line.getBytes();
            DatagramPacket dp = new DatagramPacket(buf,buf.length,InetAddress.getByName("192.168.1.254"),10001);
            ds.send(dp);
        }
        ds.close();
    }
}
class UdpRece2
{
    ......
}
```

#### 4.4 聊天
聊天程序有收数据的部分和发数据的部分，两部分同时执行，用到多线程技术；

```
import java.net.*;
import java.io.*;
class Send implements Runnable
{
    private DatagramSocket ds;
    public Send(DatagramSocket ds)
    {
        this.ds = ds;
    }
    public void run()
    {
        try
        {
            BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
            String line = null;
            while((line=bufr.readLine())!=null)
            {
                ...
            }
        }
        catch(Exception e)
        {
            throw new RuntimeException("发送端失败")；
        }
    }
}
class Rece implements Runnable
{
    private DatagramSocket ds;
    public Rece(DatagramSocket ds)
    {
        this.ds = ds;
    }
    public void run()
    {
        try
        {
            while(true)
            {
                ...
            }
        }
        catch(Exception e)
        {
            throw new RuntimeException("接收端失败")；
        }
    }
}
class ChatDemo
{
    public static void main(String[] args) throws Exception
    {
        DatagramSocket sendSocket = new DatagramSocket();
        DatagramSocket receSocket = new DatagramSocket(10002);
        new Thread(new Send(sendSocket)).start();
        new Thread(new Rece(receSocket)).start();
    }
}
```

## 5.TCP传输
- 客户端对应的对象Socket和服务端对应的对象ServerSocket
> TCP传输的步骤：

> 建立客户端和服务器端；
> 
> 建立连接后，通过Socket中的IO流进行数据的传输；
> 
> 关闭Socket；

同样，客户端与服务器端是两个**独立的运行程序**；

#### 5.1 客户端
Socket：此类实现客户端套接字，套接字是两台机器间通信的端点；

因为TCP是面向连接的，所以建立Socket服务时就要有服务端存在，并连接成功，形成通路后进行数据传输；
- **Socket(InetAddress address,int port)**:创建一个流套接字并将其连接到指定IP地址的指定端口号；
- **Socket(String host,int port)**:创建一个流套接字并将其连接到指定主机的指定端口号；
- **getInputStream()**:返回此套接字的输入流；
- **getOutputStream()**:返回此套接字的输出流；





给服务端发送一个文本数据
> 思路：

> i.创建客户端的Socket服务，并指定要连接的主机和端口；
> 
> ii.为了发送数据，应该获取Socket流中的输出流；并传输数据；
> 
> iii.关闭Socket；

```
import java.io.*;
import java.net.*;
class TcpClient
{
    public static void main(String[] args) throws Exception
    {
        Socket s = new Socket("192.168.1.254",10003);//i
        
        OutputStream out = s.getOutputStream();//ii
        out.write("tcp lai le".getBytes());
        
        s.close();//iii
    }
}
```

#### 5.2 服务器端

- **ServerSocket(int port)**:创建绑定到特定端口的服务器套接字；
- **Socket accept()**:监听并接收到此套接字的连接；
- **InetAddress getInetAddress()**:返回此套接字连接的地址；
定义端点接收数据并打印在控制台上

> 思路：

> i.建立服务端的socket服务；并监听一个端口；
> 
> ii.获取连接过来的客户端对象；通过ServerSocket的accept方法(阻塞式，没有连接会等);
> 
> iii.客户端如果发过来数据，那么服务端要使用对应的客户端对象，并获取到该客户端对象的读取流来读取发过来的数据；并打印在控制台；
> 
> iiii.关闭服务端；(可选)

```
class TcpServer
{
    public static void main(String[] args) throws Exception
    {
        ServerSocket ss = new ServerSocket(10003);//i
        
        Socket s = ss.accept();//ii
        String ip = s.getInetAddress().getHostAddress();//通过客户端对象获取IP地址
        System.out.println(ip+"....connected");
        
        InputStream in = s.getInputStream();//iii
        byte[] buf = new byte[1024];
        int len = in.read(buf);
        System.out.println(new String(buf,0,len));
        
        s.close();//关闭客户端；
        ss.close();
    }
}
```

#### 5.3 客户端和服务端的互访

##### 题一
客户端给服务端发送数据，服务端收到后，给客户端反馈信息

```
class TcpClient2
{
    public static void main(String[] args) throws Exception
    {
        Socket s = new Socket("192.168.1.254",10004);
        OutputStream out = s.getOutputStream();
        out.write("服务端，你好".getBytes());
        InputStream in = s.getInputStream();
        byte[] buf = new byte[1024];
        int len = in.read(buf);
        System.out.println(new String(buf,0,len));
        s.close();
    }
}
class TcpServer2
{
    ServerSocket ss = new ServerSocket(10004);
    Socket s = ss.accept();
    String ip = s.getInetAddress().getHostAddress();
    System.out.println(ip+"....connected");
    InputStream in = s.getInputStream();
    byte[] buf = new byte[1024];
    int len = in.read(buf);
    System.out.println(new String(buf,0,len));
    OutputStream out = s.getOutputStream();
    out.write("收到，你也好".getBytes());
    s.close();
    ss.close();
}
```

##### 题二
客户端给服务端发送文本，服务端将文本转成大写返回给客户端，可不断进行转换，客户端输入over时转换结束

```
class TransClient
{
    public static void main(Stirng[] args)
    {
        Socket s = new Socket("192.168.1.254",10005);
        BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
        //BufferedWriter bufOut = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        PrintWriter out = new PrintWriter(s.getOutputStream(),true);
        BufferedReader bufIn = new BufferedReader(new InputStreamReader(s.getInputStream()));
        String line = null;
        while((line=bufr.readLine())!=null)
        {
            if("over".equals(line))
                break;
            //bufOut.write(line);
            //bufOut.newLine();
            //bufOut.flush();
            out.println(line);
            String str = bufIn.readLine();
            System.out.println("server:"+str);
        }
        bufr.close();
        s.close();
    }
}
class TransServer
{
    public static void main(String[] args)
    {
        ServerSocket ss = new ServerSocket(10005);
        Socket s = ss.accept();
        String ip = s.getInetAddress().getHostAddress();
        System.out.println(ip+"....connected");
        BufferedReader bufIn = new BufferedReader(new InputStreamReader(s.getInputStream()));
        //BufferedWriter bufout = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        PrintWriter out = new PrintWriter(s.getOutputStream(),true);
        String line = null;
        while((line=bufIn.readLine())!=null)
        {
            System.out.println(line);
            //bufOut.write(line.toUpperCase());
            //bufOut.newLine();
            //bufOut.flush();
            out.println(line.toUpperCase());
        }
        s.close();
        ss.close();
    }
}
```

其中要注意newLine方法，因为客户端和服务端都有阻塞式方法，没有读到结束标记会导致两端都一直等待；

##### 题三
TCP复制文件；

客户端发送文件数据给服务器端，服务器端将数据保存到文件中；

```
class TextClient
{
    public static void main(String[] args)
    {
        Socket s = new Socket("192.168.1.254",10006);
        BufferedReader bufr = new BufferedReader(new FileReader("IPDemo.java"));
        PrintWriter out = new PrintWriter(s.getOutputStream(),true);
        String line = null;
        while((line=bufr.readLine())!=null)
        {
            out.println(line);
        }
        s.shutdownOutput();
        BufferedReader bufIn = new BufferReader(new InputStreamReader(s.getInputStream()));
        String str = bufIn.readLine();
        System.out.println(str);
        bufr.close();
        s.close();
    }
}
class TextServer
{
    public static void main(String[] args)
    {
        ServerSocket ss = new ServerSocket(10006);
        Socket s = ss.accept();
        String ip = s.getInetAddress().getHostAddress();
        System.out.println(ip+"....connected");
        BufferedReader bufIn = new BufferedReader(new InputStreamReader(s.getInputStream()));
        PrintWriter out = new PrintWriter(new FileWriter("server.txt",true);
        String line = null;
        while((line=bufIn.readLine())!=null)
        {
            out.println(line);
        }
        PrintWriter pw = new PrintWriter(s.getOutputStream(),true);
        pw.println("上传成功");
        out.close();
        s.close();
        ss.close();
    }
}
```

Socket类的**shutdownOutput**()方法：关闭客户端的输出流；相当于给流中加入一个结束标记-1；

另外也有**shutdownInput**方法；