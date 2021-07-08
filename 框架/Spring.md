[toc]

# 0.补充

### 0.1 简介

**优点：**
- Spring是一个开源的免费的框架（容器）
- Spring 所以是一个轻量级的、非入侵式的框架
- 控制反转（IOC）、面向切面编程（AOP）
- 支持事务的处理，对框架整合的支持


**总结：** Spring是一个轻量级的控制反转和面向切面编程的框架。

> 弊端：配置繁琐

### 0.2 IOC

**控制反转IOC（Inversion of Control）**
- 是一种设计思想，Spring中实现控制反转的是IOC容器，实现方式是依赖注入（DI）。
- 这种思想从本质上解决问题，程序员不再管理对象的创建，而是更多地关注业务实现。对象由Spring来创建、管理、装配。。耦合性大大降低。
- Spring容器在初始化时先读取配置文件，根据配置文件或元数据创建与组织对象存入容器中，程序使用时再从IOC容器中取出需要的对象。

**依赖注入**
- 依赖：bean对象的创建依赖于容器
- 注入：bean对象中的所有属性，由容器来注入


**IOC创建对象（构造器注入）**【配置文件中配置bean，name可取多个别名】
- 默认使用无参构造创建对象
- 通过配置使用有参构造
- 配置文件加载时，容器中管理的对象就已经初始化了

**set方式注入**


**bean的作用域**
- 单例模式【默认】：scope="singleton"
- 原型模式：scope="prototype"
  - 每次从容器中get，都产生新的对象
- ...


**Bean的自动装配**
- ① XML
  - <bean id="people" class="..." autowired="byName/byType/..">
- ② 注解@Autowied
  - 添加配置文件约束context
  - 需要加<context:annotation-config/>支持
  - 配合@Qualifier
 

**@Resource 和 @Autowired的区别：**
- 都用来自动装配，放在属性字段上
- @Autowired 默认通过byType的方式实现，而且必须要求这个对象存在；若无法通过属性自动装配，则需通过@Qualifier()
- @Resource 默认通过byName的方式实现；若找不到名字，则通过byType实现；若都找不到，则报错


- @Component：组件，放在类上，说明这个类被Spring管理了。【就是Bean】
- @Value：相当于配置中Bean name value


**代理模式的好处**
- 可以使真实角色的操作更加纯粹，不用去关注一些公共的业务
- 公共业务交给代理角色，实现了业务的分工
- 公共业务发生扩展的时候，方便集中管理

**代理模式的缺点**
- 一个真实角色就会产生一个代理角色；代码量会翻倍，开发效率会变低

**动态代理**
- 底层通过反射实现

![avatar](D:/其他/图片/study/动态代理.png)


**AOP**
- 提供声明式事务； 允许用户自定义切面
- 实现机制是动态代理
- 实现方式：
  - 方式①：使用Spring的API接口
  - 方式②：自定义实现
  - 方式③：使用注解实现@Aspect @Before

**Spring声明式事务**

![avatar](D:/其他/图片/study/Spring声明式事务.png)

编程式事务（手动回滚）


# 1.搭建Spring环境

<Expoer One-on-one j2eedevelopment and Design>--Rod Johnon--2002

2003年Spring，基础特性：IOC、Aop

### 1.1 下载jar

> http://maven.springframework.org/release/org/springframework/spring/下载spring-framework-4.3.9.RELEASE-dist.zip 

开发spring至少需要使用的jar（5+1）：
- spring-aop.jar【开发AOP特性时所需jar】
- spring-beans.jar【处理Bean的jar】
- spring-context.jar【处理spring上下文的jar】
- spring-core.jar【spring核心jar】
- spring-expression.jar【spring表达式】
- commons-logging.jar【日志】【第三方提供的日志jar】

### 1.2.STS工具

为了编写时有一些提示、自动生成一些配置信息：

- 方式一【版本对应】
  - 可以给eclipse增加支持spring的插件：spring tool suite
  - 【https://spring.io/tools/sts/all下载springsource-tool-suite-X.X.X.RELEASE-eX.X.X-updatesite.zip然后在eclipse中安装】
- 方式二
  - 直接下载sts工具（相当于eclipse+插件spring tool suite）
  - 【https://spring.io/tools/sts/】

> 以上，发现版本更新后事实不符，因此我找到STS3.9.11下载使用

新建：new spring bean configuration file-->applicationContext.==xml==

# 2.第一个Spring程序（IOC）

### 2.1 applicationContext.xml

- 该文件中产生的所有对象，被spring放入spring ioc容器
- id:唯一标识符
- class：指定类型
- property：该class所代表的类的属性
- name：属性名
- value：属性值

