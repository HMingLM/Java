[toc]
## 一、TCP补充内容
#### 1.TCP-上传图片

#### 2.客户端并发上传图片

- 服务端将每个客户端封装到一个单独的线程中，这样可以同时处理多个客户端请求；
- 明确每一个客户端要在服务端执行的代码，将其放在run方法中；

#### 3.客户端并发登陆

## 二、URL

- **String getFile()**：获取此URL的文件名；
- **String getHost()**：获取此URL的主机名；
- **String getPath()**：获取此URL的路径部分；
- **int getPort()**：获取此URL的端口号；
- **String getProtocoll()**：获取此URL的协议名称；
- **String getQuery()**：返回此URL的已解码的查询组成部分；

创建一个URL对象

> URL url = new URL();//括号中导入链接地址；

打开链接

> URLConnection conn = url.openConnection();  

获取输入流

> InputStream inStream = conn.getInputStream();  


    	 