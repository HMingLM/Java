[toc]
# JSP

## 1. jsp环境搭建及入门

### 1.1 JSP：动态网页

- 动态：是否随着时间、地点、用户操作的改变而改变；
- 动态网页需使用服务端脚本语言（JSP）；

### 1.2 架构

##### 1.2.1 CS（Client Server）

不足：
- 若软件升级，则全部软件都需升级；
- 维护麻烦：需维护每一台客户端软件；
- 每一台客户端都需安装客户端软件；

##### 1.2.2 BS（Broswer Server）
客户端可通过浏览器直接访问服务端；

### 1.3 tomcat解压后目录
- bin：可执行文件（startup.bat/shutdown.bat）
- conf:配置文件（server.xml）
- lib:tomcat依赖的jar文件
- log:日志文件（记录出错等信息）
- temp：临时文件
- webapps：可执行的项目（将我们开发的项目放入该目录）
- work：存放由jsp翻译成的java以及编译成的class文件（jsp-java-class）

### 1.4 配置tomcat
- 配置jdk（必须配置JAVA_HOME）
- 配置catalina_home

> 双击bin/startup.bat启动tomcat

> tomcat端口号默认8080（较常见，易冲突），建议修改（如8888）

### 1.5 访问tomcat

http://localhost:8888/

### 1.6 常见状态码
- 200：一切正常
- 300/301：页面重定向（跳转）
- 404：资源不存在
- 403：权限不足
- 500：服务器内部错误（代码有误）
- 其他编码：积累


### 1.7 创建项目

- 在webapps下创建自己项目JspProject，
JspProject下是WEB-INF文件夹，该文件夹下有classes，lib文件夹以及web.xml。
- 在JspProject下写index.jsp
如

```
<html>
    <head>
        <title>my jsp project</title>
    </head>
    
    <body>
        hello jsp...
        <%
            out.print("hello world...");
        %>
    </body>
</html>
```


> jsp：在html中嵌套的Java代码

> 在项目/WEB-INF/web.xml中设置默认的初始页面

```
<welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

## 2. 虚拟路径和虚拟主机

### 2.1 虚拟路径

将web项目配置到webapps（默认虚拟路径）以外的目录

##### 2.1.1 方式一（需重启）

```
conf/server.xml中配置

host标签中：

<Contextt  docBase="D:\study\JspProject"  path="/JspProject"  />
```
- docBase:实际路径；
- path：虚拟路径（相对路径【相对于webapps】、绝对路径）

##### 2.1.2 方式二

```
D:\study\apache-tomcat-8.5.30\conf\Catalina\localhost

中新建  “项目名.xml”   中新增一行：

<Contextt  docBase="D:\study\JspProject"  path="/JspProject"  />
```

### 2.2 虚拟主机

通过www.test.com访问本机

```
第一步   conf/server.xml

<Engine name="Catalina" defaultHost="www.test.com">
    <Host appBase="D:\study\JspProject" name="www.test.com">
        <Context docBase="D:\study\JspProject"  path="/"  />
    </Host>
    