```
    <bean id="student" class="org.entity.Student">
		<property name="stuNo" value="1"></property>
		<property name="stuName" value="zs"></property>
		<property name="stuAge" value="23"></property>
	</bean>
```

### 2.2 Test.java

- context:Spring上下文对象
- context.getBean("xxx")：从springIOC中获取一个id为student的对象


```
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
	Student student = (Student)context.getBean("student");
	System.out.println(student);
```

### 2.3 Student.java

```
public class Student{
    private int stuNo;
	private String stuName;
	private int stuAge;
	
	get...
	set...
	toString()...
}
```


# 3. IOC

### 3.1 控制反转\依赖注入

- ==控制反转==：反转的是获取对象的方式：从自己通过new产生对象，到通过getBean直接从SpringIOC容器(applicationContext.xml)中获取
- 为了更加清晰的理解==IOC==，IOC被更名为==DI==（依赖注入）
- ==依赖注入==：将属性值注入给了属性，将属性注入给了bean，将bean注入给了IOC容器；
- 在IOC容器中定义bean的前提;该bean的类必须提供了==无参构造==

### 3.2 SpringIOC发展史：

> ① Student student = new Student();  student.setXxx();
> 
> ② 简单工厂
> 
> ③ ioc（超级工厂）

- ① 普通方式创建对象的new非常零散，造成后期维护较为麻烦
- ② 通过“简单工厂”，可将创建某一类对象的new集中起来操作，方便后期维护【需自己编写】
- ③ SpringIOC容器（超级工厂），可存放任何对象【提供】


- IOC容器帮我们new了对象，并且给对象赋值；
- 之后的IOC分为2步：①先给springioc容器中存放对象并赋值 ②拿


### 3.3 依赖注入的三种方式

##### 3.3.1 set注入
> 通过setXxx()赋值

- IOC容器赋值时：
  - 若是简单类型（8个基本类型+String）,用value
  - 若是对象类型，ref="需要引用的id值"，实现了对象与对象之间的依赖关系

```
    <bean id="teacher" class="org.entity.Teacher">
		<property name="name" value="zs"></property>
		<property name="age" value="23"></property>
	</bean>
	
	<bean id="course" class="org.entity.Course">
		<property name="courseName" value="java"></property>
		<property name="courseHour" value="200"></property>
		<!-- 将Teacher对象注入到cuorse对象中 -->
		<property name="teacher" ref="teacher"></property>
	</bean>
```

> 依赖注入底层是通过反射实现的【调用set()方法】
<property..>

##### 3.3.2 构造器注入

> 通过构造方法赋值

```
    <bean id="teacher" class="org.entity.Teacher">
        <constructor-arg value="ls" type="String" index="0" name="name"></constructor-arg>
		<constructor-arg value="24"></constructor-arg>
	</bean>
	
	<bean id="course" class="org.entity.Course">
	    <constructor-arg value="c"></constructor-arg>
		<constructor-arg value="100"></constructor-arg>
		<constructor-arg ref="teacher"></constructor-arg>	
	</bean>
```

- 注意：若<constructor-arg>的顺序与构造参数的顺序不一致，则需要通过type或index或name指定

##### 3.3.3 p命名空间注入

- 引入p命名空间【xmlns:p="http://www.springframework.org/schema/p"】

```
<bean id="teacher" class="org.entity.Teacher" p:age="25" p:name="ww">
</bean>

<bean id="course" class="org.entity.Course" p:courseName="hadoop" p:courseHour="300" p:teacher-ref="teacher">
</bean>

```

> 简单类型：p:属性名="属性值"
> 
> 引用类型（除了String以外）：p:属性名-ref="属性值"

- 注意：无论是String还是Int/short/long，在赋值时都是value="值"，因此建议此种情况需要配合name\type进行区分


### 3.4 注入集合数据类型

applicationContext.xml
```
<bean id="collectionDemo" class="org.entity.AllCollectionType">
		<property name="listElement">
			<list>
				<value>足球1</value>
				<value>篮球1</value>
				<value>乒乓球1</value>
			</list>
		</property>
		
		<property name="arrayElement">
			<array>
				<value>足球2</value>
				<value>篮球2</value>
				<value>乒乓球2</value>
			</array>		
		</property>
		
		<property name="setElement">
			<set>
				<value>足球3</value>
				<value>篮球3</value>
				<value>乒乓球3</value>
			</set>
		</property>
		
		<property name="mapElement">
			<map>
				<entry>
					<key>
						<value>zuqiu4</value>
					</key>
					<value>足球4</value>
				</entry>
				<entry>
					<key>
						<value>lanqiu4</value>
					</key>
					<value>篮球4</value>
				</entry>
				<entry>
					<key>
						<value>ppq4</value>
					</key>
					<value>乒乓球4</value>
				</entry>
			</map>
		</property>
		
		<property name="propsElement">
			<props>
				<prop key="zuqiu5">足球5</prop>
				<prop key="lanqiu5">篮球5</prop>
				<prop key="ppq5">乒乓球5</prop>
			</props>
		</property>
	
	</bean>
```

