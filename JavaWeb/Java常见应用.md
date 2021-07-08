[toc]

# 反射

- 反射机制是在【运行状态】中：
  - 对于任意一个类，都能知道这个类的所有属性和方法；
  - 对于任意一个对象，都能调用它的任意一个方法和属性；
  
- 反射提供的功能：
  - 在运行时判断任意一个对象所属的类；
  - 在运行时构造任意一个类的对象；
  - 在运行时判断任意一个类所具有的成员变量和方法；
  - 在运行时调用任意一个对象的方法；
  - 生成动态代理；

- 获取反射对象（反射入口）：class
  - a、Class<?> xxx = Class.forName("全类名");
  - b、Class<?> xxx = 类名.class ;
  - c、Class<?> xxx = 对象.getClass() ;

- 获取所有的公共的方法
  - Method[] methods = xxx.getMethods();
  - 本类以及父类、接口中的所有方法
  - 符合访问修饰符规律

- 获取特定方法
  - Method method = clazz.getMethod("方法名"，参数类);
  - 通过method.invoke(对象,参数) 执行方法

- 获取当前类的所有方法
  - Method[] declaredMethods = xxx.getDeclaredMethods() ;
  - 本类
  - 忽略访问修饰符限制

- 获取所有的接口
  - Class<?>[] interfaces = xxx.getInterfaces();
  
- 获取所有的父类（1个）
  - Class<?> superclass = xxx.getSuperclass();  

- 获取所有的构造方法
  - Constructor<?>[] constructors = xxx.getConstructors() ;

- 获取所有的公共属性
  - Field[] fields = xxx.getFields() ;

- 获取所有属性
  - Field[] declaredFields = xxx.getDeclaredFields() ;

- 获取当前反射所代表类/接口的对象/实例
  - Object instance = xxx.newInstance() ;


# RPC

RPC：Remote Procedure Call

1. 客户端
2. 发布服务的接口
3. 服务的注册中心

- 反射技术：服务端通过客户端传递的字符串解析出该字符串代表的接口的一切信息
- socket：客户端-服务端通信
- 动态代理：服务端需要根据客户端的不同请求，返回不同的接口类型，客户端要接受到不同的接口类型

- 流程：
  - 客户端通过socket请求服务端，并且通过字符串形式将需要请求的接口发送给服务端
  - 服务端将可以提供的接口注册到服务中心（通过map保存 key为接口名字 value为接口的实现类）
  - 服务端接收到客户端的请求后，通过请求的接口名在服务中心的map中寻找对应的接口实现类，找到后解析刚才客户端发送来的接口名、方法名，解析完毕后通过反射技术将该方法执行，执行完毕后再将该方法的返回值返回给客户端


# socket

- 基于TCP的Socket传输
  - 服务端绑定端口【new ServerSocket(端口号)】
  - 客户端new Socket（ip地址，端口号）
  - 服务端ServerSocket的accept()用于监听客户端访问，返回一个Socket对象
  - 服务端、客户端用Socket的InputStream read和OutputStream write发送、接收信息
  - 传输文件时，new一个输入流将文件从硬盘传输到内存，new一个输出流将文件从内存传输到硬盘
  - 文件传输优化：加入多线程；【线程处理类MyDownload，new Thread(new MyDownload(socket)).strat()】
  

# 文件拆分合并

- 将一个文件拆分成若干个子文件
- 将拆分后的文件合并成源文件

> 查询当前系统的换行符：String lineSeparator = system.getProperty("line.separator") ;


- 序列化 反序列化【对象流ObjectOutputStream/ObjectInputStream】
  - 序列化/反序列化对象的操作过程都依赖对象所在的类【必须实现Serializable】


# json

> 下载json库（http://www.json.org/ 找  json-java），引入json【打成jar包】

- Map/JavaBean/String --> Json对象
  - JSONObject json = new JSONObject( Map/JavaBean/String ) ;
- 文件 --> Json对象
  - 思路：文件 -->  String --> Json对象
- 生成json文件（json.write()方法）

> json数组外层是中括号

- JsonArray
  - String格式的json数组->json数组：JSONArray jArray = new JSONArray( String );
  - String格式的json数组->json数组：需引入另一个json库，使用fromObject()方法【查json-lib，功能强大，全部可用】


# 二维码

### QRCode（日本）

Test.java
```
String imgPath = "src/二维码.png" ;     //生成图片的路径
//String content = "helloword" ;      //文字信息、网址信息
//String content = "https://www.baidu.com" ;   

//生成二维码
QRCodeUtil qrUtil = new QRCodeUtil();

//加密：文字信息--》二维码
qrUtil.encoderQRCode(content,imgPath,"png",17) ;

//解密：二维码-->文字信息
String imgContent = qrUtil.decoderQRCode(imgPath) ;
```

