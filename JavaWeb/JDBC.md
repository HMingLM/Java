[toc]
# JDBC

JDBC：Java Database Connectivity;可以为多种关系型数据库DBMS提供统一的访问方式，用Java操作数据库

## 1.结构
Java程序--JDBC-->JDBC DriveManager（管理不同的数据库驱动）-->各种数据库驱动程序（jar包）（相应数据库厂商提供，连接）-->数据库

## 2.JDBC API 主要功能

-- 与数据库建立连接

-- 发送SQL语句

-- 返回处理结果


- 具体通过以下类/接口实现
  - DriverManager:管理jdbc驱动
  - Connection:连接（通过DriverManager产生）
  - Statement(PreparedStatement):增删改查（通过Connection产生）
  - CallableStatement:调用数据库中的存储过程/存储函数（通过Connection产生）
  - Result:返回的结果集（通过上面的Statement等产生）

- Connection产生操作数据库的对象
  - Connection产生Statement对象：createStatement();
  - Connection产生PreparedStatement对象：prepareStatement();
  - Connection产生CallableStatement对象：prepareCall();

- Statement操作数据库
  - 增删改：executeUpdate()
  - 查询：executeQuery();

- preparedStatement操作数据库
  - 增删改：executeUpdate()
  - 查询：executeQuery();
  - 赋值操作：setXxx();

- ResultSet:保存结果集select * from xxx
  - next():光标下移，判断是否有下一条数据：true/false
  - previous():光标上移，判断是否有下一条数据：true/false
  - getXxx(字段名/位置)：获取具体的字段值

## 3.jdbc访问数据库的具体步骤
- 导入驱动，加载具体的驱动类
- 与数据库建立连接
- 发送sql，执行
- 处理结果集（查询）

## 4.数据库驱动


--- | 驱动jar | 具体驱动类 | 连接字符串
---|---|---|---
Oracl | ojdbc-x.jar | oracle.jdbc.OracleDriver | jdbc:oracle:thin:@localhost:1521:ORCL
MySQL | mysql-connector-java-x.jar | com.mysql.jdbc.Driver | jdbc:mysql://localhost:3306/数据库实例名
SqlServer | sqljdbc-x.jar | com.microsoft.sqlserver.jdbc.SQLServerDriver | jdbc:microsoft:sqlserver:localhost:1433;databasename=数据库实例名


- 使用jdbc操作数据库时，若对数据库进行了更换，只需要替换：驱动、具体驱动类、连接字符串、用户名、密码；（表结构）

## 5.Statement访问数据库示例

JDBCDemo.java
```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCMySQL {
	private final static String URL = "jdbc:mysql://localhost:3306/xs?serverTimezone=UTC&characterEncoding=utf-8";
    private final static String USERNAME = "root";
    private final static String PWD = "281428";
    
    public static void update(){
        Connection connection = null;
        Statement stmt = null;
        try{
            //i.导入驱动，加载具体的驱动类
            Class.forName("com.mysql.cj.jdbc.Driver"); //加载具体的驱动类
            //ii.与数据库建立连接
            connection = DriverManager.getConnection(URL,USERNAME,PWD);
            //iii.发送sql，执行（增删改）
            stmt = connection.createStatement();
            String sql = "insert into xscj values('02366','zs',50)";
            //String sql = "update xscj set 姓名='ls' where 学号=02366";
            //String sql = "delete from xscj where 学号=02366";
            //执行SQL
            int count = stmt.executeUpdate(sql);//返回值表示 增删改 几条数据
            //iiii.处理结果
            if(count>0){
                System.out.println("操作成功！");
            }
        }
        catch(ClassNotFoundException e){
            e.printStackTrace();
        }
        catch(SQLException e){
            e.printStackTrace();
        }
        catch(Exception e){
            e.printStackTrace();
        }
        finally{
            try{
                if(stmt!=null)
                    stmt.close();
                if(connection!=null)
                    connection.close();
            }
            catch(SQLException e){
                e.printStackTrace();
            }
        }
    }
    
    public static void query() {
    	Connection connection = null;
        Statement stmt = null;
        ResultSet rs = null;
        try{
            //i.导入驱动，加载具体的驱动类
            Class.forName("com.mysql.cj.jdbc.Driver"); //加载具体的驱动类
            //ii.与数据库建立连接
            connection = DriverManager.getConnection(URL,USERNAME,PWD);
            //iii.发送sql，执行（查）
            stmt = connection.createStatement();
            String sql = "select 学号,姓名 from xscj";
            //String name = "x";
            //String sql = "select * from xscj where 姓名 like '%"+name+"%'";
            //执行SQL【增删改executeUpdate(),查询executeQuery()】
            rs = stmt.executeQuery(sql);//返回结果集
            //iiii.处理结果
            while(rs.next()) {
            	String sno = rs.getString("学号");
            	String sname = rs.getString("姓名");
            	//String sno = rs.getString("学号"); //下标：从1开始计数
            	//String sname = rs.getString("姓名");
            	System.out.println(sno+"--"+sname);
            }
        }
        catch(ClassNotFoundException e){
            e.printStackTrace();
        }
        catch(SQLException e){
            e.printStackTrace();
        }
        catch(Exception e){
            e.printStackTrace();
        }
        finally{
            try{
            	if(rs!=null)
            		rs.close();
                if(stmt!=null)
                    stmt.close();
                if(connection!=null)
                    connection.close();
            }
            catch(SQLException e){
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args){
        //update();
    	query();
    }
}

```