AllcollectionType.java
```
public class AllCollectionType {
	private List<String> listElement;
	private String[] arrayElement;
	private Set<String> setElement;
	private Map<String,String> mapElement;
	private Properties propsElement;
	
	get...
	set...
	
	public String toString() {
		String strContent = "";
		for(String str:arrayElement) {
			strContent += str + "," ;
		}
		return "list:"+this.listElement+"\nset:"+this.setElement+"\nmap:"+this.mapElement+"\nprops:"+this.propsElement+"\narray:"+strContent;
	}
	

}

```

- 注：set、list、数组 各自都有自己的标签，但也可以混着用


### 3.5 value与<value>注入方式的区别


* | 使用value属性注入 | 使用子元素<value>注入
--- | --- | ---
参数值位置 | 写在value的属性值中【必须加双引号】 | 写在首尾标签（<value></value>）的中间【不加双引号】
type属性 | 无 | 有【可选】
参数值包含特殊字符(<,>,&) 时的处理方法| 使用XML预定义的实体引用 | ① 使用XML预定义的实体引用 ② 使用<![CDATA[ ]]>标记

其中，XML预定义的实体引用如下所示：

实体引用 | 表示的符号
---|---
&lt；| <
&gt； | >
&amp； | &


```
	<bean id="teacher" class="org.entity.Teacher">
		<property name="name">
			<value type="java.lang.String">z<![CDATA[<>&]]>s</value>
		</property>
```

给对象类型赋值null【注意没有value】
```
    <property name="name">
		 <null/>
	</property>
```

给对象类型赋空值""
```
    <property name="name">
		 <value></value>
	</property>
```

### 3.6 自动装配【只适用于ref】

- 约定优于配置
- 虽然可以减少代码量，但是会降低程序的可读性，使用需谨慎
- autowire="byName/byType/constructor":
  - byName：自动寻找其他bean的id值=该类的属性名【本质是byId】
  - byType：自动寻找其他bean的类型（class）与该类的ref属性类型一致【必须满足：当前ioc容器中只能有一个Bean满足条件】
  - constructor：自动寻找其他bean的类型（class）与该类的构造方法参数类型一致【本质就是byType】

```
    <bean id="course" class="org.entity.Course" autowire="byName">
			<property name="courseName" value="java"></property>
			<property name="courseHour" value="200"></property>
<!-- 			<property name="teacher" ref="teacher"></property>			 -->
		</bean>
```
> Course类中有一个ref属性的teacher(属性名)，并且该ioc容器中恰好有一个bean的id值与之相同

可以在头文件中一次性将该ioc容器的所有bean统一设置成自动装配：<beans......	default-autowire="byName">

### 3.7 注解形式的依赖注入

使用注解定义bean：通过注解的形式将bean以及相应的属性值放入ioc容器

##### 3.7.1 注解

@Component("xxx")
- xxx相当于id，范围大，可细化如下：
  - @Repository：dao层注解
  - @Service：service层注解
  - @Controller：控制器层注解
  

##### 3.7.2 配置扫描器  

- <context:component-scan base-package="xxx"></context:component-scan>【xxx相当于class】
- Spring在启动时，会根据base-package，扫描该包中所有类，查找这些类是否有注解@Component("studentDao")，如果有，则将该类加入spring ioc 容器


##### 3.7.3 注入属性值

- @Autowired:byType【自动装配】
- @Qualifier("stuDao")：byName（byId）

### 3.8 使用注解实现声明式事务

目标：通过注解使以下方法要么全成功，要么全失败

```
public void addStudent(){
    //增加班级
    //增加学生
    //crdu
}
```

##### 3.8.1 jar包

- spring-tx-4.3.9.RELEASE.jar
- mysql-connector-java-8.0.18.jar【Oracle用ojdbc.jar】
- commons-dbcp.jar  连接池使用的数据源
- commons-pool.jar  连接池
- spring-jdbc-4.3.9.RELEASE.jar
- aopalliance.jar

##### 3.8.2 配置

> jdbc\mybatis\spring

