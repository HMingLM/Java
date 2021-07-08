[toc]

## IDEA创建web项目

### 1. idea中tomcat乱码问题

1. file-settings-file encodings，配置改为UTF-8
2. idea工作目录，在idea64.exe.vmoptions/idea.exe.vmoptions中加一行【-Dfile.encoding=UTF-8】
3. 配置tomcat的页面中，VM option设置【-Dfile.encoding=UTF-8】

> 注：我的IDEA安装目录中，bin下的idea64.exe.vmoptions文件与IDEA不对应，于是在help-edit custom vm optinos中进行；

### 2. 热部署问题

总结：设置Frame为update classes and resources，以debug模式启动

- Update：更新操作（很多时候无效）【推荐：任意】
- Frame：idea失去焦点时触发【推荐：update classes and resources】
- 如果是run启动，仅JSP等静态资源有效
- 如果是debug启动，java也有效

> 编写servlet前需要先加入tomcat环境

> NoCladdDeFoundError异常：缺少jar包

### 3.IDEA中的jar问题

1. Java项目
    
    直接将jar复制到工程中，右键-Add as Library..

2. web项目

> eclipse会将Web-Content/lib中所有的jar存放于项目的全部生命周期

> IDEA会将Web-Content/lib中所有的jar只在运行阶段生效，其他阶段不生效

- 第一种jar【只在运行时有效的jar】（如ojdbc7.jar）
  - 放在WEB-INF中创建的lib目录
  - 在工程结构中 - artifacts - output loaty 
  - 原则上操作二选一，但延迟较长，均进行
- 第二种jar【开发、运行时均有效】（如commons-dbcp.jar）
  - 在第一种jar操作方法的基础上，将jar复制到工程中，右键-Add as Library..，使得开发时也有效







## JNDI

java命名与目录接口，将某一个资源（对象），以配置文件（tomact/conf/context.xml）的形式写入【多项目有效】

> pageContext < request < session < application ：一个项目运行期间有效


1. tomact/conf/context.xml配置

```
<Environment name="jndiName" value="jndiValue" type="java.lang.String" />
```

2. 使用

jsp
```
<%
    Context ctx = new InitialContext();     //类似于拿context.xml文件
    String testJndi = (String) ctx.lookup("java:comp/env/jndiName");
    out.println(testJndi);
%>
```

## 连接池

- 初衷：打开、关闭数据库比较消耗性能
- 常见连接池：Tomcat-dbcp、dbcp、c3p0、druid

> 可以用数据源(javax.sql.DataSource)管理连接池

- 以前指向数据库，现在指向数据源

### 1.Tomcat-dbcp：

1. 在context.xml中配置数据库
```
<Resource name="student" auth="Container" type="javax.sql.DataSource"
    maxActive="400" maxIdle="20" maxWait="5000" username="root" password="Ming@123" driverClassName="com.mysql.cj.jdbc.Driver" 
    url="jdbc:mysql://localhost:3306/warehouse?serverTimezone=UTC"
```

2. 在项目的web.xml中配置
```
<resource-ref>
    <res-ref-name>student</res-ref-name>
    <res-type>javax.sql.DataSource</res-type>
    <res-auth>Container</res-auth>
</resource-ref>
```

3. 使用

DButil.java
```
//传统jdbc方式访问数据库连接
//con = DriverManager.getConnection(URL,USERNAME,PASSWORD);

//数据源方式
Context ctx = new InitialContext();     //类似于拿context.xml文件
DataSource ds = (DataSource) ctx.lookup("java:comp/env/student");
con = ds.getConnection();
```

servlet.java
```
//省略三层
String sql = "delete from student where id = ?"
Object[] params = new Object[]{4};
DButil.executeUpdate(sql,params);
```




### 2.dbcp

引入jar：commons-dbcp-1.4.jar、commons-pool.jar

获取DataSource主要使用的类：BasicDataSource、或BasicDataSourceFactory

1. BasicDataSource方式【硬编码】

```
BasicDataSource dbcp = new BasicDataSource();
dbcp.setDriverClassName("com.mysql.cj.jdbc.Driver");
dbcp.setUrl("jdbc:mysql://localhost:3306/warehouse?serverTimezone=UTC");
dbcp.setUsername("root");
dbcp.setPassword("Ming@123");
dbcp.setInitialSize(20);
dbcp.setMaxActive(10);
```

2. BasicDataSourceFactory方式【配置db.properties文件（key=value）】

```
DataSource dbcp = null;
Properties props = new Properties();
InputStream input = new DBCPDemo().getClass().getClassLoader().getResourceAsStream("db.properties");
props.load(input);

dbcp = BasicDataSourceFactory.createDataSource(props);
```


### 3. c3p0

- ComboPooledDataSource
- c3p0.jar
- 将硬编码和配置文件两种方式合二为一，通过- ComboPooledDataSource的构造方法参数区分：
  - 无参：硬编码
  
```
ComboPooledDataSource c3p0 = new ComboPooledDataSource();
c3p0.set....();
```

  - 有参：配置文件【c3p0-config.xml】


