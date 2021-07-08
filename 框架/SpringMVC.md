[toc]

# 0.补充


**写URL时：**

**"${pageContext.request.contextPath}**/book/addBook"


**MVC执行流程**

![avatar](D:/其他/图片/study/MVC执行流程.png)

- / : 只匹配所有请求，不匹配jsp页面
- /* : 匹配所有请求，包括jsp页面

- ① （处理器映射器）
- ② （处理器适配器）
- ③ 视图解析器
- **①②在实际开发一般不需手动配置，只需开启注解驱动即可包含**

**RestFul风格**
- @RequestMapping(value="/add/{a}/{b}",method=RequestMethod.GET)
  - 参数@PathVariable()
- 好处：
  - 使路径变得更加简洁，一定程度上变得高效（缓存）和安全
  - 获得参数更加方便，框架会自动进行类型转换
  - 通过路径变量的类型可以约束访问参数


**通过SpringMVC来实现转发和重定向-无需视图解析器**
- return "WEB-INF/jsp/index.jsp";      //转发
- return "forward:/index.jsp";      //转发
- return "redirect:/index.jsp";     //重定向

**注意最好写上@RequestParam()**


![avatar](D:/其他/图片/study/SpringMVC的webxml.png)



**JSON 与 JavaScript 的关系 **
- JSON是JavaScript对象的字符串表示法，它使用文本表示一个JS对象的信息，本质是一个字符串
  - var obj = {a:'Hello',b:'world'};            //这是一个对象，注意键名也可以用引号包裹
  - var json = {"a":"Hello","b":"world"};       //这是一个JSON字符串，本质是一个字符串
- 互转
  - JSON->JavaScript：JSON.parse();
  - JavaScript->JSON：JSON.stringify();

**jackson的使用**

```
ObjectMapper mapper = new ObjectMapper();
String str = mapper.writeValueAsString(user/userList);   //将对象/集合转化为JSON格式的字符串


//对时间的处理
mapper.configure(SerializatinoFeature.WRITE_DATES_AS_TIMESTAMPS,false);     //关闭默认的时间戳方式
mapper.setDateFormat        //自定义日期格式    
```

![avatar](D:/其他/图片/study/Spring配置解决JSON乱码.png)


**FastJson的使用**

```
//Java对象转JSON对象
JSONObject job = (JSONObject)JSON.toJSON(user);

//JSON对象转Java对象
User user = JSON.parseObject(str,User.class);

//Java对象转JSON字符串
String str1 = JSON.toJSONString(user);

//Java对象数组转JSON字符串
String str1 = JSON.toJSONString(userList);
```

**拦截器与过滤器的区别**

- 拦截器是AOP思想的具体应用
- 拦截器是SpringMVC框架自己的，只有使用了SpringMVC框架的工程才能使用
  - 过滤器是servlet规范中的一部分，任何Java web工程都可以使用
- 拦截器只会拦截访问的控制器方法，若访问的是jsp/html/css/image/js是不会进行拦截的
  - 过滤器可以对所有要访问的资源进行拦截，只要在url-pattern中配置了/*即可








# 1.第一个程序

jsp-->Servlet（SpringMVC）-->jsp

### 1.1 jar

- commons-logging.jar
- spring-aop.jar
- spring-beans.jar
- spring-context.jar
- spring-expression.jar
- spring-core.jar
- spring-web.jar
- spring-webmvc.jar

### 1.2 配置文件

- 选中常用的命名空间：beans、aop、context、mvc

- 普通的servlet流程：请求被<url-pattern>拦截-->交给<servlet-class>中对应的servlet去处理；
  - 想用springmvc而不是普通的servlet，如何告诉程序，让springmvc介入程序？
  - 需要配置一个springmvc自带的servlet
  - 拦截所有请求，交给springmvc处理


web.xml【sts可自动生成】
```
  <servlet>
  	<servlet-name>springDispatcherServlet</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<!-- 配置spring配置文件位置 -->
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>classpath:springmvc.xml</param-value>
  	</init-param>
  	<!-- tomcat启动时让配置自动加载生效 -->
  	<load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>springDispatcherServlet</servlet-name>
  	<url-pattern>/</url-pattern>
  </servlet-mapping>
```
- 其中<url-pattern>/</url-pattern>
  - /：拦截一切请求，注意不是/*
  - /user:拦截以/user开头的请求
  - /user/abc.do ：只拦截该请求
  - .action : 只拦截.action结尾的请求 
- 使项目中同时兼容springMVC和Servlet
  - <url-pattern>.action</url-pattern>
  - 使.action结尾的请求交由springMVC处理【找@RequestMapper映射】
  - 使非.action结尾的请求交由servlet处理【找url-pattern（2.5）/@WebServlet（3.0）】
- 其中<init-param>配置springmvc.xml位置 
  - 若要省略，将springmvc.xml更名并放到默认路径【/WEB-INF/servlet-name的值-servlet.xml】


springmvc.xml
```
	<!-- 扫描有注解的包 -->
	<context:component-scan base-package="org.handler"></context:component-scan>
	
	<!-- 配置视图解析器 (InternalResourceViewResolver)-->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/views/"></property>
		<property name="suffix" value=".jsp"></property>		
	</bean>
```

### 1.3 映射及个属性


- 映射是去匹配@RequestMapper注解，可以和方法名、类名不一致
- 通过method指定请求方式【get、post、delete、put】，默认使用了请求转发的跳转方式；
- 设置name="xxxx"的情况【params={"name2=zs","age!=23"}】
  - name2：必须有name="name2"参数
  - ！name2：不能有name="name2"参数
  - age！=23：没有age 或 age值不等于23
- headers={"xx=xxxxxx"}，请求头


index.jsp
```
	<a href="handle/welcome">first springmvc - welcome【get】</a>
	
	<form action="handle/welcome" method="post">
		name:<input name="name"><br>
		age:<input name="age">
		<input type="submit" value="post">
	</form>
```

SpringMVCHandler.java【servlet/handler/controller/action】
```
@Controller
@RequestMapping(value="handle")
public class SpringMVCHandler {
	
	@RequestMapping(value="welcome",method = RequestMethod.POST,params = {"name=zs","age!=23"})
	public String welcome() {
		return "success";		//	view/success.jsp
	}

}
```

success.jsp
```
<body>
	welcome to springmvc
</body>
```

### 1.4 ant风格的请求路径

> '?'  单个字符
> 
> '*'  任意字符
> 
> '**'  任意目录

@RequestMapping(value="welcome3/**/test"),接受实例：a href="welcome3/abc/xyz/abcc/test"