第二步   C:\Windows\System32\drivers\etc\host
增加
127.0.0.1       www.test.com
```

流程：

www.test.com -> host找映射关系 -> server.xml找Engine的defaultHost -> 通过"/"映射到D:\study\JspProject

## 3. JSP执行流程

jsp-->java(Servlet文件)-->class【jsp和Servlet可以相互转换】

> 存放于D:\study\apache-tomcat-8.5.30\work\Catalina\localhost\JspProject\org\apache\jsp

- 第一次访问：服务端将jsp翻译成java，再将java编译成class文件
- 第二次访问：直接访问class文件
- （如果服务端代码修改了，将会在访问时重新翻译、编译）

## 4. 使用Eclipse开发JSP

### 4.1 在eclipse中创建web项目

- 浏览器可以直接访问WebContent中的文件，如http://localhost:8888/MyJspProject/index1.jsp
> 其中index1.jsp就在WebContent目录中；
- 但WEB-INF中的文件无法通过客户端（浏览器）直接访问，只能请求转发来访问
> 注意：并非任何的内部跳转都能访问WEB-INF，原因是跳转有2种方式：请求转发、重定向

### 4.2 配置tomcat运行时环境

jsp<->Servlet

- 方法1：将tomcat/lib中的servlet-api.jar加入项目的构建路径
- 方法2：右键项目->Build Path -> Add library -> Server Runtime

### 4.3 部署tomcat

在servers面板新建一个tomcat实例，再在该实例中部署项目（右键-add）之后运行
> 注意：一般建议将eclipse中的tomcat与本地tomcat的配置信息保持一致：将eclipse中的tomcat设置为托管模式（第一次创建tomcat实例之后，双击，选择Server Location的第二项）

### 4.4 统一字符集编码

编码分类：
- 设置jsp文件的编码（jsp文件中的pageEncoding属性）：jsp->java
- 设置浏览器读取jsp文件的编码（jsp文件中content属性）
> 一般将上述设置成一致的编码，推荐使用UTF-8
- 文本编码：
> i.将整个eclipse中的文件统一设置（推荐）
> 
> ii.设置某一项目
> 
> iii.设置单独文件

## 5. JSP的页面元素

JSP的页面元素：HTML、java代码（脚本Scriptlet）、指令、注释

### 5.1 脚本（Scriptlet）

i.

```
<%
        局部变量、java语句
%>
```

ii.
```
<%!
        全局变量、定义方法
%>
```

iii.
```
<%= 输出表达式 %>
```

- 一般而言，修改web.xml、配置文件、java，需要重启tomcat服务
- 但是若修改了JSP\html\css\js，不需要重启
- 注意，out.println()不能回车，要回车需打
```
<br/>
```
即out.print()，<%= %>可**直接解析html代码**。

### 5.2 指令

page指令

```
<%@ page... %>
```

page指定的属性：
- language：jsp页面使用的脚本语言
- import：导入类
- pageEncoding：jsp文件自身编码  jsp->java
- contentType:浏览器解析jsp的编码
- 
```
<%@ page language="java"  contentType="text/html;charset=UTF-8"
    pageEncoding="UTF-8"  import="java.util.Date"  %>
```

### 5.3 注释

##### 5.3.1 html注释
```
<!-- -->
可以被客户通过浏览器查看源码所查看到
```

##### 5.3.2 java注释

```
//
/*   */
```

##### 5.3.3 jsp注释

```
<%-- --%>
```

## 6. JSP九大内置对象

内置对象：自带的，不需要new也能使用的对象

- request：请求对象；存储“客户端向服务端发送的请求信息”
- response：响应对象
- session：会话对象
- application：全局对象
- config：配置对象（服务器配置信息）
- out：输出对象，向客户端输出内容
- page：当前JSP页面对象（相当于java中的this）
- pageContext：JSP页面容器
- exception：异常对象


### 6.1 request对象

##### 6.1.1 常见方法

- String getParameter(String name) :根据请求的字段名key（input标签的name属性），返回字段值value（input标签的value属性值）
- String[] getParameterValues(String name) :根据请求的字段名key，返回多个字段值value （如checkbox-->多选）
- void setCharacterEncoding("编码格式UTF-8") :设置**post方式**的请求编码（tomcat7以前默认iso-8859-1,tomcat8以后默认UTF-8）
- getRequestDispatcher("b.jsp").forward(request,response) : “请求转发”的方式跳转页面 a->b
- ServletContext getServerContext() :获取项目的ServletContext对象

##### 6.1.2 注册示例

register.jsp  


```
<body>
	<form action="show.jsp" method="post">
	用户名：<input type="text" name="uname" /> <br/>
	密码：<input type="password" name="upwd" /> <br/>
	年龄：<input type="text" name="uage" /> <br/>
	爱好<br/>
	<input type="checkbox" name="uhobbies" value="篮球" />篮球、
	<input type="checkbox" name="uhobbies" value="足球" />足球、
	<input type="checkbox" name="uhobbies" value="羽毛球" />羽毛球<br/>
	<input type="submit"  value="注册" />
	</form>