QRCodeUtil.java
```
public class QRCodeUtil{
    public void encoderQRCode(String content,String imgPath,String imgType,int Size){
        BufferedImage bufImg = qrcodeCommon(content,imgType,size) ;
        File file = new File(imgPath) ;
        //生成图片
        ImageIO.write(bufImg,imgType,file) ;
    }
    
    public BufferedImage qrcodeCommon(String content,String imgType,int size){
        BufferedImage bufImg = null ;
        //Qrcode对象：字符串-> boolean[][]
        Qrcode qrCodeHandler = new Qrcode();
        //设置二维码的排错率：L（7%）< M < Q < H（30%）；排错率越高，可存储的信息越少，，但是对二维码清晰的要求越小
        qrCodeHandler.setQrcodeErrorCorrect('M');
        //可存放的信息类型：N（数字）、A（数字+A-Z）、B（所有）
        qrCodeHandler.setQrcodeEncodeMode('B');
        //尺寸：取值范围1-40
        qrCodeHandler.setQrcodeVersioni(size);
        
        byte[] contentBytes = content.getBytes("UTF-8") ;
        boolean[][] codeOut = qrCodeHandler.calQrcode(contentBytes);
        
        int imgSize = 67 + 12*(size-1) ;
        //BufferedImage:内存中的图片
        bufImg = new BufferedImage(imgSize,imgSize,BufferedImage.TYPE_INT_RGB ); 
        //创建一个画板
        Graphics2D gs = bufImg.createGraphics();
        gs.setBackground(Color.WHITE) ; //将画板的背景色设置为白色
        gs.clearRect(0,0,imgSize,imgSize) ; //初始化
        gs.setColor(Color.BLACK) ;  //设置画板上的图像（二维码）的颜色
        
        int pixoff = 2 ;
        for(int j=0;j<codeOut.length;j++)
        {
            for(int i=0;i<codeOut.length;i++)
            {
                if(codeOut[j][i])
                    gs.fillRect(j*3+pixoff,i*3+pixoff,3,3);
            }
        }
        
        //增加LOGO
        Image logo = ImageIO.read(new File("src/logo.png") );
        int maxHeight = bufImg.getHeight();
        int maxWidth = bufImg.getWidth();
        //在已生成的二维码上画logo
        gs.drawImage(logo,imageSize/5*2,imageSize/5*2,maxWidth/5,maxHeight/5,null);
        
        gs.dispose();   //释放空间
        bufImg.flush(); //清理
        return bufImg;
    }

    //解密：二维码（图片路径）-->文字信息
    public String decoderQRCode(String imgPath) throws Exception{
        //BufferedImage内存中的图片
        BufferedImage bufImg = ImageIO.read(new File(imgPath));
        
        //解密
        QRCodeDecoder decoder = new QRCodeDecoder() ;
        
        TwoDimensionCodeImage tdcImage = new TwoDimensionCodeImage(bufImg);
        byte[] bs = decoder.decode(tdcImage) ;
        //byte[] --> String
        String content = new String(bs,"utf-8") ;
        return content;
    }
}

```

TwoDimensionCodeImage.java
```
public class TwoDimensionCodeImage implements QRCodeImage{
    BufferedImage bufImg ;  //内存中的二维码
    public TwoDimensionCodeImage(BufferedImage bufImg){
        this.bufImg = bufImg ;
    }
    public int getHeight(){
        return bufImg.getHeight();
    }
    public int getPixel(int x,int y){
        return bufImg.getRGB(x,y);
    }
    public int getWidth(){
        return bufImg.getWidth();
    }
}
```

### ZXing（Google）


# 加密

- 异或【可逆】
  - 同为0，异为1
  - 一个数，两次异或后，是原数本身
  - 加密：一次异或
  - 解密：两次异或

- MD5【字节串-->十六进制串】【不可逆】
  - String DigestUtils.md5Hex(byte[] data)
- SHA256【字节串-->十六进制串】【不可逆】
  - String DigestUtils.sha256(byte[] data)

> MD5速度较快，SHA256安全性较高，均依赖于commons-codec.jar

- BASE64【可逆】
  - 加密：通过反射，用Base64类的encode方法
  - 解密：通过反射，用Base64类的decode方法

> Base64直接依赖于JDK

> 加密解密的应用：登录中的密码


# MAIL