- 基于ant，通过@PathVariable获取动态参数

index.jsp
```
	<a href="handle/welcome5/zs">welcome5</a>
```

SpringMVChandler.java
```
	@RequestMapping(value="welcome5/{name}")
	public String welcome(@PathVariable("name") String name) {
		System.out.println(name);
		return "success";		//	view/success.jsp
	}
```


# 2.REST风格

软件编程风格

- SpringMVC中：
  - GET ：查
  - POST ：增
  - DELETE ：删
  - PUT ：改
- 普通浏览器只支持get、post方式
- 其他请求方式如delete、put方式是通过过滤器新加入的支持

### 2.1 其他请求方式

① 增加过滤器【web.xml】
```
  <!-- 增加HiddenHttpMethodFilter过滤器：目的是给普通浏览器增加put/get请求方式 -->
  <filter>
  		<filter-name>HiddenHttpMethodFilter</filter-name>
  		<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  
  <filter-mapping>
  		<filter-name>HiddenHttpMethodFilter</filter-name>
  		<url-pattern>/*</url-pattern>
  </filter-mapping>
```

② 表单【index.jsp】
> i. 必须是post方式；

> ii. 通过隐藏域的value值，设置实际的请求方式 DELETE|PUT

```
	<form action="handle/testPost/1234" method="post">
		<input type="submit" value="增">
	</form>
	
	<form action="handle/testDelete/1234" method="post">
		<input type="hidden" name="_method" value="DELETE">
		<input type="submit" value="删">
	</form>
	
	<form action="handle/testPut/1234" method="post">
		<input type="hidden" name="_method" value="PUT">
		<input type="submit" value="改">
	</form>
	
	<form action="handle/testGet/1234" method="get">
		<input type="submit" value="查">
	</form>
```

③ 控制器【SpringMVCHandler.java】

> 通过method=RequestMethod.DELETE匹配具体的请求方式

> 当映射名value相同时，可以通过mathod来处理不同的请求

```
	@RequestMapping(value="testPost/{id}",method=RequestMethod.POST)
	public String testPost(@PathVariable("id") Integer id) {
		System.out.println("post:增"+id);
		//Service层实现真正的增
		return "success";		//	view/success.jsp
	}
	
	@RequestMapping(value="testGet/{id}",method=RequestMethod.GET)
	public String testGet(@PathVariable("id") Integer id) {
		System.out.println("get:查"+id);
		return "success";		//	view/success.jsp
	}
	
	@RequestMapping(value="testDelete/{id}",method=RequestMethod.DELETE)
	public String testDelete(@PathVariable("id") Integer id) {
		System.out.println("delete:删"+id);
		return "success";		//	view/success.jsp
	}
	
	@RequestMapping(value="testPut/{id}",method=RequestMethod.PUT)
	public String testPut(@PathVariable("id") Integer id) {
		System.out.println("put:改"+id);
		return "success";		//	view/success.jsp
	}
```