- error及解决
  - error：关于MySql升级JDBC架包导致时区问题报错（The server time zone value '?й???????' is unrecognized or represents more than one time zone）
  - 原因：在使用mysql的jdbc驱动最新版（6.0+）版本时，数据库和系统时区差异引起的问题。
  - 解决：在jdbc连接的url后面加上serverTimezone=UTC或GMT即可，如果需要指定使用gmt+8时区，需要写成GMT%2B8，不然可能会报错误，解析为空

## 6.PreparedStatement

```
PreparedStatement pstmt = null;

//String sql = "insert into student values('02323','zs1',44)";
//pstmt = connection.prepareStatement(sql);

String sql = "insert into student values(?,?,?)";
pstmt = connection.prepareStatement(sql);
pstmt.setString(1,'02320');
pstmt.setString(2,'zs2');
pstmt.setInt(3,55);

int count = pstmt.executeUpdate();
```

```
PreparedStatement pstmt = null;

String name = "x";
//String sql = "select * from xscj where 姓名 like '%"+name+"%'";
String sql = "select * from xscj where 姓名 like ?";
pstmt = connection.prepareStatement(sql);
pstmt.setString(1,"%x%");

rs = pstmt.executeQuery();
```

## 7.PreparedStatement与Statement在使用时的区别

### 7.1 Statement

- sql
- executeUpdate(sql);

### 7.2 PreparedStatement

- sql(可能存在占位符?)
- 在创建PreparedStatement对象时，将sql预编译：prepareStatement(sql)
- setXxx()替换占位符?
- executeUpdate()

### 7.3 推荐使用PreparedStatement的原因：

- 编码更加简便（避免了字符串的拼接）
- 提高性能（因为有预编译，预编译只需执行一次[sql]）
- 安全（可以有效防止sql注入，stmt存在被sql注入的风险）

sql注入：将客户输入的内容和开发人员的sql语句混为一体

例如输入：用户名：任意值 ' or 1=1 --
          密码：任意值

## 8.JDBC总结（模板）


```
try{
        i.导入驱动，加载具体的驱动类Class.forName("具体驱动类"); 
        ii.与数据库建立连接connection = DriverManager.getConnection(...);
        iii.通过connection，获取操作数据库的对象（Statement/PreparedStatement/callablestatement
            stmt = connection.createStatement();
        iiii.（查询）处理结果集rs = stmt.executeQuery(..);
            while(rs.next()) {
            	rs.getString(...);
            }
    }
catch(ClassNotFoundException e){ ... }
catch(SQLException e){ ... }
catch(Exception e){ ... }
finally{
            //关闭顺序与打开顺序相反
            if(rs!=null)	rs.close();
            if(stmt!=null)    stmt.close();
            if(connection!=null)    connection.close();
        }
```

jdbc中，除了Class.forName()抛出ClassNotFoundException，其余方法全部抛出SQLException

## 9.CallableStatement

CallableStatement:JDBC调用存储过程和存储函数

- connection.prepareCall(参数【存储过程/存储函数名】);

- 参数格式
  - 存储过程（无返回值return，用out参数替代）
> { call 存储过程名（参数列表）}
  - 存储函数（有返回值return）
> { ? = call 存储函数名（参数列表）}

### 9.1 调用存储过程

