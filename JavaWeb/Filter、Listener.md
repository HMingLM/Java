[toc]

# Filter

### 1.概念

- 过滤器：当访问服务器的资源时，过滤器可以将请求拦截下来，完成一些特殊的功能；
- 过滤器的作用：一般用于完成通用的操作，如：登录验证、统一编码处理、敏感字符过滤等；

### 2.步骤

- i.定义一个类，实现接口Filter
- ii.复写方法
- iii.配置拦截路径【web.xml/注解】


```
import javax.servlet.*;

@WebFilter("/*")  //配置注解，访问所有资源之前，都会执行该过滤器
public class FilterDemo1 implements Filter{
    init...
    public voi doFilter(ServletRequest servletRequest,ServletResponse servletResposne,FilterChain filterChain) throws IOException,SerletException{
        System.out.print("filterDemo1被执行了...");
        
        filterChain.doFilter(servletRequest,servletResponse);  //放行
    }
    destroy...
}
```

### 3.细节 

##### 3.1 web.xml


```
<filter>
    <filter-name>demo1</filter-name>
    <filter-class>cn.itcast.web.filter.FilterDemo1</filter-class>
</filter>
<filter-mapping>
    <filter-name>demo1</filter-name>
    <!-- 拦截路径 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

##### 3.2 过滤器执行流程

- i.执行过滤器
- ii.执行放行后的资源
- iii.回来执行过滤器放行代码下边的代码

```
public voi doFilter(ServletRequest req, ServletResponse resp,FilterChain chain) throws IOException,SerletException{

    //对request对象请求信息增强
    System.out.print("filterDemo2执行了...");
    //放行
    filterChain.doFilter(servletRequest,servletResponse);  
    //对response对象的响应消息增强
    System.out.println("filterDemo2回来了...");
}
    
```

##### 3.3 过滤器生命周期方法

- i.init:在服务器启动后，会创建Filter对象，然后调用init方法；只执行一次，用于加载资源；
- ii.doFilter:每一次请求被拦截资源时执行；执行多次；
- iii.destroy:在服务器关闭后，Filter对象被销毁；如果服务器是正常关闭，则执行destroy方法；只执行一次，用于释放资源；

##### 3.4 过滤器配置详解

###### 3.4.1 拦截路径配置

- 具体资源路径： /index.jsp  只有访问index.jsp资源时，过滤器才会被执行
- 拦截目录：  /user/*   访问/user下的所有资源时，过滤器都会被执行
- 后缀名拦截：  *.jsp    访问所有后缀名为jsp资源时，过滤器都会被执行
- 拦截所有资源：  /*   访问所有资源时，过滤器都会被执行

###### 3.4.2 拦截方式配置

- 注解配置（设置dispatcherTypes属性）
  - REQUEST：浏览器直接请求资源（默认值）【拦截HTTP请求 get post】
  - FORWARD：转发请求资源
  - INCLUDE：包含访问资源
  - ERROR：错误跳转资源
  - ASYNC：异步访问资源

```
@WebFilter(value="/*",dispatcherTypes=DispatcherType.REQUEST) //浏览器直接请求index.jsp时，该过滤器才会被执行
@WebFilter(value="/*",dispatcherTypes= { DispatcherType.REQUEST,DispatcherType.FORWARD }) //浏览器直接请求或转发访问index.jsp时，该过滤器才会被执行

-------

