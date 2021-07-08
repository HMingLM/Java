[toc]

# MyBatis

## 0.补充

**MyBatis执行流程：（Debug查看）**
- Resource获取加载全局配置文件
- 实例化SqlSessionFactoryBuilder构造器
- 解析配置文件流XMLConfigBuilder
- Configuration 所有的配置信息
- 实例化SqlSessionFactory
- transactional事务管理
- 创建executor执行器
- 创建SqlSession
- 实现CRUD

![avatar](D:/其他/图片/study/SqlSessionFactory.png)

mybatis-config.xml

- MyBatis默认使用的事务管理方式为JDBC（MANAGED淘汰）
  - <transactionManager type="JDBC"/>
- 默认数据源类型为POOLED，（还有UNPOOLED，淘汰的JNDI）
  - <dataSource type="POOLED">


**<properties> 属性配置**
  - 可以直接引入外部配置文件
  - 可以在其中配置属性<property>
  - 以上两者，外部文件的优先级更高

**<typeAliases> 类型别名**
- 在XML配置中为Java类型名设置短的名字，以减少类完全限定名【全类名】的冗余
  - ① <typeAlias> 为某个类型设置别名
  - ② <package> 扫描包下的实体类，默认别名是类名的首字母小写
  - ③ @Alias("xxx") JavaBean上注解定义别名

- 注：
  - int是Integer的默认别名
  - _int-------->int
  - 【其他类似。。】
  - map是Map的默认别名
  - hashmap-------->HashMap
  - list--------->List
  - 【其他类似。。】


**日志**
  - STDOUT_LOGGING 标准日志输出
  - LOG4J
    - 0、设置<setting>
    - i、导入依赖
    - ii、配置文件
    - iii、用日志对象打印信息，参数为当前类的class
      - static Logger logger = Logger.getLogger(xxx.class);


```
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING / LOG4J" />
</settings>
```

![avatar](D:/其他/图片/study/LOG4J配置.png)
  
  
**了解：**
  - 接口中使用注解如@Select，而不用mapper.xml【适用于简单场景】
  - RowBounds分页，面向对象的方式代替limit语句

**三个面向的区别：**
- 面向对象：考虑问题时，以对象为单位，考虑它的属性及方法
- 面向过程：考虑问题时，以具体的流程（事务过程）为单位，考虑它的实现
- 面向接口：接口设计与非接口设计是针对复用技术而言的，与面向对象/过程不是一个问题，更多的实现就是对系统整体的架构


- @Param()
  - 接口中方法参数是多个基本类型/String时，必须用@Param，只有一个建议也加，引用类型不需要加，
  - SQL语句中引用的就是@Param()中设定的属性名

- #{} 防止SQL注入，而${} 不行，它原样输出


**多对一【association关联】：**

- 处理复杂的属性：
  - association--关联
  - collection--集合

按照查询嵌套处理：
```
<select id="getStudent" resultMap="StudentTeacher">
    select * from student
</select>

<resultMap id="StudentTeacher" type="Student">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <!-- 处理复杂的属性：association--对象  collection--集合-->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/> 
</resultMap>

<select id="getTeacher" resultType="Teacher">
    select * from teacher where id = #{id}
</select>
```

按照结果嵌套处理：
```
<select id="getStudent" resultMap="StudentTeacher">
    select s.id sid,s.name sname,t.name tname
    from student s,teacher t
    where s.tid = t.id
</select>

<resultMap id="StudentTeacher" type="Student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="Teacher">
        <result property="name" column="tname"/>
    </association>
</resultMap>
```

**动态SQL**
- <where> <if>
- <choose> <when> <otherwise> 只匹配一个
- <where> <set>
- <trim> 自定义前后缀替换等

**缓存**
- 一级缓存/本地缓存：默认开启【SqlSession级别（拿到连接-关闭连接）】
- 二级缓存：手动开启和配置【namespace级别】
  - config中设置cacheEnabled="true"【默认为true，即全局缓存默认开启，但需在指定mapper中开启】
  - mapper.xml中加<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true" />
- 可用Cache接口自定义二级缓存

- 增删改语句会刷新缓存，此外也有算法管理缓存
- 用户查询时先看二级缓存，再看一级缓存，最后看数据库
- 一种调优：给mapper中指定增删改的SQL设置 flushCache="false"


## 1.概念

- MyBatis可以简化JDBC操作，实现数据的持久化；是ORM的一个实现；
- ORM : Object Relational Mapping ,可以使得开发人员像操作对象一样操作数据库表；

- myBatis规定：输入参数parameterType和输出参数resultType在形式上都只能有一个
  - 输入参数：若是简单类型（8个基本类型+String），则可以使用任何占位符,#{xxx};若是对象类型，则必须是对象的属性 #{属性名}
  - 输出参数：如果返回值类型是一个对象（如person），则无论返回一个还是多个，都写成resultType="org.Person" 

- 注意：如果使用的事务方式为jdbc，则需手工commit提交，即session.commit();



## 2.步骤

1. 先导入驱动mybatis.jar和mysql.jar
2. conf.xml:配置数据库信息和需要加载的映射文件
3. 表-类,映射文件xxMapper.xml：增删改查标签<select>
4. 测试

## 3. 基础方式的增删改查CRUD