先增加事务tx的命名空间，再：
```
<!-- 配置数据库相关 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
		<property name="url" value="jdbc:mysql://localhost:3306/xs?serverTimezone=UTC"></property>
		<property name="username" value="root"></property>
		<property name="password" value="123456"></property>
		<property name="maxActive" value="10"></property>
		<property name="maxIdle" value="6"></property>
	</bean>
	
	<!-- 配置事务管理器txManager -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 增加对事务的支持 -->
	<tx:annotation-driven transaction-manager="txManager" />
```

- 注：url加上“?serverTimezone=UTC”，避免出现异常；

##### 3.8.3 使用 

==service依赖dao==

将需要成为事务的方法前增加注解：@Transactional(readOnly = false,propagation = Propagation.REQUIRED)

# 4. AOP【面向方面编程】


- 切面：一个横切功能的模块化，这个功能可能会横切多个对象（业务），如aMethod()方法就是一个“切面”，它横切导论多个业务之中；
- 切入点：可以插入“横切逻辑（如aMethod()）”的方法，如“调用add()”就是一个切点；
- 通知：
  - 前置通知：在切入点add()方法执行之前，插入的通知；
  - 后置通知：在切入点add()方法执行完毕之后，插入的通知；
  - 异常通知：当切入点add()方法抛出异常时，插入的通知；
  - 最终通知：当切入点add()方法执行完毕时，插入的通知；
  - 环绕通知：可以贯穿切入点add()方法执行的整个过程；


- 一个普通的类-->有特定功能的类 的四种方法：

> 继承类
> 
> 实现接口
> 
> 注解
> 
> 配置

## 4.1 通过实现接口实现通知

通知类型 | 需要实现的接口 | 接口中的方法 | 执行时机
---|---|---|---
前置通知 | org.springframework.aop.MethodBeforeAdvice | before() | 目标方法执行前
后置通知 | org.springframework.aop.AfterReturningAdvice | afterReturning() | 目标方法执行后
异常通知 | org.springframework.aop.ThrowsAdvice | 无 | 目标方法发生异常时
环绕通知 | org.springframework.aop.MethodInterceptor | invoke() | 拦截对目标方法调用，即调用目标方法的整个过程


### 4.1.1 前置通知

> 通用的开发流程：jar--配置--使用【编写】

##### 4.1.1.1 jar

- aopaliance.jar
- aspectjweaver.jar

##### 4.1.1.2 编写


- aop：每当执行add()之前自动执行一个方法before()
- add():业务方法【IStudentService.java中的addStudent()】
- before():自动执行的通知，即aop前置通知【LogBefore.java中的before()】

【StudentServiceImpl.java】
```
public class StudentServiceImpl implements IStudentService{
	
	IStudentDao studentDao ;

	public void setStudentDao(IStudentDao studentDao) {
		this.studentDao = studentDao;
	}

	@Transactional(readOnly = false,propagation = Propagation.REQUIRED)
	@Override
	public void addStudent(Student student) {
		studentDao.addStudent(student);
	}

}
```

【LogBefore.java】
```
public class LogBefore implements MethodBeforeAdvice{

	//前置通知的具体内容
	@Override
	public void before(Method method, Object[] args, Object target) throws Throwable {
		System.out.println("前置通知...");		
	}

}
```


##### 4.1.1.3 配置

【applicationContext.xml】
```
    <bean id="studentDao" class="org.dao.Impl.StudentDaoImpl">
	</bean>
	
	<!-- 配置前置通知 -->
	
	<!-- addStudent()所在类 -->
	<bean id="studentService" class="org.service.StudentServiceImpl">
		<property name="studentDao" ref="studentDao"></property>
	</bean>
	
	<!-- “前置通知”类 -->
	<bean id="LogBefore" class="org.aop.LogBefore">
	</bean>
	
	<!-- 将addStudent()和通知进行关联 -->
	<aop:config>
		<!-- 配置切入点（在哪里执行通知） -->
		<aop:pointcut expression="execution(public void org.service.StudentServiceImpl.addStudent(Student);)" id="pointcut"/>
		<!-- advisor：相当于链接切入点和切面的线 -->
		<aop:advisor advice-ref="LogBefore" pointcut-ref="pointcut"/>
	</aop:config>
```

> 注：以上为一个切面对应一个切点，也可对应多个切点，即在多个方法前执行before()，expression表达式如下

```
expression="execution(...) or execution(...)" 
```

##### 4.1.1.4 expression表达式常见示例