## ApacheDbUtils

下载commons-dbutils-1.7.jar，其中包含的重点类有DbUtils、QueryRunner、ResultSetHandler

- DbUtils：辅助，如关闭资源；
- QueryRunner：增删改查
- ResultSetHandler接口处理查询结果，有很多实现类，一个实现类对应于一种不同的查询结果类型：
  - ArrayHandler：返回结果集中的第一行数据，并用Object[]接收
  - ArrayListHandler：返回结果集中的多行数据，List<Object[]>
  - BeanHandler：返回结果集中的第一行数据，用对象接收
  - BeanListHandler：返回结果集中的多行数据，List<Student>
  - BeanMapHandler： 1:stu1 , 2:stu2 , 3:stu3
  - MapHandler：返回结果集中的第一行数据
  - MapListHandler：返回结果集中的多行数据
  - keyedHandler：将某一列作为map的key，某一行数据作为map的value
  - ColumnListHandler：将结果集中的某一列保存到List中
  - ScalarHandler：单值结果

QueryRunner类的常用方法：
1. QueryRunner构造方法：查，无参需要手动管理事务，有参自动管理事务
2. update：增删改：若手动提交事务则参数列表中需要一个connection，自动不需要

QueryRunner的实现类的参数列表中，最后一个是可变参数：既可以写一个单值，也可以写一个数组；

> 反射会通过无参构造来创建对象


### ThreadLocal

手动提交事务时，【如转账操作】，如果既要保证数据安全，又要保证性能，可以考虑ThreadLocal

ThreadLocal可以为每个线程创建一个副本，每个线程可以访问自己内部的副本。（别名：线程本地变量）

- set()：给ThreadLocal中存放一个变量
- get()：从ThreadLocal中获取变量（副本）
- remove()：删除副本


对于数据库来说，一个连接对应于一个事务，一个事务可以包含多个DML操作

如果给每个dao操作都创建一个connection，则多个dao操作对应于多个事务，但一般来讲，一个业务（service）中的多个dao操作应该包含在一个事务中

使用ThreadLocal，在第一个dao操作时真正创建一个connection对象，之后其他几次dao操作时，借助于ThreadLocal本身特性，自动将该connection复制多个【connection只创建了一个，因此该connection中的所有操作必然对应于同一个事务，并且ThreadLocal将connection在使用层面复制了多个，因此可以完成多个dao操作】


事务流程：
- 开启事务（自动提交->手动提交） 
- 进行各种DML
- 若正常，将刚才所有DML全部提交【全部成功】
- 若失败（异常），将刚才所有DML全部回滚【全部失败】

## 元数据

- 元数据（Metadata）：描述数据的数据
- 三类：
  - 数据库元数据【Connection -> DataBaseMetadata】
  - 参数元数据【pstmt -> ParameterMetadata】
  - 结果集元数据【ResultSet -> ResultSetMatadata】


> 数据库对元数据的支持问题

1. Oracle目前必须使用ojdbc7.jar作为驱动包
2. MySQL必须在url中附加参数配置：
```
jdbc:mysql://localhost:3306/数据库名?generateSimpleParameterMetadata=true
```


## 自定义标签

1. 编写标签处理类

- 如果jsp在编译阶段发现了自定义标签，就会交给doStartTag()或doTag()

2. 编写标签描述符【tld】

建议：可以仿照一个其他标签语言（el/jstl）的tld文件

3. 导入并使用

- 将myTag.tld导入WEB-INF或其子目录下（特例排除：不能是其下lib/classes）
- 导入具体要使用的tld文件<%@ taglib uri="..." prefix="d" %>
  - uri：每个tld文件的唯一描述符
  - prefix：使用tld标签时的前缀


### 传统方式（JSP1.1）

**顶级接口javax.servlet.jsp.tagext.Tag接口**

- 先确保项目有tomcat环境
- doStartTag()是标签处理类的核心方法（标签体的执行逻辑），该方法有以下2个返回值
  - int SKIP_BODY = 0 ;   标签体不会被执行
  - int EVAL_BODY_INCLUDE = 1 ;  标签体会被执行
- doEndTag()是标签执行完毕之后的方法，例如可以让标签在执行完毕后再执行一次，有以下2个返回值
  - int SKIP_PAGE = 5 ;  后面的JSP页面内容不被执行
  - int EVAL_PAGE = 6 ;  后面的JSP页面内容继续执行
- Tag接口中的所有方法执行顺序：
  - 当JSP容器（tomcat、jetty）在将.jsp翻译成.servlet(.java)的时候，如果遇到jsp中有标签，就会依次执行setPageContext() setParent() doStartTag() doEndTag() realease() ，用于解析标签的执行逻辑

- javax.servlet.jsp.tagext.IterationTag接口是Tag的子接口，如果有循环则使用IterationTag
  - doAfterBody()：IterationTag接口的方法，当标签体执行完毕之后的操作，通过返回值决定：
  - EVAL_BODY_AGAIN = 2 ：重复执行
  - SKIP_BODY = 0 ：不再执行