###### 9.1.1 示例：求两数和

```
create or replace procedure addTwoNum ( num1 in number,num2 in number,result out number )
as
begin
    result :=num1+num2;
end;
/
```

```
callablestatement cstmt = null;

cstmt = connection.prepareCall("{ call addTwoNum(?,?,?) }");
cstmt.setInt(1,10);
cstmt.setInt(2,20);
cstmt.registerOutParameter(3,Types.INTEGER); //设置输出参数的类型
cstmt.execute();
int result = cstmt.getInt(3); //获取计算结果
System.out.println(result);
```

- 强调：若通过sqlplus访问数据库，只需开启OracleServiceSID，通过其他程序访问数据（sqldevelop、navicate、JDBC），需开启OracleServiceSID、XxxListener；

###### 9.1.2 JDBC调用存储过程的步骤

1. 产生调用存储过程的对象（CallableStatement） cstmt = connection.prepareCall( "..." );
2. 通过setXxx()处理输入参数值 cstmt.setInt(...);
3. 通过cstmt.registerOutParameter(...) 处理输出参数的类型
4. cstmt.execute() 执行
5. 接受输出值（返回值） getXxx()

### 9.2 调用存储函数

注意与调用存储过程的区别，在调用时注意参数；

```
create or replace function addTwoNum ( num1 in number,num2 in number )
return number
as
    result number;
begin
    result :=num1+num2;
    return result;
end;
/
```

```
callablestatement cstmt = null;

cstmt = connection.prepareCall("{ ? = call addTwoNumfunction(?,?) }");
cstmt.setInt(2,10);
cstmt.setInt(3,20);
cstmt.registerOutParameter(1,Types.INTEGER); //设置输出参数的类型
cstmt.execute();
int result = cstmt.getInt(1); //获取计算结果
System.out.println(result);
```

## 10.处理CLOB/BLOB类型数据

### 10.1 处理稍大型数据

- i.存储路径

> 通过JDBC存储文件路径，然后根据IO操作处理
> 
> 例：JDBC将 E:\JDK_API_zhCN.CHM 文件以字符串形式存储到数据库中
> 
> 获取：先获取该路径"E:\JDK_API_zhCN.CHM"，再IO

- ii.CLOB/BLOB


CLOB：大文本数据（小说->数据）【CLOB--Oracle/Text--MySQL】

BLOB：二进制

设置CLOB类型：setCharacterStream

### 10.2 clob实例


1. 先通过pstmt的?代替小说内容（占位符）
2. 再通过pstmt.setCharacterStream(2,reader,(int)file.length()); 将上一步的?替换为小说流，注意第三个参数是Int类型

> 通过jdbc存储大文本数据（小说）

```
create table mynovel(id number primary key,novel clob);
```

```
PreparedStatement pstmt = null;

String sql = "insert into mynovel values(?,?)";
pstmt = connection.prepareStatement(sql);
pstmt.setInt(1,1);

File file = new File("E:\\all.txt");
InputStream in = new FileInputStream(file);
Reader reader = new InputStreamReader(in,"UTF-8"); //使用转换流：转换流可设置编码；
pstmt.setCharacterStream(2,reader,(int)file.length());

int count = pstmt.executeUpdate();
if(count>0){
    out.println("操作成功！");
}
in.close();
```

3. 通过 Reader reader = rs.getCharacterStream("NOVEL"); 将clob类型的数据保存到Reader对象中
4. 将Reader通过Writer输出即可


> 读取小说

```
String sql = "select NOVEL from mynovel where id = ? ";
pstmt = connection.prepareStatement(sql);
pstmt.setInt(1,1);

rs = pstmt.executeQuery();
if(rs.next())
{
    Reader reader = rs.getCharacterStream("NOVEL");
    Writer writer = new FileWriter("src/小说.txt");
    
    char[] chs = new char[100];
    int len = -1;
    while( (len = reader.read(chs)) !=-1 ) {
        writer.write( chs,0,len );
    }
    writer.close();
    reader.close();
}
```

### 10.3 blob实例

与clob步骤基本一致，区别：setBinaryStream(),getBinaryStream()


> 通过jdbc存储二进制类型（mp3）

```
create table mymusic(id number primary key,music blob);
```