举例 | 含义
---|---
public boolean addStudent(org.entuty.Student)) | 所有对应返回类型、参数类型的对应方法
public boolean org.service.IStudentService.addStudent(org.entuty.Student)) | 所有对应返回类型、参数类型、对应类（或接口）中的对应方法
public * addStudent(org.entuty.Student)) | *表示任意返回类型
public void *(org.entuty.Student)) | *表示任意方法名
public void addStudent(..)) | “..”代表任意参数列表
* org.service.IStudentService.*.*(..)) | 对应包中包含的所有方法（不包含子包中的方法）
* org.service.IStudentService..*.*(..)) | 对应包中包含的所有方法（包含子包中的方法）



- 注：若出现异常类似以下，则说明缺少jar
```
could not find class that it depends on; nested exception is java.lang.NoClassDefFoundError: org/apache/commons/pool/ObjectPool
```

### 4.1.2 后置通知

- ① 通知类，普通类实现接口
```
public class LogAfter implements AfterReturningAdvice{
	@Override
	public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
		System.out.println("后置通知：目标对象："+target+"，调用的方法名："+method.getName()+",方法的参数个数："+args.length+"方法的返回值："+returnValue);
	}
}
```

- ② 业务类、业务方法【StudentServiceImpl.java中的addStudent()】

- ③ 配置
  - 将业务类、通知纳入springIOC容器
  - 配置切入点（一端）、配置通知类（另一端），通过pointcut-ref将两端连接起来
```
	<aop:config>
		<!-- 配置切入点（连接线的一端：业务类的具体方法） -->
		<aop:pointcut expression="execution(public void org.service.StudentServiceImpl.addStudent(org.entity.Student))" id="pointcut2"/>
		<!-- （连接线的另一端：通知类） -->
		<aop:advisor advice-ref="org.aop.LogAfter" pointcut-ref="pointcut2"/>
	</aop:config>
	
```

### 4.1.3 异常通知

根据异常通知的定义可以发现，异常通知的实现类必须编写以下方法
-public void afterThrowing([Method, args, target], ThrowableSubclass)【两种情况】:
  - ① public void afterThrowing(ThrowableSubclass);
  - 或② public void afterThrowing(Method, args, target, ThrowableSubclass);

① 通知类
```
public class LogException implements ThrowsAdvice{
	//异常通知的具体方法
	public void afterThrowing(Method method, Object[] args, Object target, Throwable ex) {
		System.out.println("异常通知：目标对象："+target+"，方法名："+method.getName()+",方法的参数个数："+args.length+",异常类型："+ex.getMessage());
	}
}
```
② 业务类、业务方法

③ 配置
```
	<bean id="LogException" class="org.aop.LogException">
	</bean>
	
	<aop:config>
		<!-- 配置切入点（连接线的一端：业务类的具体方法） -->
		<aop:pointcut expression="execution(public void org.service.StudentServiceImpl.addStudent(org.entity.Student))" id="pointcut3"/>
		<!-- （连接线的另一端：通知类） -->
		<aop:advisor advice-ref="LogException" pointcut-ref="pointcut3"/>
	</aop:config>
```

### 4.1.4 环绕通知

在目标方法的前后、异常发生时、最终等各个地方都可以进行的通知，最强大的一个通知；
- 可以获取目标方法的全部控制权：
  - 目标方法是否执行、执行之前、执行之后、参数、返回值等
  - 使用环绕通知时，目标方法的一切信息都可以通过参数invocation获取到

① 通知类
- result = invocation.proceed();
  - 此方法控制着目标方法addStudent()的执行，result是方法的返回值
  - 此前的代码为前置通知，此后的代码为后置通知
```
public class LogAround implements MethodInterceptor{

	@Override
	public Object invoke(MethodInvocation invocation) throws Throwable {
		Object result = null;
		//方法体1
		try {
			//方法体2
			System.out.println("用环绕通知实现的【前置通知】");
			
			result = invocation.proceed();
			//此方法控制着目标方法addStudent()的执行，result是方法的返回值
			//此前的代码为前置通知，此后的代码为后置通知
			
			System.out.println("用环绕通知实现的【后置通知】");
			System.out.println("目标对象target:"+invocation.getThis()+"，调用的方法名："+invocation.getMethod().getName()+",方法的参数个数："+invocation.getArguments().length+"方法的返回值："+result);
			
		}catch(Exception e) {
			//方法体3
		}
		return null;
	}

}
```
② 业务类、业务方法

③ 配置
```
	<!-- 将环绕通知增加到ioc容器 -->
	<bean id="LogAround" class="org.aop.LogAround"></bean>
	
	<aop:config>
		<aop:pointcut expression="execution(public void org.service.Impl.StudentServiceImpl.addStudent(org.entity.Student))" id="pointcut4"/>
		<!-- （连接线的另一端：通知类） -->
		<aop:advisor advice-ref="LogAround" pointcut-ref="pointcut4"/>
	</aop:config>
```