### 2.2 @RequestParam

接收前台传递的值的另一种方法：@RequestParam

index.jsp
```
	<form action="handle/testParam" method="get">
		<input name="uname" type="text">
		<input type="submit" value="查">
	</form>
```

SpringMVCHandler.java
```
	@RequestMapping(value="testParam")
	public String testParam(@RequestParam("uname") String name,@RequestParam(value="uage",required = false,defaultValue = "23") Integer age) {
//		String name = request.getParameter("uname");
		System.out.println(name);
		return "success";		//	view/success.jsp
	}
```

### 2.3 @RequestHeader

获取请求头信息【@RequestHeader】

```
	@RequestMapping(value="testRequestHeader")
	public String testRequestHeader(@RequestHeader("Accept-Language") String al) {
		System.out.println(al);
		return "success";		//	view/success.jsp
	}
```

### 2.4 @CookieValue

获取cookie值（JSESSIONID）【@CookieValue】

> 前置知识：服务端在接受客户端第一次请求时，会给该客户端分配一个session（该session包含一个sessionId），并且服务端会在第一次响应客户端时，将该sessionId复制给JSESSIONID并传递给客户端的cookie中

```
	@RequestMapping(value="testCookieValue")
	public String testCookieValue(@CookieValue("JSESSIONID") String jsessinoid) {
		System.out.println(jsessinoid);
		return "success";		//	view/success.jsp
	}
```

### 2.5 使用对象（实体类）接受请求参数

index.xml
```
	<form action="handle/testObjectProperties" method="post">
		id<input name="id" type="text">
		name<input name="name" type="text">
		homeAddress<input name="Address.homeAddress" type="text">
		schoolAddress<input name="Address.schoolAddress" type="text">
		<input type="submit" value="查">
	</form>
```

SpringMVCHandler.java
```
	@RequestMapping(value="testObjectProperties")
	public String testObjectProperties(Student student) {//student属性必须和form表单中的属性Name值一致（支持级联属性）
		System.out.println(student.getId()+","+student.getName()+","+student.getAddress().getHomeAddress()+","+student.getAddress().getSchoolAddress());
		return "success";		//	view/success.jsp
	}
```

### 2.6 使用原生态的ServletAPI

在SpringMVC中使用原生态的Servlet API【如HttpServletRequest】：直接将servlet-api中的类、接口等写在springMVC所映射的方法参数中即可


```
	@RequestMapping(value="testServletAPI")
	public String testServletAPI(HttpServletRequest request,HttpServletResponse response) {
//		request.getParameter("uname");
		System.out.println(request);
		return "success";		
	}
```

# 3.处理模型数据及使用注解

- 如果跳转时需要带数据：V（View）、M（Model），则可以使用以下方式：
  - ModelAndView、ModelMap、Map、Model--【数据放在request作用域】
  - @SessionAttributes、@ModelAttribute--【数据放入session域中】

### 3.1 ModelAndView

SpringMVCHandler.java
```
	@RequestMapping(value="testModelAndView")
	public ModelAndView testModelAndView() {
		ModelAndView mv = new ModelAndView("success"); //view: Views/success.jsp
		
		Student student = new Student();
		student.setId(2);
		student.setName("zs");
		
		mv.addObject("student",student); //相当于request.setAttribute("student",student)
		return mv;		
	}
```

success.jsp
```
	welcome to springmvc<br>
	${requestScope.student.id } - ${requestScope.student.name }<br>
```

### 3.2 ModelMap

```
	@RequestMapping(value="testModelMap")
	public String testModelMap(ModelMap mm) {
		Student student = new Student();
		student.setId(2);
		student.setName("zs");
		
		mm.put("student2", student);//request域
		
		return "success";		
	}
```


### 3.3 Map

```
	@RequestMapping(value="testMap")
	public String testMap(Map<String,Object> m) {
		Student student = new Student();
		student.setId(2);
		student.setName("zs");
		
		m.put("student3", student);//request域

		return "success";		
	}
```

### 3.4 Model

```
	@RequestMapping(value="testModel")
	public String testModel(Model model) {
		Student student = new Student();
		student.setId(2);
		student.setName("zs");
		
		model.addAttribute("student4", student);//request域

		return "success";		
	}
```

### 3.5 @SessionAttributes

SpringMVCHandler.java【类上的注解】
```
//@SessionAttributes(value="student3,student4")  //如果要在request中存放student4对象，则同时将该对象放入session域中
@SessionAttributes(types={Student.class,Address.class})  //如果要在request中存放student类型对象，则同时将该类型对象放入session域中
```

