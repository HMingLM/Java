[toc]
## MVC设计模式
- M：Model，模型；处理业务逻辑/数据；用JavaBean实现。
- V：View，视图；展示、与用户交互；使用HTML/css/jsp/js/jQuery等前端技术实现。
- C：Controller，控制器；接受请求，将请求跳转到模型进行处理，模型处理完毕后，再将处理的结果返回给请求处；可以用jsp实现，但一般建议使用Servlet。

==项目的根目录==：
- WebContent、src【所有的构建路径】就是根目录
- web.xml中的/：代表项目根路径【http://localhost:8888/Servlet25Project/】
- jsp中的/：代表服务器根路径【http://localhost:8888/】

## Servlet

jsp-->Java(Servlet)-->jsp

- servlet是符合一定规范的Java类【可通过eclipse自动生成】：
  - 必须继承javax.servlet.http.HttpServlet
  - 重写其中的doGet()或doPost()方法
  - 必须配置web.xml【Servlet2.5】，或@WebServlet【Servlet3.0】

> doGet()：接受并处理所有get提交方式的请求

> doPost()：接受并处理所有post提交方式的请求

#### Servlet流程【2.5】：
  
- 请求 
- -> <url-pattern>
- 根据<servlet-mapping>中的<servlet-name>去匹配<servlet>中的<servlet-name>
- 找到<servlet-class>，最终交由该<servlet-class>

#### Servlet流程【3.0】：

- 不需要在web.xml中配置，但需要在Servlet类的定义处之上编写注解@WebServlet("url-pattern的值")
- 请求地址与@WebServlet中的值进行匹配，若匹配成功，则说明请求的就是该注解所对应的类

#### Servlet生命周期


- 加载
- 初始化：init()，该方法默认会在Servlet被加载并实例化(即第一次访问servlet时)之后执行，只执行一次
- 服务：  service() -> 具体实现： doGet、doPost，调用几次则执行几次
- 销毁：  destroy()， Servlet被系统回收时（即关闭tomcat服务时）执行，只执行一次
- 卸载

++init()可以修改为在tomcat启动时自动执行：++

servlet2.5：
```
在web.xml中<servlet-class>下加<load-on-startup>1</load-on-startup>,其中1代表第一个servlet
```

servlet3.0：

```
@WebServlet( value="url-pattern的值" , loadOnStartup=1 )
```

#### Servlet API

- 由两个软件包组成：对应于HTTP协议的软件包、对应于除了HTP协议以外的其他软件包——即Servlet API适用于任何通信协议

Servlet继承关系

- ServletConfig接口
  - ServletContext getServletContext()：获取Servlet上下文对象  application
  - String getInitParameter(String name)：在当前Servlet范围内，获取名为name的参数值（初始化参数）

- ServletContext中的常见方法(application)：
  - getContextPath()：相对路径
  - getRealPath()：绝对路径
  - setAttribute()、getAttribute()
  - String getInitParameter(String name)：在当前web容器范围内，获取名为name的参数值（初始化参数）

**out：jsp内置对象**
- 在servlet中使用： 
  - PrintWriter out = response.getWriter();
  - response.getWriter().println("...");
  - 类似的out.write("xxx");【可选参数比out.print("xxx")略少】
  - out为输出流


- session：request.getSession();
- application: request.getServletContext();




## 三层架构

与MVC设计模式的目标一致：都是为了解耦合、提高代码复用；

++注意区别++：二者对项目理解的角度不同；表示层中，前台对应于MVC中的V，后台对应于C，业务逻辑层对应于M中封装业务的JavaBean，数据访问层对应于M中封装数据的JavaBean

**组成：**
- 表示层（USL，User Show Layer；视图层）
  - 前台：对应于MVC中的View，用于和用户交互、界面的显示【jsp、js、html、jquery等web前端技术】
  - 后台：对应于MVC中的Controller，用于控制跳转、调用业务逻辑层【Servlet（SpringMVC Struts2）
- 业务逻辑层（BLL，Business Access Layer；Service层）
  - 接收表示层的请求，调用
  - 组装数据访问层，逻辑性的操作（增删改查，删：查+删）
- 数据访问层（DAL，Data Access Layer；Dao层）
  - 直接访问数据库的操作，原子性的操作（增删改查）

**三层间的关系：**

- 上层将请求传递给下层，下层处理后返回给上层
- 上层依赖于下层，依赖在代码的理解就说持有成员变量