> 环绕通知底层是通过拦截器实现的

## 4.2 通过注解实现通知

### 4.2.1 jar

与实现接口方式相同


### 4.2.2 配置

- 开启注解对AOP的支持
- 通过注解形式将对象增加到IOC容器时需要设置扫描器

> 扫描器会将指定的包中的【@Componet @Service @Respository @Controller】修饰的类产生的对象增加到IOC容器中

> @Aspect不需要加入扫描器，只需要开启即可

```
<!-- 开启注解对AOP的支持 -->
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
<!-- 设置扫描器 -->
	<context:component-scan base-package="org.aop"></context:component-scan>
```

### 4.2.3 编写

- 通过注解形式实现的aop，若想获取目标对象的一些参数，则需要使用一个对象：==JoinPoint==【环绕通知用其子类==ProceedingJoinPoint==】
- 参数returningValue是返回值，但需声明【returning="returningValue"】
- 异常通知中，若只捕获特定类型的异常，则可通过第二个参数实现【xxException e】,也需声明【throwing="e"】
- 注解形式实现aop时，通知的方法的参数不能多/少

```
@Component("logAnnotation")	 //将LogAspectAnnotation纳入springIOC容器中【相当于<bean id="" class="">】
@Aspect //此类是一个通知
public class LogAspectAnnotation {
	
	//前置通知
	@Before("execution(public * addStudent(..))")	//属性：定义切点
	public void myBefore(JoinPoint jp) {
		System.out.println("《注解形式-前置通知》:目标对象："+jp.getTarget()+",方法名："+jp.getSignature().getName()+"，参数列表："+Arrays.toString(jp.getArgs()));
	}
	
	//后置通知
	@AfterReturning(pointcut="execution(public * addStudent(..))" ,returning="returningValue")
	public void myAfter(JoinPoint jp,Object returningValue) {
		System.out.println("《注解形式-后置通知》:目标对象："+jp.getTarget()+",方法名："+jp.getSignature().getName()+"，参数列表："+Arrays.toString(jp.getArgs())+"返回值："+returningValue);
	}
	
	//异常通知
	@AfterThrowing(pointcut="execution(public * addStudent(..))" ,throwing="e")
	public void myException(NullPointerException e) {
		System.out.println("《注解形式-异常通知》--e:"+e.getMessage());
	}
	
	//环绕通知
	@Around("execution(public * addStudent(..))")
	public void myAround(ProceedingJoinPoint jp) {
	        //前置通知
			System.out.println("【环绕】：前置通知");
		try {
			//执行方法
			jp.proceed();
			
			//后置通知
			System.out.println("【环绕】：后置通知");
		}catch(Throwable e) {
			//异常通知
			System.out.println("【环绕】：异常通知");
		}finally {
			//最终通知
			System.out.println("【环绕】：最终通知");
			
		}
	}
	
	//最终通知
	@After("execution(public * addStudent(..))")	//属性：定义切点
	public void myAfter() {
		System.out.println("[myAfter]注解形式---最终通知");
	}
	
}
```

## 4.3 通过配置实现通知

基于Schema配置

- 获取目标对象信息：
  - 注解、Schema：JoinPoint
  - 接口：Method method，Object[] qrgs,Object target

“通知”
```
public class LogSchema {
	//后置通知
	public void myAfter(JoinPoint jp,Object returningValue) {
		System.out.println("》》》》》后置通知:目标对象："+jp.getThis()+",方法名："+jp.getSignature().getName()+"，参数个数："+jp.getArgs().length+"返回值："+returningValue);
	}
	
	//前置通知
	public void before() {
		System.out.println("》》》》》前置通知。。。");
	}
	
	//异常通知
	public void whenException(JoinPoint jp,NullPointerException e) {
		System.out.println("》》》》》异常通知："+e.getMessage());
	}
	
	//环绕通知
	public Object around(ProceedingJoinPoint jp) {
		Object result = null;
		System.out.println("》》》》》环绕通知：前置通知。。");
		try {
			result = jp.proceed();
			System.out.println(">>>>>"+jp.getSignature().getName()+","+result);
			System.out.println("》》》》》环绕通知：后置通知。。");
		}catch(Throwable e) {
			System.out.println("》》》》》环绕通知：异常通知。。");
		}
		return result;
	}
}
```