### 3.6 @ModelAttribute 

> 经常在【更新（查询+修改）】时使用

- 通过@ModelAttribute修饰的方法，会在每次请求前先执行；
- 并且该方法的参数map.put()可以将对象放入即将查询的参数中；
- 必须满足约定：map.put(k，v)中的k必须是即将查询的方法参数的首字母小写（即Xxx->xxx）;
- 若不一致，需要在即将查询的方法参数中加@ModelAttribute("xyz")
- @ModelAttribute在请求该类的各个方法前均被执行的设计基于“一个控制器只做一个功能”的思想，使用时需注意


```
	//查询
	@ModelAttribute		//在任何一次请求前，都会先执行@ModelAttribute修饰的方法
	public void queryStudentById(Map<String,Object> map) {
		//StudentService studentService = new StudentServiceImpl();
		//Student student = studetService.queryStudentByID(31);
		//模拟调用三层查询数据库的操作
		Student student = new Student();
		student.setId(31);
		student.setName("zs");
		student.setAge(23);
		map.put("student", student);	//约定：map的key是方法参数类型的“首字母小写”
	}
	
	//修改
	@RequestMapping(value="testModelAttribute")
	public String testModelAttribute(Student student) {
		student.setName(student.getName());  //将名字修改为ls
		System.out.println(student.getId()+","+student.getName()+","+student.getAge());
		return "success";		
	}
```

# 4.视图、视图解析器

- 视图的顶级接口：View【常见视图InternalResourceView】
- 视图解析器的顶级接口：ViewResolver【常见视图解析器InternalResourceViewResolver】


- InternalResourceView：将JSP或其他资源封装成一个视图，被视图解析器InternalResourceViewResolver默认使用；
- JstView：InternalResourceView的子类；
  - 可以解析jstl，实现国际化操作
  - 如果JSP中使用了JSTL的国际化标签，就需要使用该视图类。
  - SpringMVC解析jsp时，会默认使用InternalResourceView，若发现Jsp中包含了jstl语言相关内容，则自动转为JstView


### 4.1 国际化

国际化：针对不同国家、不同地区，进行不同的显示

- ① 创建资源文件
  - 基名_语言_地区.properties
  - 基名_语言.properties

> i18n.properties       //如果其他资源文件中没有设置一些属性的值，则在该文件中找
>
> i18n_en_US.properties
>
> i18n_zh_CN.properties

- ② 配置springmvc.xml，加载配置文件
  - i.将ResourceBundleMessageSource在程序加载时 加入springmvc【约定：springmvc在启动时，会自动查找一个id="messageSource"的bean，如果有则自动加载】
  - ii.ResourceBundleMessageSource会在springnvc响应程序时介入【解析国际化资源文件】

springmvc.xml
```
	<!-- 加载国际化资源文件 -->
	<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
		<property name="basename" value="i18n"></property>
	</bean>
```

- ③ 通过jstl使用国际化【jstl.jar、standar.jar】

success.jsp
```
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt"  prefix="fmt"%>

	<fmt:message key="resource.welcome"></fmt:message>
	<fmt:message key="resource.exit"></fmt:message>
```

### 4.2 InternalResourceViewResolver的其他功能

##### 4.2.1 <mvc:view-controller...>

> index.jsp->Controller(@QrequestMapper)->success.jsp

- 要用SpringMVC实现index.jsp->success.jsp：
  - <mvc:view-controller path="handler/testMvcViewController" view-name="success"/>
  - 以上配置会让所有的请求转入<mvc:...>中匹配映射地址，而会忽略掉@RequestMapper()
  - 若想让<mvc:...>和@RequestMapper()共存，则需要加入一个很重要的注解:
  - <mvc:annotation-driven></mvc:annotation-driven>


```
	<mvc:view-controller path="handler/testMvcViewController" view-name="success"/>
	
	<!-- 此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调 -->
	<mvc:annotation-driven></mvc:annotation-driven>
```

##### 4.2.2 指定跳转方式

forward：\redirect： 需要注意此种方式 不会被视图解析器加上前缀、后缀
```
return "forward:/views/success.jsp";
```

##### 4.2.3 处理静态资源

html、css、js、图片、视频...

- 在SpringMVC中，弱智既然访问静态资源：404，原因：
  - 之前将所有请求通过通配符"\"拦截，进而交给SpringMVC的入口DispatcherServlet去处理【找该请求映射对应的@RequestMapper】
- 不需要springmvc处理，则使用tomcat默认的Servlet去处理：
  - 有对应的请求被拦截，则交给对应的Servlet去处理
  - 如果没有对应的servlet，则直接访问