- BodyTag接口：在标签体被显示之前，进行一些其他的额外操作
  - 包含属性int EVAL_BODY_BUFFERED =2 ， 是doStartTag()的第三个返回值，服务器自动将需要显示的内容放入缓冲区（BodyContent）中
  - BodyContent是abstract，具体使用时需要使用实现类BodyContentImpl【引入jasper.jar】
  - 因此要更改最终显示结果，只需要从缓冲区获取原来的数据【getXxx()】，进行修改，再输出【getEnclosingWriter()】。



### 简单方式（JSP2.0）

**顶级接口javax.servlet.jsp.tagext.SimpleTag接口**

将传统方式的doStartTag() doEndTag() doAfterBody() 等方法简化成一个通用的doTag()方法

- 简单方式没有缓冲区，通过 流 显示内容
- javax.servlet.jsp.tagext.JspFragment类：代表一块元素【该块不包含scriptlet，因此tld文件中的<body-content>不能是JSP】
- JspFragment中有一个invoke(Writer var1)方法，入参是“流”，获取此流，修改显示内容
- 每调用一次invoke()方法，就会执行一次标签体
- 实现类SimpleTagSupport：可以获取JspFragment对象

> scriptlet：<%  %> <%！  %> <%=  %> 



## 集群

tomcat：理论上单节点tomcat能够稳定的处理请求并发量200-300

服务端集群（水平+垂直）：
- 水平集群：将服务器安装在各个不同的计算机上（失败迁移）
- 垂直集群：将多个服务器安装在同一个计算机上（负载均衡）

搭建集群（apache+tomcat动静分离）
- apache：特点是处理静态资源
  - 这里的apache是一个服务工具，不是基金组织
- tomcat：特点是处理动态资源

1. 下载apache服务器工具【https://www.apachehaus.com/cgi-bin/download.plx?dli=QYGxmNRVVQ08ERjtWZVp1cKV1UGR1Uw11WUJVe】

- 配置【conf/httpd.conf】
```
Define SRVROOT "D:\dev\cluster\Apache24"
```

- 可以将apache配置成windows服务：
  - 管理员身份打开cmd，通过命令注册apache服务

```
"D:\dev\cluster\Apache24\bin\httpd.exe" -k install -n apache24
```

> 如果注册apache服务时，提示“丢失VCRUNTIME140.DLL”，则需要下载并安装vc_redist.x64.exe，下载地址https://www.microsoft.com/en-US/download/details.aspx?id=48145

2. 准备tomcat：复制两份，规划修改端口，配置引擎，打开集群开关【server.xml】


规划修改端口 | server端口号 | http协议端口 | ajp协议端口
---|---
tomcat-a | 1005 | 1080 | 1009
tomcat-b | 2005 | 2080 | 2009

```
//配置引擎Engine，增加jvmRoute
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat-a/b"
```

```
//打开集群开关，打开以下注释
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
```

3. 结合apache+tomcat：mod_jk.so【下载http://archive.apache.org/dist/tomcat/tomcat-connectors/jk/binaries/windows/ 中的httpd.zip】

- 配置mod_jk.so
  - 存放位置：\Apache24\modules\mod_jk.so
  - 配置\Apache24\conf\workers.properties文件
  - 配置\Apache24\conf\mod_jk.conf（用于加载mod_jk.so和workers.properties）
  - 配置httpd.conf,使得在apache启动时自动加载mod_jk.conf（因为apache程序会自动加载httpd.conf）

workers.properties
```
worker.list=controller,tomcata,tomcatb

#tomcata
worker.tomcata.port=1009
worker.tomcata.host=localhost
worker.tomacta.type=ajp13
#负载均衡的权重
worker.tomcata.lbfactor=1

#tomcatb
worker.tomcatb.port=2009
worker.tomcatb.host=localhost
worker.tomcatb.type=ajp13
#负载均衡的权重
worker.tomcatb.lbfactor=2

#controller
worker.controller.type=lb
worker.controller.balanced_workers=tomcata,tomcatb
worker.controller.sticky_session=false
```

- 分布式session策略
  - sticky：固定将每一个用户的请求分给特定的服务器，后期的请求不会分给其他服务器（弊端：无法失败迁移）
  - session广播：自动同步session（弊端：如果服务器太多，可能造成广播风暴）
  - 集中管理方式【推荐】：将各个服务器的session集群存储到一个数据库中


mod_jk.conf
```
#加载mod_jk.so
LoadModule jk_module modules/mod_jk.so

#加载workers.properties
JKWorkersFile conf/workers.properties

#拦截一切请求
JKMount /* controller
```

httpd.conf
```
#include conf/mod_jk.conf
```

- 使用时在web.xml中加：
```
<distributable/>
```

> CATALINA_HOME会使得启动tomcat时，自动开启CATALINA_HOME指定的tomcat，而集群中需要开启多个不同的tomcat，因此在单机环境下，需要删除CATALINA_HOME