conf.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 通过environments的default值和environment的id来指定MyBatis运行时的数据库环境 -->
    
 	<environments default="development">
 	<environment id="development">
 	    
 	    <!-- 事务提交方式：
 	        JDBC:利用JDBC方式处理事务（commit rollback close）
 	        MANAGED：将事务交由其他组件去托管（spring，jobss），默认会关闭连接。
 	    -->
 		
 		<transactionManager type="JDBC"/>
 		
 		<!-- 数据源类型：
 		    POOLED：使用数据库连接池
 		    UNPOOLED：传统的JDBC模式（每次访问数据库，均需要打开、关闭等较消耗性能的数据库操作
 		    JNDI：从tomcat中获取一个内置的数据库连接池【数据源】
 		-->
 		
 		<dataSource type="POOLED">
 			<!-- 配置数据库信息 -->
 			<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
 			<property name="url" value="jdbc:mysql://localhost:3306/xs?serverTimezone=UTC"/>
 			<property name="username" value="root"/>
 			<property name="password" value="281428"/>
 		</dataSource>
 	</environment>
 	</environments>
 	<mappers>
 		<!-- 加载映射文件 -->
 		<mapper resource="org/personMapper.xml"/>
 	</mappers>
</configuration>
```

personMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
 <!-- namespace：该mapper.xml 映射文件发唯一标识 -->

<mapper namespace="org.personMapper">  <!-- 映射文件的路径 -->
    
    <!-- 后续通过namespace.id定位sql -->
    <!-- 
        parameterType:输入参数的类型
         resultType：查询返回结果集的类型
    --> 
 	
 	<select id="quaryPersonById" resultType="org.Person" parameterType="int">
 		select * from person where id = #{id}
 	</select>
 	<insert id="addPerson" parameterType="org.Person">
 		insert into person(id,name,age) value(#{id},#{name},#{age})
 	</insert>
 	<delete id="deletePersonById" parameterType="int">
 		delete from person where id = #{id}
 	</delete>
 	<update id="updatePersonById" parameterType="org.Person">
 		update person set name=#{name},age=#{age} where id=#{id}
 	</update>
 	<select id="quaryAllPersons" resultType="org.Person" >
 		select * from person
 	</select>
</mapper>
```

Person.java
```
package org;

public class Person {
	private int id;
	private String name;
	private int age;
	public Person( ) {
		
	}
	public Person(int id, String name, int age) {
		this.id = id;
		this.name = name;
		this.age = age;
	}
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public String toString() {
		return this.id+"-"+this.name+"-"+this.age;
	}
	
}

```

TestMyBatis.java
```
package org;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class TestMyBatis {
	//查询单个person
	public static void queryPersonById() throws IOException {
		//加载MyBatis配置文件（为了访问数据库）
		Reader reader = Resources.getResourceAsReader("conf.xml");
		
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//session - connection
		SqlSession session = sessionFactory.openSession();
		String statement = "org.personMapper."+"quaryPersonById";
		Person person = session.selectOne( statement,1 );
		System.out.println(person);
		session.close();
	}
	
	//查询全部person
	public static void queryAllPersons() throws IOException {
		//加载MyBatis配置文件（为了访问数据库）
		Reader reader = Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//session - connection
		SqlSession session = sessionFactory.openSession();
		String statement = "org.personMapper."+"quaryAllPersons";
		List<Person> persons = session.selectList(statement);
		System.out.println(persons);
		session.close();
	}
	
	//增加person
	public static void addPerson() throws IOException {
		//加载MyBatis配置文件（为了访问数据库）
		Reader reader = Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//session - connection
		SqlSession session = sessionFactory.openSession();
		String statement = "org.personMapper."+"addPerson";
		Person person = new Person(3,"ww",25);
		int count = session.insert(statement, person);
		session.commit(); //提交事务
		System.out.println("增加了"+count+"个person");
		session.close();
	}
	
	//删除person
	public static void deletePersonById() throws IOException {
		//加载MyBatis配置文件（为了访问数据库）
		Reader reader = Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//session - connection
		SqlSession session = sessionFactory.openSession();
		String statement = "org.personMapper."+"deletePersonById";
		int count = session.delete(statement, 3 );
		session.commit(); //提交事务
		System.out.println("删除了"+count+"个person");
		session.close();
	}		
	
	//修改person
	public static void updatePersonById() throws IOException {
		//加载MyBatis配置文件（为了访问数据库）
		Reader reader = Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(reader);
		//session - connection
		SqlSession session = sessionFactory.openSession();
		String statement = "org.personMapper."+"updatePersonById";
		Person person = new Person();
		person.setId(2);
		person.setName("lxs");
		person.setAge(44);
		int count = session.update(statement, person);
		session.commit(); //提交事务
		System.out.println("修改了"+count+"个person");
		session.close();
	}	
		
	public static void main(String[] args) throws IOException {
		queryAllPersons();
		addPerson();
		queryAllPersons();
		deletePersonById();
		queryAllPersons();
		updatePersonById();
		queryAllPersons();
	}
}

```

## 4. mapper动态代理方式的增删改查【MyBatis接口开发】

### 4.1 原则：约定优于配置

- 硬编码方式
```
//abc.java
    Configuration conf = new Configuration();
    con.setName("myProject");
```

- 配置方式
```
<!-- abc.xml -->
    <name> myProject </name>
```

- 约定
```
默认值即为myProject
```

### 4.2 具体实现步骤

不同之处：约定的目标是省略掉statement，即根据约定直接定位SQL语句；

- 基础环境：mybatis.jar 、 mysql.jar 、 conf.xml 、 mapper.xml
- 接口，接口中的方法遵循以下约定：
  - 方法名和mapper.xml文件中标签的id值一致
  - 方法的输入参数和mapper.xml文件中标签的parameterType类型一致
  - 方法的返回值和mapper.xml文件中标签的resultType类型一致

- 要实现接口中的方法和Mapper.xml中SQL标签一一对应，还需注意：**namespace的值就是接口的全类名**；

personMapper.java
```
package orh;

import org.Person;

public interface PersonMapper {
	Person quaryPersonById(int id);
	Person quaryAllPersons();
	void addPerson(Person person);
	void deletePersonById(int id);
	void updatePersonById(Person person);
}

```