- 解决静态资源方案：
  - 如果有对应的@RequestMapper则交给spring处理，如果没有对应的@RequestMapper，则交给服务器tomcat默认的servlet去处理：
  - 只需要增加2个注解即可，一个是<mvc:annotation-driven>，另一如下:


springmvc.xml
```
	<!-- 该注解会让springmvc接收一个请求，并且该请求没有对应的@RequestMapper时，将该请求交给服务器默认的servlet去处理 -->
	<mvc:default-servlet-handler/>
```

##### 4.2.4 类型转换

- Spring自带一些常见的类型转换器
  - 如testDelete(@PathVariable("id") Integer id)，既可接受int类型数据id，也可接受String类型id
- 可以自定义类型转换器
  - ① 编写自定义类型转换器的类（实现Converter接口）

MyConverter.java
```
public class MyConverter implements Converter<String,Student>{

	@Override
	public Student convert(String source) {	
		//source接受前端传来的String:2-zs-23
		String[] studentStrArr = source.split("-");
		Student student = new Student();
		student.setId( Integer.parseInt( studentStrArr[0] ) );
		student.setName( studentStrArr[1] );
		student.setAge( Integer.parseInt( studentStrArr[2] ) );
		return student;
	}
}
```

  - ② 配置：将Converter加入到Springmvc容器中

springmvc.xml
```
	<!-- 1.将自定义转换器纳入SpringIOC容器 -->
	<bean id="myConverter" class="org.entity.converter.MyConverter"></bean>
	
	<!-- 2.将myConverter再纳入SpringMVC提供的转换器Bean -->
	<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
		<property name="converters">
			<set>
				<ref bean="myConverter"/>
			</set>
		</property>
	</bean>
	
	<!-- 3. 将conversionService注册到annotation-driven中 -->
	<!-- 此配置是SpringMVC的基础配置，很多功能都需要通过该注解来协调 -->
	<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>

```

  - ③ 测试转换器

index.jsp
```
	<form action="handle/testConverter" method="post">
		学生信息：<input name="studentInfo" type="text" />
		<input type="submit" value="转换">
	</form>
```

SpringMVCHandler.java
```
	@RequestMapping(value="testConverter")
	public String testConverter(@RequestParam("studentInfo") Student student) {
		System.out.println(student.getId()+","+student.getName()+","+student.getAge());
		return "success";		
	}
```

- @RequestParam("studentInfo")是触发转换器的桥梁


##### 4.2.5 数据格式化

- SpringMVC提供了很多方便我们数据格式化的注解
  - @DateTimeFormat(pattern="yyyy-MM-dd")  
  - @NumberFormat(pattern="###,#")

① 配置

springmvc.xml
> 其中id只能是conversionService

> FormattingConversionServiceFactoryBean：既可以实现格式化，又可以实现类型转换【即可将类型转换的配置中第二步也写在该bean下】

```
	<!-- 配置数据格式化注解所依赖的bean -->
	<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean"></bean>
```

② 通过注解使用

student.java
> //格式化：对象是前台，将前台传递来的数据“固定”为"yyy-MM-dd"

```
	@DateTimeFormat(pattern="yyyy-MM-dd")  
	private Date birthday;	// 2018-12-13
```


SpringMVCHandler.java
> 如果Student格式化出错，会将出错信息传入result中

> 其中Student和BindingResult之间不能有其他参数

> 如果要将控制台的错误消息传到jsp中显示，则可以将错误消息对象放入request域中，然后在hsp中从request获取

```
	@RequestMapping(value="testDateTimeFormat")
	public String testDateTimeFormat(Student student,BindingResult result,Map<String,Object> map) {
		System.out.println(student.getId()+","+student.getName()+","+student.getBirthday());
		if(result.getErrorCount()>0) {
			for(FieldError error: result.getFieldErrors() ) {
				System.out.println(error.getDefaultMessage());
				map.put("errors", result.getFieldErrors()); //将错误信息传入request的errors中 
			}
		}
		return "success";		
	}
```

success.jsp
```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core"  prefix="c"%>

	<c:forEach items="${requestScope.errors}" var="error">
		${error.getDefaultMessage()}<br>
	</c:forEach>
```

##### 4.2.6 数据校验

Hibernate Validator是JSR303的扩展

使用Hibernate Validator步骤

① jar（注意版本兼容问题）
- hibernate-validator.jar
- classmate.jar
- jboss-logging.jar
- validation-api.jar
- hibernate-validator-annotation-processor.jar

> 注：报错信息NoClassDefFoundError:jar包引起的错误

② 配置

```
	<mvc:annotation-driven></mvc:annotation-driven>
```
- 此时该配置的作用：
  - 要实现Hibernate Validator/JSR303（或其他）校验，必须实现springmvc提供的一个接口：ValidatorFactory
  - LocalValidatorFactoryBean是ValidatorFactory的一个实现类
  - 该注解会在springmvc日期中自动加载一个LocalValidatorFactoryBean类，因此可以直接实现数据校验