**优化三层:**

- 加入接口
- DBUtil：通用的数据库帮助类，可以简化Dao层的代码量

## 分页

- 实现分页必须知道某一页的数据从哪里开始，到哪里结束
- ==mysql从0开始计数，Oracle/sqlserver从1开始计数==

#### 分页SQL

**MYSQL实现分页的sql：limit 开始，多少条**

```
select * from student limit (当前页-1)*页面大小，页面大小；
```

++页面大小：每页显示的数据量++

**Oracle分页**

```
select * from
(
    select rownum r, t.*from
    (select s.* from student s order by sno asc) t
    where rownum<=n*10
)
where r>=(n-1)*10;
```

- 如果根据sno排序则rownum会混乱（解决方案：分开使用->先只排序，再只查询rownum）

++rownum不能查询>的数据++


**sqlserver分页**

```
select * from 
(
    select row_number() over (sno order by sno asc) as r,* from student
    where r<=页数*页面大小
)
where r>=(页数-1)*页面大小+1 ;


------以上为sqlserver2005后支持，下为sqlserver2003：
select top 页面大小 * from student where id not in
( select top (页数-1)*页面大小 id from student order by sno asc )


------sqlserver2012后支持offset fetch next only:
select * from student order by sno
offset (页数-1)*页面大小+1 rows fetch next 页面大小 rows only;
```

- SQLServer此种分页sql与Oracle分页sql的区别：
  - ①rownum，row_number()
  - ②Oracle需要排序（单独写了一个子查询），但是在sqlserver中可以省略该排序的子查询，因为sqlserver中可以通过over直接排序

#### 分页实现：

5个变量（属性）
- 数据总数【查数据库，select count(*)..】
- 页面大小（每页显示的数据条数）【用户自定义】
- 总页数（数据总数%页面大小==0？数据总数/页面大小：数据总数/页面大小+1）【程序自动计算】
- 当前页（页码）【用户自定义】
- 当前页的对象集合（实体类集合），即每页所显示的所有数据【查数据库，分页sql】


## 上传

- 引入两个jar：commons-fileupload.jar、commons-io.jar【前者依赖于后者】

- 前端代码jsp：
  - 表单提交方式必须为post
  - 在表单中必须增加一个属性：enctype="mulltipart/form-data"
- 后端代码servlet
  - 可限制上传（类型/大小），写在parseRequest之前

```
<input type="file" name="spicture" />
```
> 注意：定义上传的目录时，若该目录为tomcat下，则当修改代码后重新启动tomcat时，该目录被删除。因为修改代码后tomcat会重新编译一份class文件并重新部署（重新创建各种目录）。为防止目录丢失：方法1-定义虚拟路径；方法2-更换目录到非tomcat目录。


## 下载

++不需要任何jar++

- ① 请求Servlet（地址a form）

下载文件需要设置消息头,同时解决各浏览器下载时文件名乱码问题

```
  response.addHeader("content-Type","application/octet-stream");
  //对于不同浏览器分情况处理
  //获取客户端User-Agent信息
  String agent = request.getHeader("User-Agent");
  if(agent.toLowerCase().indexOf(firefox)!=-1){
      //火狐浏览器
      response.addHeader("content-Disposition","attachment;filename==?UTF-8?B?"+ new String( Base64.encodeBase64(filename.getBytes("UTF-8")) ) +"?=" );
  }else{
      //edge浏览器
      response.addHeader("content-Disposition","attachment;filename="+URLEncoder.encode(fileName,"UTF-8"));
  }
 
```

> MIME：多用途互联网邮件扩展类型

文件类型 | Content-Type
---|---
二进制文件（任何类型的文件） | application/octet-stream
Word | application/msword
Excel | application/vnd.ms-excel
PPT | application/vnd.ms-powerpoint
图片 | image/gif,image/bmp,image/jpeg
文本文件 | text/plain
html网页 | text/html


- ② Servlet通过文件的地址，将文件转为输入流，读到Servlet中

```
InputStream in = getServletContext().getResourceAsStream("res/MIME.png")
```

- ③ 通过输出流将刚才已经转为输入流的文件输出给用户

```
ServletOutputStream out = response.getOutputStream();
byte[] bs = new byte[10];
int len = -1 ;
while( (len=in.read(bs)) !=-1 ){
    out.write(bs,0,len);
}
out.close();
in.close();
```