```
public class JavaMail{
    public static void main(String[] args) throws Exception{
        Properties props = new Properties();
        props.setProperty("mail.transport.protocol","smtp");    //使用协议：smtp
        props.setProperty("mail.smtp.host","smtp.qq.com");    //协议地址
        props.setProperty("mail.smtp.port","465");    //协议端口
        props.setProperty("mail.smtp.auth","true");    //需要授权
        //QQ：SSL安全认证
        props.setProperty("mail.smtp.socketFactory.class","javax.net.ssl.SSLSocketFactory");
        props.setProperty("mail.smtp.socketFactory.fallback","false");
        props.setProperty("mail.smtp.socketFactory.port","465");
        
        Session session = Session.getInstance(props);
        session.setDebug(true) ;    //开启日志
        
        //创建邮件
        MimeMessage message = createMimeMessage(session,"928472215@qq.com","xxx@qq.com","yyy@qq.com","zzz@qq.com");
        Transport transport = session.getTransport();   //创建连接对象
        transport.connect("928472215@qq.com","hahltbedjwksbjbc");   //建立连接，其中密码是“授权码”形式
        transport.sendMessage(message,message.getAllRecipients());
        transport.close();
    }
    
    //MimeMessage(带图片+附件的邮件)
    public static MimeMessage createMimeMessage(Session session,String send,String receive,String,creceive,String,mreceive){
        MimeMessage message = new MimeMessage(session) ;
        //邮件：标题、正文、发件人、收件人{图片、附件}
        Address address = new InternetAddress(send,"发件人的名字","UTF-8");
        message.setFrom(address);
        message.setSubject("这是标题（还有图片+附件）","UTF-8");
        
        //创建图片节点
        MimeBodyPart imagePart = new MimeBodyPart();
        DataHandler imageDataHandler = new DataHandler(new FileDataSource("src/JVM.png"));
        imagePart.setDataHandler(imageDataHandler);
        imagePart.setContentID("myJVMImage");
        
        //创建文本节点：目的是加载图片节点
        MimeBodyPart textPart = new MimeBodyPart();
        textPart.setContent("正文内容...NiHao。。image:<img src='cid:myJVMImage' />","text/html;charset=utf-8");
        
        //将文本节点，图片节点组装成一个复合节点
        MimeMultipart mm_text_image = new MimeMultipart();
        mm_text_image.addBodyPart(imagePart);
        mm_text_image.addBodyPart(textPart);
        mm_text_image.setSubType("related");    //关联关系
        
        //【正文】中只能出现普通节点MimeBodyPart不能出现复合节点MimeMultipart
        //MimeMultipart-->MimeBodyPart
        MimeBodyPart text_image_bodyPart = new MimeBodyPart();
        text_image_bodyPart.setContent(mm_text_image);
        
        //附件
        MimeBodyPart attachment = new MimeBodyPart();
        DataHandler fileDataHandler = new DataHandler(new FileDataSource("src/二维码.png"));
        attachment.setDataHandler(fileDataHandler);
        //给附件设置文件名
        attachment.setFileName(MimeUtility.encodeText(fileDataHandler.getName()));
        
        //将刚才处理好的“文本+图片：节点与附件设置成一个新的复合节点
        MimeMultipart mm = new MimeMultipart();
        mm.addBodyPart(text_image_bodyPart);
        mm.addBodyPart(attachment);
        mm.setSubType("mixed");     //混合关系
        message.setContent(mm,"text/html;charset=utf-8");
        
        //收件人类型：TO普通收件人、CC抄送、BCC密送
        message.setRecipient(MimeMessage.RecipientType.To,new InternetAddress(receive,"收件人A","UTF-8"));
        message.setRecipient(MimeMessage.RecipientType.CC,new InternetAddress(creceive,"抄送人B","UTF-8"));
        message.setRecipient(MimeMessage.RecipientType.BCC,new InternetAddress(mreceive,"密送人C","UTF-8"));
        
        message.setSentDate(new Date());    //设置发送时间
        message.saveChanges();  //保存邮件
        
        return message;
    }
}
```


# 中文分词

中科院 分词包ictclas4j
1. 将ictclas4j的Data目录拷贝到项目根目录
2. 将ictclas4j的bin目录覆盖项目根目录中的bin目录
3. 将ictclas4j的src源码复制到项目中
4. 导入需要的commons-lang.jar

```
SegTag tag = new SegTag(1);
SegResult rsult = tag.split("今天天气不错，你想跟我出去玩吗？");
System.out.println(rsult.getFinalResult());
```

# xml解析

### dom

DOM：W3C的标准解析方法；效率较低

### SAX

SAX：事件驱动；扫描开头和结束标签，触发相应的事件【√】