<dispatcher>REQUEST</dispatcher>
<dispatcher>FORWARD</dispatcher>
```

- web.xml配置（设置<dispacher></dispacher>标签即可）

##### 3.5 过滤器链（配置多个过滤器）

###### 3.5.1 执行顺序

若有两个过滤器：过滤器1和过滤器2，顺序为

1. 过滤器1
2. 过滤器2
3. 资源执行
4. 过滤器2
5. 过滤器1

###### 3.5.2 过滤器先后顺序

- 注解配置：按照类名的字符串比较规则比较，值小的先执行；
- web.xml配置：<filter-mapping>谁定义在上边谁先执行；

### 4.案例

##### 4.1 登录验证

需求：

- 访问xx资源，验证其是否登录；
- 若已登录，则直接放行；
- 若未登录，则跳转到登录页面，提示“您尚未登录，请先登录”

```
public voi doFilter(ServletRequest req, ServletResponse resp,FilterChain chain) throws IOException,SerletException{

    //0.强制转换
    HttpServletRequest request = (HttpServletRequest) req;
    
    //i.获取资源请求路径
    String uri = request.getRequestURI();
    //ii.判断是否包含登录相关资源路径，要注意排除掉css/js/图片/验证码等资源
    if(uri.contains("/login.jsp") || uri.contains("/loginServlet") || uri.contains("/js/") ){
        //包含，用户就是想登录，放行
        chain.doFilter(req,resp);
    }
    else{
        //不包含，需要验证用户是否登录
        //iii.从获取的session中获取user
        Object user = request.getSession().getAttribute(s:"user");
        if(user!=null){
            //已登录，放行
            chain.doFilter(req,resp);
        }
        else{
            未登录，跳转至登录页面
            request.setAttribute(s:"login_msg",o:"您尚未登录，请登录");
            request.getRequestDispatcher(s:"/login.jsp").forward(request,resp);
        }
    }
}
```

##### 4.2 敏感词汇过滤

需求：

- 对xx案例录入的数据进行敏感词过滤
- 敏感词汇参考《敏感词汇.txt》
- 如果是敏感词汇，替换为***

###### 4.2.1 分析：

- 对request对象进行增强（增强获取参数相关方法）；
- 放行，传递代理对象；

> 增强对象功能的设计模式：装饰模式（较笨重）、代理模式

###### 4.2.2 代理模式（！.）

- 概念：代理对象代理真实对象（被代理的对象），达到增强真实对象功能的目的；
- 静态代理：有一个类文件描述代理模式
- 动态代理：在内存中形成代理类（实现步骤如下）
  - 代理对象和真实对象实现相同的接口
  - 代理对象 = Proxy.newProxyInstance();
  - 使用代理对象调用方法
  - 增强方法




# Listener

监听器

### 1. 监听对象的创建和销毁

监听对象 | 实现的类
---|---
request | ServletRequestListener
session | HttpSessionListener
application | ServletContextListener

每个监听器各自提供了2个方法：监听开始、监听结束的方法

### 2. 监听对象中属性的变更

监听对象 | 实现的类
---|---
request | ServletRequestAttributeListener
session | HttpSessionAttributeListener
application | ServletContextAttributeListener


### 3.事件监听机制

- 事件：一件事情
- 事件源：事件发生的地方
- 监听器：一个对象
- 注册监听：将事件、事件源、监听器绑定在一起；当事件源上发生某个事件后，执行监听器代码

### 4.ServletcontextListener

监听ServletContext对象的创建和销毁

##### 4.1 方法

- void contextDestroyed(ServletContextEvent sce) :ServletContext对象被销毁之前会调用该方法；
- void contextInitialized(ServletContextEvent sce) :ServletContext对象创建后会调用该方法；

##### 4.2 步骤

- 定义一个类，实现ServletContextListener接口
- 复写方法
- 配置【个别不需要配置，如HttpSessionBindingListener、HttpSessionActivationListener】
  - web.xml
  - 注解

web.xml
```
<listener>
    <listener-class>cn.itcast.web.listener.ContextLoaderlistener</listener-class>
</listener>

//指定初始化参数
<context-param>
```

注解
```
@WebListener
```



### session的四种状态


++监听session对象的绑定、解绑：HttpSessionBindingListener【不需要配置web.xml】、Serializable++
- 绑定  session.setAttribute("a",xxx) :将对象a绑定到session域中
- 解绑  session.removeAttribute("a") ：将对象a从session中解绑

> 序列化、反序列化需要实现Serializable接口

++监听session对象的钝化、活化：HttpSessionActivationListener【不需要配置web.xml】、Serializable++
- 钝化（序列化）  ：内存 -> 硬盘
  - 如何钝化：配置tomcat安装目录/conf/context.xml
- 活化（反序列化）：硬盘 -> 内存
  - 如何活化：session中获某一个对象时，内存中找不到会自动从硬盘中找（已钝化文件）（自动活化）

context.xml【通过配置实现钝化】
```
<Manager className="org.apache.catalina.session.PersistenManager" maxIdleSwap="5" >
    <Store className="org.apache.catalina.session.FileStore" directory="lq"/>
</Manager>
------maxIdleSwap：最大空闲时间，如果超过该时间，将会被钝化
------FileStore：通过该类具体实现钝化操作
------directory:绝对路径或相对路径（tomcat安装目录下work\Catalina\local\项目名\）
```