</body>
```

show.jsp

```
<body>
	<%
		request.setCharacterEncoding("UTF-8") ;   //设置编码
		String name = request.getParameter("uname") ;
		String pwd = request.getParameter("upwd") ;
		int age = Integer.parseInt( request.getParameter("uage") ) ;
		String[] hobbies = request.getParameterValues("uhobbies") ;
	%>
	
	注册成功，信息如下：<br/>
	姓名：<%=name %><br/>
	密码：<%=pwd %><br/>
	年龄：<%=age %><br/>
	爱好：<br/>
	<%
		if(hobbies!=null){
			for(String hobby :hobbies){
				out.print(hobby + "&nbsp;" ) ;
			}
		}
	%>
</body>
```

get提交方式：
- method="get"
- 地址栏
- 超链接<a href=""
- 默认

get与post请求方式的区别：
- get方式在地址栏显示请求信息；post不显示（但是地址栏能够容纳的信息有限[4-5kb]；若请求数据存在大文件，图片等，地址栏无法容纳全部数据而会出错）

```
http://localhost:8888/MyJspProject/show.jsp?uname=xiaoming&upwd=aishaomin&uage=3&uhobbies=%E7%AF%AE%E7%90%83&uhobbies=%E7%BE%BD%E6%AF%9B%E7%90%83

链接/文件？参数名1=参数值1 & 参数名2=参数值2 & 参数名3=参数值3 
```

- 文件上传操作，必须是post
推荐使用post

##### 6.1.3 统一请求的编码

###### 6.1.3.1 get请求方式

若出现乱码

- 方法1

统一每个变量的编码（不推荐）
```
new String( 旧编码，新编码 )；
name = new String(name.getBytes("iso-8859-1"),"UTF-8");
```

- 方法2

修改server.xml，一次性的更改tomcat默认get提交方式的编码（UTF-8），建议使用tomcat时首先进行

```
URIEncoding="UTF-8"
```

###### 6.1.3.2 post请求方式


```
request.setCharacterEncoding("UTF-8") ;
```

### 6.2 response对象

##### 6.2.1 常见方法

- voidaddCookie( Cookie cookie ) :服务端向客户端增加cookie对象
- void sendRedireact( String location ) throw IOEXception :页面跳转的一种方式（重定向）
- void setContentType( String type ) :设置服务端响应的编码（设置服务端的contentType类型）

##### 6.2.2 登陆示例

login.jsp

```
<body>
	<form action="check.jsp" method="post">
		用户名：<input type="text" name="uname" /><br/>
		密码：<input type="password" name="upwd" /><br/>
		<input type="submit" value="登录" /><br/>
	</form>
</body>
```

check.jsp

```
<body>
	<%
		request.setCharacterEncoding("UTF-8") ;
		String name = request.getParameter("uname") ;
		String pwd = request.getParameter("upwd") ;
		if(name.equals("zs") && pwd.equals("abc")){
			//response.sendRedirect("success.jsp") ;  //重定向跳转页面，导致数据丢失
			request.getRequestDispatcher("success.jsp").forward(request,response) ;  
			//请求转发跳转页面,可获取到数据，且地址栏不变
		}
		else{
			out.print("用户名或密码有误！") ;
		}
	%>
</body>
```

success.jsp

```
<body>
	登录成功！<br/>
	欢迎您：
	<%
		String name = request.getParameter("uname") ;
		out.print(name) ;
	%>