配置
```
	<bean id="LogSchema" class="org.aop.LogSchema">
	</bean>
	<aop:config>
		<!-- 配置切入点（连接线的一端：业务类的具体方法） -->
		<aop:pointcut expression="execution(public void org.service.Impl.StudentServiceImpl.addStudent(org.entity.Student))" id="schema"/>
		
		<!-- （连接线的另一端：通知类） 【schema方式】-->
		<aop:aspect ref="LogSchema">
			<!-- 连接线：连接业务类和通知before -->
			<aop:before method="before" pointcut-ref="schema"/>
		
			<!-- 连接线：连接业务类和通知afterReturning -->
			<aop:after-returning method="myAfter" returning="returningValue" pointcut-ref="schema"/>
			
			<!-- 连接线：连接业务类和通知whenException -->
			<aop:after-throwing method="whenException" throwing="e" pointcut-ref="schema"/>
			
			<!-- 连接线：连接业务类和通知around -->
			<aop:around method="around" pointcut-ref="schema"/>
			
		</aop:aspect>
	</aop:config>
```



# 5. Spring开发web项目 及拆分Spring配置文件


> 用spring开发web项目至少需要7个jar：spring-java的6个jar + spring-web.jar

> 注意：web项目的jar包是存入WEB-INF中

> 使tomcat支持项目：BuildPath-libraries-add library-server runtime-tomcat

- SpringIOC容器初始化：
  - 时机【Java程序中】：new ClassPathXmlApplicationContext("applicationContext.xml");【可放在main里->只初始化一次，而Web项目没有统一的入口】
  - 将ioc容器中的所有bean实例化为对象
  - 将各个bean依赖的属性值注入进去


## 5.1 初始化SpringIOC容器

Web项目如何初始化SpringIOC容器？
- 思路：当服务启动时（tomcat），通过监听器将SpringIOC容器初始化一次【该监听器spring-web.jar已经提供】
- web项目启动时会自动加载web.xml，使得同时自动实例化IOC容器，需要在web.xml中加载监听器（ioc容器初始化），并确定此容器的位置,方法有二：
  - ① 告诉监听器此容器的位置：context-param
  - ② 默认约定的位置：WEB-INF/applicationContext.xml

web.xml
```
  <listener>
  	<!-- 配置pring-web.jar提供的监听器，此监听器可以在服务器启动时初始化IOC容器
  		初始化ioc容器（applicationContext.xml），必须告诉监听器此容器的位置【context-param】 -->
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  
  <context-param>
  	<!-- 监听器的父类ContextLoader中有一个属性contextConfigLocation，
  		该属性值保存着容器配置文件applicationContext.xml的位置 -->
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:applicationContext.xml</param-value>
  </context-param>
```

## 5.2 拆分spring配置文件【Web项目】

> 根据什么拆分？

- ① 三层结构
  - i.UI(html/css/jsp、Servlet): applicationController.xml
  - ii.Service: applicationService.xml
  - iii.Dao: applicationDao.xml
  - iiii.公共【数据库】： applicationDB.xml
- ② 功能结构
  - 学生相关配置applicationContextStudent.xml
  - 班级相关配置applicationContextClass.xml

> 合并：如何将多个配置文件加载

- ① 在web.xml中【推荐】
```
在web.xml中

<param-value>
  		classpath:applicationContext.xml
  		classpath:applicationContext-*.xml
</param-value>

或
<param-value>
  		classpath:applicationContext.xml
  		classpath:applicationContext-Dao.xml
  		classpath:applicationContext-Service.xml
  		classpath:applicationContext-Controller.xml
</param-value>
```

- ② 只在web.xml中加载主配置文件，然后在主配置文件中加载其他配置文件
```
<!-- web.xml中 -->
<param-value>
  		classpath:applicationContext.xml
</param-value>


<!-- applicationContext.xml中 -->

<import resource="applicationContext-Controller.xml">
<import resource="applicationContext-Service.xml">
<import resource="applicationContext-Dao.xml">
或
<import resource="applicationContext-*.xml">
```


## 5.3 spring整合web项目

通过springIOC容器将studentService注入给Servlet，将studentDao注入给通过springIOC容器将studentService注入给Servlet

applicationContext-Controller.xml
```
<bean id="studentServlet" class="org.servlet.QueryStudentByIdServlet">
    <property name="studentService" ref="studentService"> </property>
</bean>
```
applicationContext-Service.xml
```
<bean id="studentService" class="org.service.impl.studentServiceImpl">
    <property name="studentDao" ref="studentDao"> </property>
</bean>
```
applicationContext-Dao.xml
```
<bean id="studentDao" class="org.service.impl.studentDaoImpl">
</bean>
```


> Servlet容器与Ioc容器及二者桥梁

- 启动时：Ioc在启动时会给Servlet的Service属性通过DI赋值，studentService不为空
  - bean的实例化、DI是保存在SpringIoc容器中