### 4.3 匹配的过程（约定的过程）

1. 根据接口名找到mapper.xml文件【namespace=接口全类名】
2. 根据接口的方法名找到mapper.xml文件中的SQL标签

- 由此可以保证：当我们调用接口中的方法时，程序能自动定位到某一个Mapper.xml文件中的sql标签
- 习惯：把SQL映射文件（mapper.xml）和接口放在同一个包中；（注意修改conf.xml中加载mapper.xml文件的路径）

```
StudentMapper studentMapper = session.getMapper(StudentMapper.class);
studentMapper.方法名();
```
- 通过session对象获取接口，再调用该接口中的方法，程序会自动执行该方法对应的SQL；

## 5. 优化

### 5.1 属性文件

db.properties：key=value
```
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/xs?serverTimezone=UTC
username=root
password=281428
```

conf.xml:引入配置信息后，使用${key}
```
<configuration>
    <properties  resource="db.properties" />

    <property name="driver" value="${driverr}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
```

### 5.2 全局参数

在conf.xml中<configuration>下设置【一般不设置】
```
<settings>
    <setting name="cacheEnableed" value="false" />
    <setting name="lazyLoadEnables" value="false" />
</settings>
```

### 5.3 别名

在conf.xml中<configuration>下设置单个/批量别名

- 别名忽略大小写！
- 除了自定义别名外，MyBatis还内置了一些常见类的别名

```
<typeAliases>
    <!--  单个别名 -->
    <typeAlias  type="org.Person"  alias="person" />
    
    <!--  批量别名【自动将该包中的所有类批量定义别名：别名即类名（不带包名！）】 -->
    <package name="org" />
</typeAliases>
```

## 6.类型处理器（类型转换器）

- MyBatis自带一些常见的类型处理器（int-number)
- 自定义Mybatis类型处理器【java-数据库（jdbc类型）】

### 6.1 自定义类型转换器

##### 6.1.1 步骤

- i.创建转换器(2选1)
  - 实现TypeHandler接口
  - 继承TypeHandler接口的实现类BaseTypeHandler<java类型>（推荐）

- ii.配置conf.xml

#####  6.1.2 创建转换器

实体类Student  stuSex  Boolean（true/false）  --  表student  stuSex  number（1/0）

- java(boolean)-->DB(number):
  setNonNullParameter(PreparedStatement ps, int i, Boolean parameter, JdbcType arg3)
  - ps:PreparedStatement对象；
  - i:PreparedStatement对象操作参数的位置；
  - parameter：java值；
  - jdbcType：jdbc操作的数据库类型；

- DB(number)-->java(boolean)；
  - getNullableResult(ResultSet rs, String columnName)
  - getNullableResult(ResultSet rs, int columnIndex)
  - getNullableResult(CallableStatement cs, int columnIndex)


BooleanAndIntConverter.java
```
public class BooleanAndIntConverter extends BaseTypeHandler<Boolean> {

    public void setNonNullParameter(PreparedStatement ps, int i, Boolean parameter, JdbcType jdbctype) throws SQLException {
		if(parameter) {
			ps.setInt(i, 1);
		}
		else {
			ps.setInt(i, 0);
		}
	}
	
    public Boolean getNullableResult(ResultSet rs, String columnName) throws SQLException {
		int sexNum = rs.getInt(columnName);
		return sexNum ==1?true:false;
	}
	
    public Boolean getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
		int sexNum = rs.getInt(columnIndex);
		return sexNum ==1?true:false;
	}


    public Boolean getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
		int sexNum = cs.getInt(columnName);
		return sexNum ==1?true:false;
	}
	
}
```

- 注意：INTEGER不忽略大小写

##### 6.1.3 配置conf.xml

- 如果类中属性和表中的字段类型能够合理识别（String-varchar2），则可使用javaType，否则（boolean-number）使用==resultMap==
- 如果类中属性名和表中的字段名能够合理识别（stuNo-stuno），则可使用javaType，否则（id-stuno）使用==resultMap==

conf.xml
```
<typeHandlers>
    <typeHandler handler="org.converter.BooleanAndIntConverter" javaType="Boolean"  jdbcType="INTEGER"/>
</typeHandlers>
```

studentMapper.xml【查询：使用了类型转换器】
```
<select id="quaryStudentByStunoWithConverter" parameterType="int" resultMap="studentResult">
 		select * from student where stuno = #{stuno}
</select>
 
<resultMap type="student" id="studentResult">
    <!-- 分为主键id 和非主键result，指定类中的属性和表中字段的对应关系 -->
    <id property="stuNo" column="stuno" />
    <result property="stuName" column="stuname" />
    <result property="stuAge" column="stuage" />
    <result property="graName" column="graname" />
    <result property="stuSex" column="stusex" javaType="boolean" jdbcType="INTEGER" />
</resultMap>

<insert id="addStudentWithConverter" parameterType="student">
 	insert into student(stuno,stuname,stuage,graname,stusex) value(#{stuno},#{stuname},#{stuage},#{graName},#{ stuSex, javaType=boolean, jdbcType=INTEGER} )
</insert>
```