```
PreparedStatement pstmt = null;

String sql = "insert into mymusic values(?,?)";
pstmt = connection.prepareStatement(sql);
pstmt.setInt(1,1);

File file = new File("d:\\luna.mp3");
InputStream in = new FileInputStream(file);
pstmt.setBinaryStream(2,in,(int)file.length());

int count = pstmt.executeUpdate();
if(count>0){
    out.println("操作成功！");
}
in.close();
```

> 读取二进制文件

```
String sql = "select music from mymusic where id = ? ";
pstmt = connection.prepareStatement(sql);
pstmt.setInt(1,1);

rs = pstmt.executeQuery();
if(rs.next())
{
    InputStream in = rs.getBinaryStream("music");
    OutputStream out = new FileOutputStream("src/music.mp3");
    
    byte[] chs = new byte[100];
    int len = -1;
    while( (len = in.read(chs)) !=-1 ) {
        out.write( chs,0,len );
    }
    out.close();
    in.close();
}
```

## 11.JSP访问数据库

- 导包操作
  - java项目：将jar复制到工程中，再右键build path -> add to builld Path
  - Web项目：将jar复制到WEB-INF/lib

- 注意：

> error：The import Xxx cannot be resolved...

尝试解决步骤

1. （可能是jdk、tomcat版本问题）右键项目->build path，将其中报错的library或lib删除后重新导入
2. 清空各种缓存：右键项目->Clean tomcat...\clean(Project - clean 或者进入tomcat目录删除work子目录)
3. 删除之前的tomcat，重新解压缩、配置tomcat，重启计算机
4. 若类没有包，则将该类加入包中


login.jsp
```
<body>
	<form action="check.jsp" method="post" ><br>
		用户名<input type="text" name="uname"><br>
		密码<input type="password" name="upwd"><br>
		<input type="submit" value="登录"><br>
	</form>
</body>
```

check.jsp
```
<body>
	<%
		request.setCharacterEncoding("UTF-8");
		String name = request.getParameter("uname");
		String pwd = request.getParameter("upwd");
		Login login = new Login(name,pwd);
		loginDao dao = new loginDao();
		int result = dao.login(login);
		if(result>0){
			out.print("登录成功！");
		}
		else if(result==0){
			out.print("用户名或密码错误，登录失败！");
		}
		else{
			out.print("系统错误，登录失败！");
		}
	%>
</body>
```

Login.java
```
package org;

public class Login {
	//private int id;
	private String name;
	private String pwd;
	public Login(){
		
	}
	public Login(String name,String pwd){
		//this.id = id;
		this.name = name;
		this.pwd = pwd;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getPwd() {
		return pwd;
	}
	public void setPwd(String pwd) {
		this.pwd = pwd;
	}
	
}
```

loginDao.java
```
package org;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class loginDao {
	public int login(Login login) {
		String URL = "jdbc:mysql://localhost:3306/xs?serverTimezone=UTC&characterEncoding=utf-8";
		String USERNAME = "root";
		String PWD = "281428";
		Connection connection = null;
		Statement stmt = null;
		ResultSet rs = null;
		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
			connection = DriverManager.getConnection(URL,USERNAME,PWD);
			stmt = connection.createStatement();
			
			String sql = "select count(*) from user where uname='"+login.getName()+"' "
					+ "and upwd='"+login.getPwd()+"' ";
			rs = stmt.executeQuery(sql);
			int count = -1;
			if(rs.next()) {
				count = rs.getInt(1);
			}
			return count;
		}
		catch(ClassNotFoundException e) {
			e.printStackTrace();
			return -1;
		}
		catch(SQLException e) {
			e.printStackTrace();
			return -1;
		}
		catch(Exception e) {
			e.printStackTrace();
			return -1;
		}
		finally {
			try {
				if(rs!=null)  rs.close();
				if(stmt!=null)  stmt.close();
				if(connection!=null)  connection.close();
			}
			catch(SQLException e) {
				e.printStackTrace();
				return -1;
			}
		}
	}
}

```



## 12.JavaBean

- JavaBean：满足以下两点的Java类
1. public修饰的类，public无参构造
2. 所有属性（如果有）都是private，并且提供set/get（若是boolean，则get可替换为is）

- 使用层面，JavaBean分为两大类：
  - 封装业务逻辑的JavaBean（如loginDao.java），用于操作一个封装数据的JavaBean
  - 封装数据的JavaBean（实体类，如login.java），对应于数据库的一张表

- 作用：
  - 减轻jsp的复杂度
  - 提高代码复用