</body>
```

##### 6.2.3 请求转发与重定向


-- | 请求转发 | 重定向
--- | --- | ---
地址栏是否改变 | 不变（check.jsp）| 改变（success.jsp）
是否保留第一次请求时的数据 | 保留 | 不保留
请求的次数 | 1 | 2
跳转发生的位置 | 服务端 | 客户端发出的第二次跳转

- 转发

> 张三（客户端） -> 【服务窗口A -> 服务窗口B 】

- 重定向

> 张三（客户端） -> 服务窗口A -> 去找B
> 
> 张三（客户端） -> 服务窗口B -> 结束

### 6.3 seeeion对象（服务端）

##### 6.3.1 Cookie

- Cookie:客户端，不是内置对象；由服务端产生，再发送给客户端保存
- 作用：相当于本地缓存；提高访问服务端的效率，但是安全性较差
- Cookie:name=value（javax.servlet.http.Cookie）
- cookie要使用必须new，但服务端会自动生成一个name=JSESSIONID的cookie（服务端自动new一个cookie），并返回给客户端；

###### 6.3.1.1 方法：
- public Cookie(String name,String value):构造方法
- String getName():获取name
- String getValue():获取value
- void setMaxAge(int expiry):最大有效期（秒）

###### 6.3.1.2 流程：
- 服务端准备Cookie：**response.addCookie(Cookie cookie)**
- 页面跳转（转发/重定向）
- 客户端获取cookie：**request.getCookies()**;

###### 6.3.1.3 注意：
- 服务端增加cookie：response对象；客户端获取对象：requesst对象
- 不能直接获取某一个单独对象，只能一次性将全部的cookie拿到
- 除了自己设置的Cookie对象外，还有一个name为JSESSIONID的cookie
- 建议cookie只保存英文和数字，否则需要进行编码、解码

###### 6.3.1.4 记住用户名示例

login.jsp

```
<body>
	<%!
		String uname;
	%>
	<%
		Cookie[] cookies = request.getCookies();
		for(Cookie cookie :cookies){
			if(cookie.getName().equals("uname")){
				uname = cookie.getValue();
			}
		}
	%>
	<form action="check.jsp" method="post">
		用户名：<input type="text" name="uname" value="<%=(uname==null?"":uname)%>"/><br/>
		密码：<input type="password" name="upwd" /><br/>
		<input type="submit" value="登录" /><br/>
	</form>
</body>
```

check.jsp

```
<body>
	<%
		request.setCharacterEncoding("UTF-8") ;
		String name = request.getParameter("uname") ;
		String pwd = request.getParameter("upwd") ;
		
		Cookie cookie = new Cookie("uname",name);
		response.addCookie(cookie);
		response.sendRedirect("A.jsp");
	%>
</body>
```

A.jsp(【空】 客户端页面？)

##### 6.3.2 session机制

- session存储在服务端
- session是在同一用户（客户）请求时共享
- session在同一次会话共享
- session：会话（开始->结束）

实现机制：

- 客户端第一次请求服务端时，（jsessionid-sessionid无法匹配），服务端会产生一个session对象（用于保存该客户的信息 ），并且每个session对象都会有一个唯一的sessionId（用于区分其他session）；
- 服务端又会产生一个cookie，且该cookie的name=JSESSIONID，value=服务端session的值；
- 然后服务端会在响应客户端的同时，将该cookie发送给客户端，至此，客户端就有了一个cookie（JSESSIONID），因此客户端的cookie就可以和服务端的session一一对应（JSEESSIONID-sessionID）；
- 客户端第多次请求服务端时：服务端会先用客户端的cookie中的JSESSIONID去服务端的session中匹配sessionid，若匹配成功（cookie‘jsessionid和session‘sessionid），说明此用户不是第一次访问，无需登录；

##### 6.3.3 session方法

- String geiID():获取sessionId；
- boolean isNew():判断是否是新用户（第一次访问）；
- void invalidate():使session失效（退出登录、注销）；

- void setAttribute()
- Object getAttribute()

- void setMaxInactive(秒)：设置最大有效非活动时间;
- int getMaxInactive(秒)：获取最大有效非活动时间；

##### 6.3.4 cookie和session的区别


--- | session | cookie
---|---|---
保存的位置 | 服务端 | 客户端
安全性 | 较安全 | 较不安全
保存的内容 | Object | String


### 6.4 application对象

String getContextPath():虚拟路径
String getRealPath(String name):绝对路径（虚拟路径相对的绝对路径）

### 6.5 四种范围对象

范围从小到大：pageContext-->request-->session-->application

##### 6.5.1 共有的方法
- Object getAttribute(String name):根据属性名，获取属性值
- void setAttribute(String nama,Object obj):设置属性值（新增、修改）
- void removeAttribute(String name):根据属性名，删除对象

##### 6.5.2 范围

- pageContext:当前页面有效；页面跳转后无效
- request:同一次请求有效；请求转发后有效；重定向后无效
- session:同一次会话有效；从登录到退出之间无论怎么跳转都有效；关闭、切换浏览器后无效
- application:全局变量；整个项目运行期间都有效；切换浏览器仍有效；关闭服务、其他项目无效

> JNDI：多个项目共享；重启后仍有效
> 
> 注意：尽量使用最小的范围。因对象范围越大，造成的性能损耗越大