③ 使用注解

- 校验注解，如@Past
- 在校验的Controller中，给校验的对象前增加@Valid

```
	@Past	//当前时间以前
	private Date birthday;	// 2018-12-13
```

```
public String testDateTimeFormat(@Valid Student student,BindingResult result,Map<String,Object> map){...}
```

# 5.通过Ajax处理json数据

① jar
- jackson-annotations.jar
- jackson-core.jar
- jackson-databin.jar

② @ResponseBody修饰的方法，服务端会将该方法的返回值以json数组的形式传给result，返回给前台

index.jsp<head>中
```
		<script type="text/javascript" src="http://code.jquery.com/jquery-1.8.3.js"></script>
		<script type="text/javascript">
			$(document).ready(function(){
				$("#testJson").click(function(){
					//通过ajax请求springmvc
					$.post(
						"handle/testJson", //服务器地址	
						//{"name":"zs","age":23}
						function(result){	//服务器处理完毕后的回调函数，List<Studnt> students，加上@ResponseBody后，students的实质是一个json数组
							//eval(result);
							for(var i=0;i<result.length;i++){
								alert(result[i].id+"-"+result[i].name+"-"+result[i].age);
							}
						}
					);
					
				});
			});
		</script>
```

SpringMVCHandler.java
```
	@ResponseBody	//告诉SpringMVC，此时的返回值不是一个view页面，而是一个ajax调用的返回值
	@RequestMapping(value="testJson")
	public List<Student> testJson() {
		//模拟调用service的查询操作
		Student stu1 = new Student(1,"zs",23);
		Student stu2 = new Student(2,"ls",24);
		Student stu3 = new Student(3,"ww",25);
		List<Student> students = new ArrayList<>();
		students.add(stu1);
		students.add(stu2);
		students.add(stu3);
		return students;		
	}
```

# 6.文件上传

- 和Servlet方式的本质一样，都是通过commons-fileupload.jar和commons-io.jar
- SpringMVC可以简化文件上传的代码，但是必须满足条件：实现MultipartResolver接口
- SpringMVC提供了该接口的实现类：CommonsMultipartResolver
- 直接使用CommonsMultipartResolver实现上传

① jar
commons-fileupload.jar和commons-io.jar

② 配置CommonsMultipartResolver，将其加入SpringIOC容器

springmvc.xml
```
	<!-- 配置CommonsMultipartResolver，用于文件上传
		springioc容器在初始化时，会自动寻找一个Id="multipartResolver"的bean，并将其加入到SpringIOC容器
	 -->	
	<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="defaultEncoding" value="UTF-8"></property>
		<!-- 上传单个文件的最大值，单位Byte;值为-1表示无限制 -->
		<property name="maxUploadSize" value="104857600"></property>
	</bean>
```

③ 处理方法

index.jsp
```
	<form action="handle/testUpload" method="post" enctype="multipart/form-data">
		<input name="file" type="file" >
		描述：<input name="desc" type="text" >
		<input type="submit" value="上传">
	</form>
```
SpringMVChandler.java
```
	@RequestMapping(value="testUpload")
	public String testUpload(@RequestParam("desc") String desc, @RequestParam("file") MultipartFile file) throws IOException {
		
		System.out.println("文件描述信息："+desc);
		
		//jsp中上传的文件：file
		InputStream input = file.getInputStream(); //IO
		String fileName = file.getOriginalFilename();
		OutputStream out = new FileOutputStream("d:\\"+fileName);
		
		//将file上传到服务器中的某一个硬盘文件中
		byte[] bs = new byte[1024];
		int len = -1;
		while((len = input.read(bs) )!=-1 ) {
			out.write(bs,0,len);
		}
		out.close();
		input.close();
		
		System.out.println("上传成功！");
		
		return "success";		
	}
```

> ctrl+shift+r:自己编写的代码

> ctrl+shift+t:jar中的代码


# 7.拦截器

- 拦截器的原理和过滤器一样
- SpringMVC中，要想实现拦截器，必须实现一个接口HandlerInterceptor
- 可以有拦截器链


① 编写拦截器

- preHandle()：拦截请求
- postHandle()：拦截响应
- afterComletion()：当最终页面被渲染完毕后触发
  - 渲染：将jsp中的<% String name ...%>、css、js等组装完毕，最终显示出来

MyInterceptor.java
```
public class MyInterceptor implements HandlerInterceptor{
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("拦截请求");
		return true;	//true：拦截操作之后放行；false：拦截之后不放行，请求终止
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("拦截响应");
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		System.out.println("视图被渲染完毕");
	}
}
```