- 查询时：studentService为null
  - 每一次request是请求Servlet容器


- 桥梁：获取一份Ioc容器对象context。
  - Servlet初始化方法中，在初始化时，获取SpringIoc容器中的bean对象
  - 当前是在Servlet容器中，通过getBean获取Ioc容器中的Bean
  - 其中，Web项目获取spring上下文对象context建议用【WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());】

```
public void init() throws ServletException {
//    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext-Service.xml");
    ApplicationContext context = WebApplicationContextUtils.getWebApplicatinoContext(this.getAervletContext());
    studentService = (IStudentService)context.getBean("studentService");
}
```

# 6.Spring整合MyBatis

- 思路：
  - 因MyBatis最终是通过SqlSessionFactory来操作数据库的【SqlSessionFactory -> SqlSession -> StudentMapper -> CRUD】
  - 故Spring整合MyBatis其实就是将MyBatis的SqlSessionFactory交给Spring。


> ① jar

- mybatis-spring.jar
- spring-tx.jar
- spring-jdbc.jar
- spring-expression.jar
- spring-context-support.jar
- spring-core.jar
- spring-context.jar
- spring-beans.jar
- spring-aop.jar
- spring-web.jar
- commons-loggong.jar
- commons-dbcp.jar
- ojdbc.jar
- mybatis.jar
- log4j.jar
- commons-pool.jar


> ② 类-表

> ③ MyBatis配置文件conf.xml【整合时conf.xml可省】

> ④ 通过mapper.xml将类、表建立映射关系

> ⑤ 配置Spring配置文件【applicationContext.xml】

    - 通过Spring管理SqlSessionFactory，因此产生SqlSessionFactory所需要的数据库信息不再放入conf.xml，而是放入spring配置文件中。

> ⑥ 使用Spring-MyBatis整合产物开发程序

- 目标：通过Spring产生mybatis最终操作需要的动态mapper对象（StudentMapper对象）
- Spring产生动态mapper对象 有三种方法：
  - i.DAO层实现类 继承 SqlSessionDaoSupport类【SqlSessinDaoSupport类提供了一个属性SqlSession】
  - ii.省略掉第一种方式中的实现类【直接使用MyBatis提供的org.mybatis.spring.mapper.MapperFactoryBean（无需StudentDaoImpl.java）】
  - iii.批量配置实现类【无需每个mapper都配置一次】


- 注：第一种方式，SM整合时为了和mybatis更加紧密，方便识别，将IStudentDao.java改名为StudentMapper.java，并将其放入.mapper包中


applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 加载db.properties【为${ }】 -->
	<bean id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
		<property name="locations">
			<array>
				<value>classpath:db.properties</value>
			</array>
		</property>
	</bean>
	
	<!-- 配置数据库信息（替代mybatis配置文件conf.xml） -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
		<property name="driverClassName" value="${driver}"></property>
		<property name="url" value="${url}"></property>
		<property name="username" value="${username}"></property>
		<property name="password" value="${password}"></property>
		<property name="maxIdle" value="${maxIdle}"></property>
		<property name="maxActive" value="${maxActive}"></property>
	</bean>
	
	<!-- 在SpringIOC容器中创建MyBatis的核心类SqlSessionFactory -->
	<bean id="SqlSessionFactory"  class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<!-- 加载mybatis配置文件
		<property name="configLocation" value="classpath:conf.xml"></property>
		 -->
		<!-- 加载mapper.xml路径 -->
		<property name="mapperLocations" value="org/mapper/*.xml"></property>
	</bean>
	
	<bean id="studentService" class="org.service.impl.StudentServiceImpl">
		<property name="studentMapper" ref="studentMapper"></property>
	</bean>
	
	<!-- 第一种方式生成mapper对象
	<bean id="studentMapper" class="org.dao.impl.StudentDaoImpl">
	 	将Spring配置的sqlSessionFactory对象交给mapper(dao) 
		<property name="sqlSessionFactory" ref="SqlSessionFactory"></property>
	</bean>
	 -->
	 
	 <!-- 第二种方式生成mapper对象
	<bean id="studentMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
		<property name="mapperInterface" value="org.mapper.StudentMapper"></property>
		<property name="sqlSessionFactory" ref="SqlSessionFactory"></property>
	</bean>
	  -->
	  
	 <!-- 第三种方式生成mapper对象
	 		批量产生Mapper对象在SpringIOC中的id值默认就是首字母小写的接口名 -->
	<bean id="Mappers" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="sqlSessionFactoryBeanName" value="SqlSessionFactory"></property>
		<property name="basePackage" value="org.mapper"></property>
	</bean>

</beans>

```