- 注意
insert into student(stuno,stuname,stuage,graname,stusex) value(#{stuno},#{stuname},#{stuage},#{graName},#{ stuSex, javaType=boolean, jdbcType=INTEGER} )中存放的是属性值，严格区分大小写；



##### 6.1.4 测试

StudentMapper.java
```
Student quaryStudentByStunoWithConverter(int stuno);  //查询

void addStudentWithConverter(Student student);  //增加
```


Test.java
```
Student student = sessionMapper.quaryStudentByStunoWithConverter(1);  //查询

//增加
student.setStuSex(true); //1
sessionMapper.addStudentWithConverter(student);
```


### 6.2 resultMap

功能：
- i.类型转换
- ii.属性-字段的映射关系

## 7. 输入参数

### 7.1 简单类型

输入参数类型为简单类型（8个基本类型+String）

##### 7.1.1 #{} ${}的区别
1. 任意值／value

- #{任意值}；
- 而${value}，其中的标识符只能是value

2. ''

- #{}自动给String类型加上''【自动类型转换】；
- 而${}原样输出，但是适合于动态排序【动态字段】

```
select stuno,stuname,stuage from student where stuname = #{value}
select stuno,stuname,stuage from student where stuname = '${value}'

动态排序：
select stuno,stuname,stuage from student order by ${vakue} asc
```

3. SQL注入

- #{}可以防止SQL注入
- ${}不可以防止SQL注入


##### 7.1.2 #{} ${}相同之处

- 都可以获取对象的值（嵌套类型对象）

### 7.2 对象类型

- #{属性名}
- ${属性名}

studentMapper.xml
```
<select id="quaryStudentByStuageOrstuName" parameterType="student" resultType="student">
 		select stuno,stuname,stuage from student where where stuage = #{stuAge} or stuname like '%${stuName}%'
</select>
```

StudentMapper.java
```
List<Student> quaryStudentBystuageOrstuName(Student student);
```

Test.java
```
Student student = new Student();
student.setstuAge(24);
student.setstuName("w");
List<Student> students = studentMapper.quaryStudentBystuageOrstuName(student);
```


### 7.3 嵌套对象类型

studentMapper.xml
```
<!-- 输入参数为级联属性 -->
<select id="quaryStudentByaddress" parameterType="student" resultType="student">
 		select stuno,stuname,stuage from student  where homaAddress = #{address.homaAddress} or schoolAddress = '${address.schoolAddress}'
</select>
```

StudentMapper.java
```
List<Student> quaryStudentByaddress(Student address);
```

Test.java
```
Student student = new Student();
Address address = new Address();
address.setHomeAddress("xa");
address.setSchoolAddress("bj");
student.setAddress(address);
List<Student> students = studentMapper.quaryStudentByaddress(student);
```

- 对比

studentMapper.xml
```
<select id="quaryStudentByaddress" parameterType="address" resultType="student">
 		select stuno,stuname,stuage from student where where homaAddress = #{homaAddress} or schoolAddress = '${schoolAddress}'
</select>
```

StudentMapper.java
```
List<Student> quaryStudentByaddress(Address address);
```

Test.java
```
Address address = new Address();
address.setHomeAddress("xa");
address.setSchoolAddress("bj");
List<Student> students = studentMapper.quaryStudentByaddress(address);
```

### 7.4 HashMap

输入对象为HashMap:用map中key的值匹配占位符#{stuAge}，若匹配成功，就用map中的value替换占位符；

studentMapper.xml
```
<select id="quaryStudentByStuageOrstuNameWithHashMap" parameterType="HashMap" resultType="student">
 		select stuno,stuname,stuage from student where where stuage = #{stuAge} or stuname like '%${stuName}%'
</select>
```

StudentMapper.java
```
List<Student> quaryStudentBystuageOrstuNameWithHashMap(Map<String ,Object> map);
```

Test.java
```
Map<String,Object> studentMap = new HashMap<>();
studentMap.put("stuAge",24);
studentMap.put("stuName","zs");
List<Student> students = studentMapper.quaryStudentBystuageOrstuNameWithHashMap(studentMap);
```

## 8. 调用存储过程执行CRUD

- 通过调用存储过程实现查询，查询某个年级的学生总数
- statementType="CALLABLE",设置SQL的执行方式是存储过程
- 存储过程的输入参数，在MyBatis中用Map来传递（HashMap）
- 使用时，通过HashMap的put方法传入输入参数的值；通过HashMap的get方法获取输出参数的值
- mybatis不支持某些类型，查表【MyBatis中javaType与jdbcType对应关系】


```
create or replace procedure quaryCountByGradeWithProcedure(gName in varchar,scount out number)
as
begin
select count(*) into scount from student where graname = aname ;
end;
/
```


studentMapper.xml
```
<select id="quaryCountByGradeWithProcedure" statementType="CALLABLE" parameterType="HashMap" >
    {
        CALL quaryCountByGradeWithProcedure(
            #{gName,jdbcType=VARCHAR,mode=IN},
            #{scount,jdbcType=INTEGER,mode=OUT}
        )
    }
```

StudentMapper.java
```
void quaryCountByGradeWithProcedure(Mao<String,Object> params);
```

Test.java
```
//通过map给存储过程指定输入参数
Map<String,Object> params = new HashMap<>();
params.put("gName","g1");
studentMapper.quaryCountByGradeWithProcedure(params); //调用存储过程，并传入输入参数
int count = params.get("sCount");  //获取存储过程的输出参数
```

## 9.输出参数


resultType为：
- 简单类型（8个基本类型+String）
- 实体对象类型
- 实体对象类型的集合（resultType依然只写集合的元素类型）
- HashMap

### 9.1 输出参数类型为HashMap

- HashMap本身是一个集合，可以存放多个元素
- 但返回值为HashMap时，查询的结果只能是1个学生（no，name）
- 结论：一个HashMap可以对应一个学生的多个属性，用多个HashMap对应多个学生

studentMapper.xml
```
<!-- 一个学生 -->
<select id="quaryStudentOutByHashMap" resultType="HashMap" >
    select stuno "no",stuname "name" from student where stuno = 1
</select>

<!-- 多个学生 -->
<select id="quaryAllStudentsOutByHashMap" resultType="HashMap" >
    select stuno "no",stuname "name" from student
</select>
```

studentMapper.java
```
<!-- 一个学生 -->
HashMap<String,Object> quaryStudentOutByHashMap();

<!-- 多个学生 -->
List<HashMap<String,Object>> quaryAllStudentsOutByHashMap();
```

Test.java
```
<!-- 一个学生 -->
HashMap<String,Object> studentMap = studentMapper.quaryStudentOutByHashMap();
System.out.println(studentMap);

<!-- 多个学生 -->
List<HashMap<String,Object>> studentMap = studentMapper.quaryAllStudentsOutByHashMap();
System.out.println(studentMap);
```

### 9.2 resultMap

- 实体类的属性、数据表的字段：类型、名字不同时（stuno，id）用resultMap
- 当属性名和字段名不一致时，除了使用resultMap外，还可使用**resultType+HashMap**
  - 通过 【select 表的字段名 "类的属性名" from... 】来指定字段名和属性名的对应关系

studentMapper.xml
```
<select id="quaryStudentByIdWithHashMap" parameterType="int" resultType="Student" >
select id "stuNo",name "stuName" from student where id=#{id}
```

studentMapper.java
```
Student quaryStudentByIdWithHashMap(int stuno);
```

Test.java
```
Student student = studentMapper.quaryStudentByIdWithHashMap();
System.out.println(studentMap);
```

- 注：若查询多个字段，发现某一字段结果始终为默认值（0/0.0/null），则可能是表的0字段名和类的属性名写错；


## 10.动态SQL

### 10.1 <where>、<if>

studentMapper.xml
```
<select id="quaryStuByNOrAWithSQLTag" parameterType="student" resultType="student">
 		select stuno,stuname,stuname,stuage from student
 		<where>
 		    <!-- <if test="student有属性且不为null"> -->
 		    <if test="stuName!=null and stuName!='' ">
 		        and stuname = #{stuName}
 		    </if>
 		    <if test="stuAge!=null and stuAge!=0 ">
 		        and stuage = #{stuAge}
 		    </if>
 		</where>
</select>
```

StudentMapper.java
```
Student quaryStuByNOrAWithSQLTag(Student student);
```

Test.java
```
Student student = new Student();
student.setstuAge(24);
student.setstuName("ls");
Student student = studentMapper.quaryStuByNOrAWithSQLTag(student);
```

<where>会自动处理第一个<if>标签中的and，但不会处理之后<if>中的and

### 10.2 <foreach>

```
ids = {1,2,53};
select * from student where stuno in(1,2,53)
```

<foreach>迭代的类型：数组、对象数组、集合、属性(Grade类:List<Integer> ids)

##### 10.2.1 属性

Grade.java
```
public calss Grade{
    //学号
    private List<Integer> stuNos;
    
    get...
    set...
}
```

studentMapper.xml
```
<select id="queryStudentWithNosInGrade" parameterType="grade" resultType="student">
 		select * from student
 		<where>
 		    <if test="stuNos!=null and stuNos.sizes>0 ">
 		        <foreach collection="stuNos" open=" and stuno in (" close=")" item="stuNo" separator=",">
 		            #{stuNo}
 		        </foreach>
 		    </if>
 		</where>
</select>
```

studentMapper.java
```
List<Student> queryStudentWithNosInGrade(Grade grade);
```

Test.java
```
Grade grade = new Grade();
List<Integer> stuNos = new ArrayList<>();
stuNos.add(1);
stuNos.add(2);
stuNos.add(53);
grade.setStuNos(stuNos);
List<Student> students = studentMapper.queryStudentWithNosInGrade(grade);
```

##### 10.2.2 简单类型数组

- 约定：无论编写代码时，传递的是什么参数名（stuNos），在mapper.xml中必须用array代替该数组；

studentMapper.xml
```
<select id="queryStudentWithArray" parameterType="int[]" resultType="student">
 		select * from student
 		<where>
 		    <if test="array!=null and array.length>0 ">
 		        <foreach collection="array" open=" and stuno in (" close=")" item="stuNo" separator=",">
 		            #{stuNo}
 		        </foreach>
 		    </if>
 		</where>
</select>
```

studentMapper.java
```
List<Student> queryStudentWithArray(int[] stuNos);
```

Test.java
```
stuNos = {1,2,53};
List<Student> students = studentMapper.queryStudentWithArray(stuNos);
```

##### 10.2.3 集合

- 约定：无论编写代码时，传递的是什么参数名（stuNos），在mapper.xml中必须用list代替该数组；

studentMapper.xml
```
<select id="queryStudentWithList" parameterType="list" resultType="student">
 		select * from student
 		<where>
 		    <if test="list!=null and list.size>0 ">
 		        <foreach collection="list" open=" and stuno in (" close=")" item="stuNo" separator=",">
 		            #{stuNo}
 		        </foreach>
 		    </if>
 		</where>
</select>
```

studentMapper.java
```
List<Student> queryStudentWithList(List<Integer> stuNos);
```

Test.java
```
List<Integer> stuNos = new ArrayList<>();
stuNos.add(1);
stuNos.add(2);
stuNos.add(53);
List<Student> students = studentMapper.queryStudentWithList(stuNos);
```

##### 10.2.4 对象数组

- 约定：无论编写代码时，传递的是什么参数名（Student[]），在mapper.xml中必须用Object[]代替该数组；

Student[] students = {student0,student1,student2},每个studentx包含一个学号属性

studentMapper.xml
```
<select id="queryStudentWithObjectArray" parameterType="Object[]" resultType="student">
 		select * from student
 		<where>
 		    <if test="array!=null and array.length>0 ">
 		        <foreach collection="array" open=" and stuno in (" close=")" item="student" separator=",">
 		            #{student.stuNo}
 		        </foreach>
 		    </if>
 		</where>
</select>
```

studentMapper.java
```
List<Student> queryStudentWithObjectArray(Student[] students);
```

Test.java
```
Student stu1 = new Student();
stu1.setStuNo(1);
Student stu2 = new Student();
stu2.setStuNo(2);
Student stu53 = new Student();
stu53.setStuNo(53);
Student[] stus = new Student[]{stu1,stu2,stu53};
List<Student> students = studentMapper.queryStudentWithObjectArray(stus);
```

### 10.3 SQL片段

- 提炼代码
  - java:方法
  - 数据库：存储过程、存储函数
  - MyBatis：SQL片段

步骤：
1. 提取相似代码
2. 引用

```
<sql id="objectArrayStunos">
	<where>
	    <if test="array!=null and array.length>0 ">
	        <foreach collection="array" open=" and stuno in (" close=")" item="student" separator=",">
	            #{student.stuNo}
	        </foreach>
	    </if>
 	</where>
</sql>

<select id="queryStudentWithObjectArray" parameterType="Object[]" resultType="student">
 		select * from student
 	    
 	    <include refid="objectArrayStunos"></include>
 	    
</select>
```

注意：若SQL片段和引用处不在同一文件中，则需要在refid引用时加上namespace，即namespace.id


## 11.关联查询

### 11.1 一对一

##### 11.1.1 业务扩展类

- 核心：用resultType指定类的属性包含多表查询的所有字段

studentMapper.xml
```
<select id="queryStudentByNoWithOO" parameterType="int" resultType="StudentBusiness" >
    select s.*,c.* from student s inner join studentcard c
    on s.cardid=c.cardid
    where s.stuno = #{stuNo}
</select>
```

StudentBusiness.java
```
public class StudentBusiness extends Student{
    //学生业务扩展类
    private int cardId;
    private String cardInfo;
    get...
    set...
    public String toString(){
        return super.toString()+","+this.cardId+","+this.cardInfo;
    }
}
```

Student.java
```
public class Student{
    private int stuNo;
    private String stuName;
    private int stuAge;
    private String graName;
    private boolean stuSex;
    get...
    set...
}
```


StudentMapper.java
```
StudentBusiness queryStudentByNoWithOO(int stuno);
```

Test.java
```
StudentBusiness studentBusiness = studentMapper.queryStudentByNoWithOO(2);
```



##### 11.1.2 resultMap

1. 通过属性成员将2个类建立起联系
2. 一对一：association；一对多：collection

Student.java【学生类包含：学生信息、学生证信息】
```
public class Student{
    //学生信息
    private int stuNo;
    private String stuName;
    private int stuAge;
    private String graName;
    private boolean stuSex;
    //学生证信息
    private StudentCard card;
    get...
    set...
    public String toString(){
        return stuNo+"-"+this.stuName+"-"+this.stuAge+"-"+this.card.getCardId()+"-"+this.card.getCardInfo();
    }
}
```

StudentCard.java
```
public class StudentCard{
    private int CardId;
    private String cardInfo;
    get...
    set...
}
```


studentMapper.xml
```
<select id="queryStudentByNoWithOO2" parameterType="int" resultMap="student_card_map" >
    select s.*,c.* from student s inner join studentcard c
    on s.cardid=c.cardid
    where s.stuno = #{stuNo}
</select>

<resultMap type ="student" id="student_card_map">
    <!-- 学生的信息 -->
    <id property="stuNO" column="stuNo" />
    <result property="stuName" column="stuName" />
    <result property="stuAge" column="stuAge" />

    <!-- 一对一时，对象成员使用association映射；javaType指定该属性的类型 -->
    <association property="card" javaType="StudentCard" >
        <id property="cardId" column="cardId" />
        <result property="cardIdfo" column="cardIdfo" />
    </association>
</resultMap>
```

StudentMapper.java
```
Student queryStudentByNoWithOO2(int stuno);
```

Test.java
```
Student student = studentMapper.queryStudentByNoWithOO2(2);
```

### 11.2 一对多

- 表：student-studentclass【关联：classid】
- 类：Student-StudentClass【关联：LIst<Student> students】

StudentClass.java
```
public class StudentClass{
    private int classId;
    private String className;
    
    //增加学生属性（通过该字段让两个类建立起联系）
    List<Student> students;
    
    get...
    set...
}
```

Student.java
```
public class Student{
    //学生信息
    private int stuNo;
    private String stuName;
    private int stuAge;
    private String graName;
    private boolean stuSex;
    //学生证信息
    private StudentCard card;
    get...
    set...
    public String toString(){
        return stuNo+"-"+this.stuName+"-"+this.stuAge+"-"+this.card.getCardId()+"-"+this.card.getCardInfo();
    }
}
```

studentMapper.xml
```
<select id="queryClassAndStudents" parameterType="int" resultMap="class_student_map">
    <!-- 查询g1班的班级信息和g1班的所有学生信息 -->
    select c.*,s.* from student s inner join studentclass c
    on c.classid = s.classid
    where c.classid = #{classId}
</select>

<resultMap tyoe="StudentClass" id="class_student_map">
    <!-- 因type主类是班级，先配置班级的信息 -->
    <id property="classId" column="classId" />
    <result property="className" column="className" />
    
    <!-- 配置成员属性学生，一对多；属性类型：javaType，属性的元素类型ofType -->
    <collection property="students" ofType="student" />
        <id property="stuNo" column="stuNo" />
        <result property="stuName" column="stuName" />
        <result property="stuAge" column="stuAge" />
    </collection>
</resultMap>
```

StudentMapper.java
```
StudentClass queryClassAndStudents(int classId);
```

Test.java
```
//班级
StudentClass studentClass = studentMapper.queryClassAndStudents(1);
System.out.println(studentClass.getClassId()+","+studentClass.getClassName());
//班级对应的学生
List<Student> students = studentClass.getStudents();
for(Student student:students){
    System.out.println(student.getStuNo()+","+student.getStuName()+","student.getStuAge());
}
```

- MyBatis中多对一、多对多的本质是一对多的变化


## 12.Log4j

- 作用：可以通过日志信息，详细阅读MyBatis执行情况（观察mybatis实际执行的sql语句 以及sql中的参数和返回结果）

1. log4j.jar(mybatis.zip中lib中包含此jar)
2. 开启日志【conf.xml】
```
<settings>
    <!-- 开启日志，并指定使用的具体日志(LOG4J) -->
    <setting name="logImpl" value="LOG4J" /> 
</settings>
```

- 如果不指定，MyBatis就会根据以下顺序寻找日志:
  - SLF4J -> Apache Commons Logging -> Log4j 2 -> Log4j -> JDK logging

3. 编写配置日志输出文件

log4j.properties
```
log4j.rootLogger = DEBUG , stdout
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = %5p [%t] - %m%n
```

- 日志级别：DEBUG<INFO<WARN<ERROR
  - 若设置为info，则只显示info及以上级别的信息；
- 建议：开发时设置debug，运行时设置为info或以上；

## 13.延迟加载(懒加载)

一对一、一对多、多对一、多对多

- 延迟加载：如若一对多中，想要暂时只查询1的一方，先不查询多的一方，而是在需要的时候再去查询，即延迟加载；
  - 查询学生的sql是通过select属性指定的，并提供column指定外键；

- 例子：一对一中查询全部学生，并延迟加载每个学生对应的学生证信息
  - 通过debug发现：如果程序只需要学生，则只向数据库发送了查询学生的SQL，当后续需要用到学生证时，再第二次发送查询学生证的SQL

- 一对多：和一对一的延迟加载配置方法相同；

conf.xml
```
<settings>
    <!-- 开启延迟加载 -->
    <setting name="lazyLoadingEnabled" value="true" /> 
    
    <!-- 关闭立即加载 -->
    <setting name="aggressiveLazyLoading" value="false" /> 
</settings>

<mappers>
    <mapper resource="org/lanqiao/mapper/studentCardMapper.xml" />
</mappers>
```

studentMapper.xml
```
<select id="queryStudentWithOO2" parameterType="int" resultMap="student_card_lazyLoad_map" >
    select * from student 
</select>

<resultMap type ="student" id="student_card_lazyLoad_map">
    <!-- 学生的信息 -->
    <id property="stuNO" column="stuNo" />
    <result property="stuName" column="stuName" />
    <result property="stuAge" column="stuAge" />

    <!-- 此次采用延迟加载：在查询学生时，并不立即加载学生证信息 -->
    <association property="card" javaType="StudentCard" select="org.lanqiao.mapper.StudentCardMapper.queryCardById" column="cardid">
        <!--
        <id property="cardId" column="cardId" />
        <result property="cardIdfo" column="cardIdfo" />
        -->
    </association>
</resultMap>
```

studentCardMapper.xml【增加了mapper.xml，要修改conf.xml配置文件，即将新增的mapper.xml加载进去】
```
<mapper namespace="org.lanqiao.mapper.StudentCardMapper">
    <select id="queryCardById" parameterType="int" resultType="studentCard">
        select * from studentCard where cardid = #{cardId}
    </select>   
</mapper>
```

StudentMapper.java
```
List<Student> queryStudentWithOO2();
```

Test.java
```
List<Student> students = studentMapper.queryStudentWithOO2();
for(Student student:students){
    System.out.println(student.getStuNo()+","+student.getStudentName());
    StudentCard card = student.getCard();
    System.out.println(card.getCardId()+","+card.getCardInfo());
}
```

延迟加载的步骤(先查班级，按需查询学生)：
- 开启延迟加载，conf.xml配置settings
- 配置mapper.xml
  - 班级mapper.xml,
  - 学生mapper.xml
  

## 14.查询缓存

### 14.1 一级缓存

- 范围：同一SqlSession对象
- MyBatis默认开启一级缓存；
- 若用同样的SqlSession对象查询相同的数据，则只会在第一次查询时向数据库发送SQL语句，并将查询的结果放入SqlSession中，作为缓存存在；
- 后续再次查询该相同的对象时，则直接从缓存中查询该对象即可，即省略了数据库的访问；
- 若提交commit()，则会清理所有的缓存对象；

### 14.2 二级缓存

MyBatis自带二级缓存
- 范围：【同一namespace】生成的mapper对象
  - 若有多个xxMapper.xml的namespace值相同，则通过这些xxMapper.xml产生的xxMapper对象仍然共享二级缓存；
  - 回顾：namespace的值就是接口的全类名（包名.类名），通过接口可以产生代理对象（studentMapper对象），即namespace决定了studentMapper对象的产生；
- 触发将对象写入二级缓存的时机：执行SqlSession对象的close()方法；
  - 用同样的SqlSession对象查询相同数据时，共享一级缓存，此时未写入二级缓存，同一namespace而不同SqlSession对象查询数据时，只当colse()时触发，数据才写入二级缓存;
- MyBatis默认没有开启二级缓存，需要手工打开
1. conf.xml
```
<settings>
    <!-- 开启二级缓存 -->
    <setting name="cacheEnabled" value="true" />
</settings>
```

2. 在具体的mapper.xml中声明开启
```
<mapper namespace="org.lanqiao.mapper.StudentMapper">
    <!-- 声明此namespace开启二级缓存,MyBatis自带的二级缓存 -->
    <cache/>
```

- MyBatis的二级缓存是将对象放入硬盘文件中，准备缓存的对象必须实现了序列化接口
- 序列化对象为student，则需要序列化Student类，以及Student的级联属性，和父类
- 【异常提示：NotSerializableException】
  - 序列化：内存 -> 硬盘
  - 反序列化：硬盘 -> 内存
```
public calss Student implements Serializable{
    private StudentCard card;     //级联属性，也需序列化
}
```

##### 14.2.1 禁用二级缓存

- select标签中useCache="false"

studentMapper.xml
```
<!-- 禁用此select的二级缓存 -->
<select id="queryStudentBuStuno" parameter="int" resultMap="student" useCache="false" useCache="false">
    select * from student where stuno = ${value}
</select>
```

##### 14.2.2 清理二级缓存

- 方法一：与清理一级缓存的方法相同：commit();
  - 一般执行增删改时，会清理掉缓存【防止脏数据】；
  - commit会清理一级和二级缓存，但是清理二级缓存时，不能是查询自身的commit；
- 方法二：在select标签中增加属性flushCache="true"

##### 14.2.3 三方提供的二级缓存

ehcache、memcache
- 要想整合三方提供的/自定义的二级缓存，必须实现org.aoache.ibatis.cache.Cache接口，该接口的默认实现类是PerpetualCache

整合ehcache二级缓存
1. 导入jar包
  - Ehcache-core.jar
  - mybatis-Ehcache.jar
  - slf4j-api.jar

2. 编写ehcache配置文件Ehcache.xml

Ehcache.xml【copy from 蓝桥软件学院】
```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <1-- 当二级缓存的对象超过内存限制时(缓存对象的个数>maxElementsInMemory)，存放入的硬盘文件 -->
    <diskStore path="D:\Ehcache" />

<!--
    maxElementsInMemory:设置在内存中缓存对象的个数
    maxElementsOnDisk：设置在硬盘中缓存对象的个数
    eternal：设置缓存是否永远不过期
    overflowToDisk：当内存中缓存的对象个数超过maxElementsInMemory的时候，是否转移到硬盘
    timeToIdleSeconds：当2次访问超过该值的时候，将缓存对象失效
    timeToLiveSeconds：一个缓存对象最多存放的时间（生命周期）
    diskExpiryThreadIntervalSeconds：设置每隔多长时间，通过一个线程来清理硬盘中的缓存
    memoryStoreEvictionPolicy：当超过缓存对象的最大值时，处理的策略;LRU,FIFO,LFU
-->

    <defaultCache
            maxElementsInMemory="1000"
            maxElementsOnDisk="1000000"
            eternal="false"
            overflowToDisk="false"
            timeToIdleSeconds="100"
            timeToLiveSeconds="100"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU" >
    </defaultCache>
  
</ehcache>
```

3. 开启EhCache二级缓存

```
<mapper namespace="org.lanqiao.mapper.StudentMapper">
    <!-- type的值为实现接口的类 -->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <!-- 通过property覆盖Ehcache.xml中的值 -->
        <property anme="maxElementsInMemory" value="2000" />
    </cache>
```

## 15.逆向工程

- 表、类、接口、mapper.xml四者密切相关，因此当知道一个的时候，其他三个应该可以自动生成；

表->其他三个，实现步骤：
1. mybatis-generator-core.jar、mybatis.jar、ojdbc.jar
2. 逆向工程的配置文件generator.xml【copy from 蓝桥软件学院】

```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
      PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
      "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    <generatorConfiguration>
     <context id="DB2Tables" targetRuntime="MyBatis3">
      <commentGenerator>
       <!--
    				suppressAllComments属性值：
    					true:自动生成实体类、SQL映射文件时没有注释
    					true:自动生成实体类、SQL映射文件，并附有注释
    			  -->
       <property name="suppressAllComments" value="true" />
      </commentGenerator>
      <!-- 数据库连接信息 -->
      <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
       connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:XE" 
    userId="system"  password="sa">
      </jdbcConnection>
      <!-- 
    			forceBigDecimals属性值： 
    				true:把数据表中的DECIMAL和NUMERIC类型，
    解析为JAVA代码中的java.math.BigDecimal类型 
    				false(默认):把数据表中的DECIMAL和NUMERIC类型，
    解析为解析为JAVA代码中的Integer类型 
    		-->
      <javaTypeResolver>
       <property name="forceBigDecimals" value="false" />
      </javaTypeResolver>
      <!-- 
    			targetProject属性值:实体类的生成位置  
    			targetPackage属性值：实体类所在包的路径
    		-->
      <javaModelGenerator targetPackage="org.lanqiao.entity"
                                 targetProject=".\src">
       <!-- trimStrings属性值：
    				true：对数据库的查询结果进行trim操作
    				false(默认)：不进行trim操作
    			  -->
       <property name="trimStrings" value="true" />
      </javaModelGenerator>
      <!-- 
    			targetProject属性值:SQL映射文件的生成位置  
    			targetPackage属性值：SQL映射文件所在包的路径
    		-->
      <sqlMapGenerator targetPackage="org.lanqiao.mapper" 
    targetProject=".\src">
      </sqlMapGenerator>
      <!-- 生成动态代理的接口  -->
      <javaClientGenerator type="XMLMAPPER" targetPackage="org.lanqiao.mapper" targetProject=".\src">
      </javaClientGenerator>
      <!-- 指定数据库表  -->
      <table tableName="Student"> </table>
      <table tableName="studentCard"> </table>
      <table tableName="studentClass"> </table>
     </context>
    </generatorConfiguration>
```

3. 执行

```
public class Test{
    public static void main(String[] args){
        File file = new File("src/generator.xml");   //配置文件
        List<String> warnings = new ArrayList<>();
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(file);
        DefaultShellCallback callBack = new DefaultShellCallback(true);
        
        MyBatisGenerator generator = new MyBatisGenerator(config,callBack,warnings);
        generator.generate(null);
    }
}
```