② 配置：将自己写的拦截器配置到SpringMVC中

springmvc.xml
```
	<!-- 将自己写的拦截器配置到SpringMVC中,默认拦截全部请求 -->
	<mvc:interceptors>
		<!-- 配置具体的拦截路径 -->
		<mvc:interceptor>
			<!-- 指定拦截的路径，基于ant风格 -->
			<mvc:mapping path="/**"/>
			<!-- 指定拦截的路径，基于ant风格 -->
			<mvc:exclude-mapping path="/handle/testUpload"/>
			<bean class="org.Interceptor.MyInterceptor"></bean>
		</mvc:interceptor>
		
		<!-- 配置具体的拦截路径 -->
		<mvc:interceptor>
			<!-- 指定拦截的路径，基于ant风格 -->
			<mvc:mapping path="/**"/>
			<!-- 指定拦截的路径，基于ant风格 -->
			<mvc:exclude-mapping path="/handle/testUpload"/>
			<bean class="org.Interceptor.MySecondInterceptor"></bean>
		</mvc:interceptor>
	</mvc:interceptors>
```

# 8.异常处理

SpringMVC中HandlerExceptionResolver接口的每个实现类都是异常的一种处理方式

### 8.1 HandlerExceptionResolver

HandlerExceptionResolver:主要提供了@ExceptionHandler注解，并通过该注解处理异常

1. @ExceptionHandler标识的方法参数必须是异常类型（Throwable或其子类），不能包含其他类型的参数
2. @ExceptionHandler默认只能捕获当前类中的异常方法，若发生异常和处理异常的方法不在同一类中：@ControllerAdvice
3. 如果一个方法用于处理异常：
- 且只处理当前类中的异常：@ExceptionHandler
- 且处理所有类中的异常：类前加@ControllerAdvice，方法前加@ExceptionHandler

4. 异常处理路径：最短优先【即若有ArithmeticException异常，则@ExceptionHandler({ArithmeticException.class})比...Exception.class优先】


SecondSpringMVCHandler.java
```
@Controller
@RequestMapping("second")
public class SecondSpringMVCHandler {
	@RequestMapping("testExceptionHandler")
	public String testExceptionHandler() {
//		try {}catch{}
		System.out.println(1/0);	//ArithmeticException
		
		return "success";
	}
	
	@RequestMapping("testExceptionHandler2")
	public String testExceptionHandler2() {
		int[] nums = new int [2];
		System.out.println(nums[2]);	//ArrayIndexOutOfBoundsException
		
		return "success";
	}
	
	//该方法可以捕获本类中抛出的ArithmeticException异常
	@ExceptionHandler({ArithmeticException.class,ArrayIndexOutOfBoundsException.class})
	public ModelAndView handlerArithmeticException(Exception e) {
		ModelAndView mv = new ModelAndView("error");
		mv.addObject("er",e);
		System.out.println(e+"=====");
		return mv;
	}
}
```
error.jsp
```
	${requestScope.er }
```

MyExceptionHandler.java
```
@ControllerAdvice
public class MyExceptionHandler {	//不是控制器，仅仅是用于处理异常的类
	@ExceptionHandler({Exception.class})
	public ModelAndView handlerArithmeticException2(Exception e) {
		ModelAndView mv = new ModelAndView("error");
		mv.addObject("er",e);
		System.out.println(e+"====="+"@ControllerAdvice中处理的异常方法，可以处理任何类中的异常");
		return mv;
	}
}
```

### 8.2 ResponseStatusExceptionResolver

自定义异常显示页面【@ResponseStatus】

myArrayIndexOutofBoundsException.java
```
@ResponseStatus(value=HttpStatus.FORBIDDEN,reason="数组越界222！！！")
public class MyArrayIndexOutofBoundsException extends Exception{	//自定义异常
}
```
- @ResponseStatus也可标志在方法前

```
	@RequestMapping("testMyException2")
	public String testMyException2(@RequestParam("i") Integer i) {
		if(i==3) {
			return  "redirect:testResponseStatus";
		}
		return "success";
	}
	
	@ResponseStatus(value=HttpStatus.CONFLICT,reason="测试！！！")
	@RequestMapping("testResponseStatus")
	public String testResponseStatus() {
		return "success";
	}
```

- 其他两种异常处理方式
  - DefaultHandlerExceptionResolver:SpringMVC在一些常见异常（300、404、500）的基础上，新增了一些异常（405）
  - SimpleMappingExceptionResolver:通过配置来实现异常的处理
 

# 9.SSM整合

- Spring - SpringMVC - MyBatis
  - Spring - MyBatis ：将MyBatis的SqlSessionFactory交给Spring
  - Spring - SpringMVC ：将Spring - SpringMVC各自配置一遍


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
- spring-webmvc.jar


> ②  类-表

> ③  通过mapper.xml将类、表建立映射关系【同时编写mapper.java】

> ④  配置Spring配置文件【applicationContext.xml】（web项目）：

- i. 在web配置文件中，引入spring配置【配置ContextLoaderListener（可自动生成）】

web.xml
```
  	<!-- 在web项目中，引入spring -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```

- ii. 在Spring配置文件中加载数据源【同时加载属性文件】

applicationContext.xml
```
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
```

属性文件db.properties
```
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/xs?serverTimezone=UTC
username=root
password=123456
maxIdle=1000
maxActive=500
```

> ⑤ Spring整合MyBatis：配置MyBatis的SqlSessionFactory【包含数据源和mapper.xml和配置】并交给Spring


applicationContext.xml【方式：扫描包】
```
	<!-- conf.xml:数据源、mapper.xml -->
	<!-- 在SpringIOC容器中配置MyBatis的核心类SqlSessionFactory -->
	<bean id="SqlSessionFactory"  class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
	<!-- 加载mapper.xml路径 -->
		<property name="mapperLocations" value="classpath:org/mapper/*.xml"></property>
	</bean>

	<!-- Spring整合MyBatis，将MyBatis的SqlSessionFactory交给Spring -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="sqlSessionFactoryBeanName" value="SqlSessionFactory"></property>
		<!-- 指定批量生产哪个包中的mapper对象 -->
		<property name="basePackage" value="org.mapper"></property>
		<!-- 上面basePackage所在的property的作用：
			将org.mapper包中所有的接口，产生与之对应的动态代理对象
			对象名就是首字母小写的接口名
		 -->
	</bean>
```

> ⑥ 继续整合SpringMVC:将SpringMVC加入项目即可

- i. 在web.xml中给项目加入springMVC支持【配置dispatcherServlet（可自动生成）】
```
	<!-- 整合SpringMVC -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:applicationContext-controller.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

- ii. 编写SpringMVC配置文件applicationContext-controller.xml
```
	<!-- 配置视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
	
	<!-- SpringMVC基础配置 -->
	<mvc:annotation-driven></mvc:annotation-driven>
```

> ⑦ 示例

- i. xxxService.java接口
- ii. xxxServiceImpl.java实现类
- iii. xxxController.java控制器
- iiii. 在applicationContext-controller.xml中扫描注解，将控制器所在包加入IOC容器

```
	<!-- 将控制器所在包加入IOC容器 -->
	<context:component-scan base-package="org.controller"></context:component-scan>
```

- iiiii. 编写xxxController.java控制器拿前台传递的数据

index.jsp
```
		<a href="controller/queryStudentByNo/5">查询5号学生</a>
```

- iiiiii. 调用底层【MyBatis已经实现了Dao的实现类，写Service层接口和实现类（同时依赖注入dao）】

StudentService.java
```
public interface StudentService {
	Student queryStudentByNo(int stuNo);
}
```
StudentServiceImpl.java【set注入需set方法】
```
public class StudentServiceImpl implements StudentService {
	//service依赖于dao（mapper）
	private StudentMapper studentMapper;
	
	public void setStudentMapper(StudentMapper studentMapper) {
		this.studentMapper = studentMapper;
	}

	@Override
	public Student queryStudentByNo(int stuNo) {
		return studentMapper.quaryStudentByStuno(stuNo);
	}
}
```
applicationContext.xml写依赖注入【给service注入dao】
```
	<!-- 依赖注入：给service注入dao -->
	<bean id="studentService" class="org.service.impl.StudentServiceImpl">
		<property name="studentMapper" ref="studentMapper"></property> 
		<!--value值为mapper对象首字母小写 -->
	</bean>
```
- iiiiiii. 调用底层【写控制器，同时通过注解方式注入Service（避免注入方式冲突）】

StudentController.java
```
@Controller
@RequestMapping("controller")
public class StudentController {
	//控制器依赖于Service
	@Autowired
	@Qualifier("studentService")
	private StudentService studentService ;
	
	public void setStudentService(StudentService studentService) {
		this.studentService = studentService;
	}

	@RequestMapping("queryStudentByNo/{stuno}")
	public String queryStudentByNo(@PathVariable("suono") Integer stuNo,Map<String,Object> map) {
		Student student = studentService.queryStudentByNo(stuNo);
		map.put("student", student);
		return "result" ;
	}
}
```


- iiiiiiii. 结果界面

result.jsp
```
		${requestScope.student.stuNo } -
		${requestScope.student.stuName } -
		${requestScope.student.stuAge }
```

