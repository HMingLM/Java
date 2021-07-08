[toc]
## EL

- Expression Language ，可以替代JSP页面中的Java代码
- 传统的在JSP中用java代码显示数据的弊端：类型转换、需要处理null、代码参杂

<html>
${requestScope.student.address.schoolAddress}
${域对象.域对象中的属性.属性.属性.级联属性}
</html>

#### EL操作符

- **点操作符** . 使用方便
- **中括号操作符** [] 功能强大：
  - 可以包含特殊字符（. 、 -）可以访问数组，获取变量值
  - 如存在变量name，则可以${requestScope[name]}
  - 常量时：需加 "" 或 ''


#### EL运算

- 关系运算符
- 逻辑运算符
- Empty运算符：判断一个值 null、不存在 ->  true

#### EL表达式的隐式对象

自带的对象，不需要new就能使用

- 作用域访问对象（EL域对象）：
  - pageScope < requestScope < sessionScope < applicationScope
  - 如果不指定域对象，则默认会根据从小到大的顺序，依次取值
- 参数访问对象：
  - 获取表单数据（或 超链接 / 地址栏 中传的值 a.jsp?a=b&c=d )
  - ${param} 相当于 request.getParameter()
  - ${paramValues} 相当于 request.getParameterValues()
- JSP隐式对象：pageContext
  - 在jsp中可以通过EL隐式对象pageContext间接获取其他的jsp隐式对象
  - ${pageContext.将方法名get和()后将首字母小写}如${pageContext.getRequest()} --> ${pageContext.request}
  - 可以使用此方法 级联 获取方法： 如${pageContext.request.serverPort}


## JSTL

比EL更加强大

- 需要引入两个jar：jstl.jsr 、 standard.jar
- 引入tablib：<%@ tablib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
  - 其中prefix为前缀


> 三个核心标签库

#### 通用标签库

**<c:set>赋值**

- ① 在某个作用域之中（4个范围对象），给某个变量赋值

```
<c:set var="name" value="zhangsan" scope="request" />
-----相当于request.setAttribute("name","zhangsan");
${requestScope.name}
```

- ② 在某个作用域之中（4个范围对象），给某个对象的属性赋值
  - 此种写法不能指定scope属性

```
<c:set target="${requestScope.student}" property="sname" value="zxs" />
${requestScope.student.sname}
```

**<c:out>显示**

```
<c:out value="${requestScope.student}" default="zs-23" />
------显示value中的值【当value为空时，显示default中的默认值】

<c:out value='  <a href="https://www.baidu.com">百度</a>  '   escapeXml="false" />
------当escapeXml为true，最终显示<a href="https://www.baidu.com">百度</a>
------当escapeXml为false，最终显示百度链接
```


**<c:remove>删除属性**

```
<c:remove var="a" scope="request" />
```


#### 条件标签库(选择)

**单重选择**

```
<c:if test="..."> ... </c:if>
```

**多重选择**

```
<c:choose>
    <c:when test="..."> ... </c:when>
    <c:when test="..."> ... </c:when>
    <c:otherwise> ... </c:otherwise>
</c:choose>
```

> 注意test中书写

```
test="${10>2}"      --> true
test="${10>2} "     --> 非true
```


#### 迭代标签库（循环）

- ① for(int i=0;i<=5;i++)
```
<c:foreach begin="0" end="5" step="1" varStatus="status">
    ${status.index}
    test...
</c:foreach>
```

- ② for(String name:names)
```
<c:foreach var="name" items="${requestScope.names}" >
    ${name}
</c:foreach>

-----对象集合
<c:foreach var="student" items="${requestScope.students}" >
    ${student.sname}--${student.sno}
</c:foreach>
```


## Ajax

> 异步js和xml

如果网页中某一个地方需要修改，异步刷新可以实现只刷新该需要修改的地方，而页面中其他地方保持不变


#### js方式实现

XMLHttpRequest对象

**XMLHttpRequest对象的方法：**

- open(方法名【提交方式get|post】，服务器地址，true) ：与服务器建立连接
- send():
  - get：send(null)
  - post：send(参数值)
- setRequestHeader(header,value):
  - get：不需要设置此方法
  - post：需要设置此方法：
    - 如果请求元素中包含了文件上传：setRequestHeader("Content-Type","multipart/form-data");
    - 不包含文件上传：setRequestHeader("Content-Type","application/x-www-form-urlencoded");

**XMLHttpRequest对象的属性：**

- readyState：请求状态，只有状态为4代表请求完毕
- status：响应状态，只有200代表响应正常
- onreadystatechange：回调函数
- responseText：响应格式为String
- responseXML：响应格式为XML


xx.jsp
```
<script type="text/javascript">
    function register()
    {
        var mobile = document.getElementById("mobile").value;
        //通过ajax异步方式请求服务端
        xmlHttpRequest = new XMLHttpRequest();      //xmlHttpRequest前不加var：全局变量
        //设置xmlHttpRequest对象的回调函数
        xmlHttpRequest.onreadystatechange = callBack ;
        xmlHttpRequest.open("post","MobileServlet",true);
        //设置post方式的头信息
        xmlHttpRequest.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
        xmlHttpRequest.send("mobile="+mobile);  //k=v
    }
    
    //定义回调函数（接收服务端的返回值
    function callBack(){
        if(xmlHttpRequest.readyState ==4 && xmlHttpRequest.status ==200){
            var data = xmlHttpRequest.responseText;     //服务端返回值为String格式
            if(data == "true"){
                alert("该号码已存在，请更换！")；
            }else{
                alert("注册成功！");
            }
        }
    }
</script>


<body>
    手机：<input id="mobile" /><br/>
    <input type="button" value="注册" onclick="register()" />
</body>
```

MobileServlet.java
```
request.setCharacterEncoding("utf-8");
response.setCharacterEncoding("utf-8");
response.setContentType("text/html;charset=UTF-8");

String mobile = request.getParameter("mobile");
PrintWriter out = response.getWriter();
//假设此时数据库中只有一个号码18888888888
if("18888888888".equals(mobile)){
    out.writer("true");     //servlet以输出流的方式将信息返回给客户端
}else{
    out.write("false");
}
out.close();
```


#### jquery方式实现【推荐】

> 注意做对比

先导入jquery库

###### $.ajax
```
<script type="text/javascript" src="js/jquery-1.8.3.js"></script>
<script type="text/javascript">
    function register()
    {
        var $mobile = $("#mobile").val();
        $.ajax({
            url:"MobileServlet",
            请求方式:"post",
            data:"mobile="+$mobile,
            success:function(result,testStatus)
            {
                if(result == "true"){
                    alert("该号码已存在，请更换！")；
                }else{
                    alert("注册成功！")；
                }
            },
            error:function(xhr,errorMessage,e){
                alert("系统异常！")；
            }
        });
    }
</script>
```


###### $.get
```
$.get(
    //只写value，严格按照顺序
    "MobileServlet",        //服务器地址
    "mobile="+$mobile,          //请求数据
    function(result){
        if(result == "true"){
                    alert("该号码已存在，请更换！")；
                }else{
                    alert("注册成功！")；
                }
    };
    "text"          //预期返回值类型（String/xml/json）
);
```


###### $.post
```
$.post(
    //只写value，严格按照顺序
    "MobileServlet",        //服务器地址
    "mobile="+$mobile,          //请求数据
    function(result){
        if(result == "true"){
                    alert("该号码已存在，请更换！")；
                }else{
                    alert("注册成功！")；
                }
    };
    "text"          //预期返回值类型【"text"或"xml"或"json"）
);
```



###### $.getJSON

- 如果客户端是getJSON(),则需要以json格式返回数据

```
$.getJSON(
        "MobileServlet",        //服务器地址
        //"mobile="+$mobile,          //请求数据
        {"mobile":$mobile},
    function(result){       //msg:true/false
        if(result.msg == "true"){
            alert("该号码已存在，请更换！")；
        }else{
            alert("注册成功！")；
        }
    }
);
```

MobileServlet.java
```
String mobile = request.getParameter("mobile");
PrintWriter out = response.getWriter();
//假设此时数据库中只有一个号码18888888888
if("18888888888".equals(mobile)){
    //out.writer("true");     //servlet以输出流的方式将信息返回给客户端
    out.write( "{\"msg\":\"true\"}" );      //msg:true
}else{
    //out.write("false");
    out.write( "{\"msg\":\"false\"}" );      //msg:false
}
out.close();
```

- **Ajax处理json对象**【改进】

处理json对象先引入6个jar

> commons-beanutils-1.7.0.jar

> commons-collections-3.2.1.jar

> commons-lang.jar

> commons-logging-1.1.3.jar

> ezmorph-1.0.6.jar

> json-lib-2.4-jdk15.jar


```
function testJson()
{
    $getJSON(
           "JsonServlet",        //服务器地址
            {"name":"zs", "age":"24"},          //请求数据
            
---------json中只有一个对象的情况---------
            
        function(result){
            //js需要通过eval()函数将返回值转为一个js能够识别的json对象
            var jsonStudent = eval(result.stu1);
            alert(jsonStudent.name +"---"+ jsonStudent.age);
        } 
        
---------json中有多个对象的情况---------
        function(result){
            //result：{"stu1":stu1, "stu2":stu2, "stu3":stu3}
            //js需要通过eval()函数将返回值转为一个js能够识别的json对象
            var json = eval(result);
            $.each( json, function(i,element){
                alert(this.name +"---"+ jsonStudent.age);
            });
        }
        
        
        
    );
}



<body>
    <input type="button" value="测试json" onclick="testJson()" />
</body>
```


JsonServlet.java
```
request.setCharacterEncoding("utf-8");
response.setCharacterEncoding("utf-8");
response.setContentType("text/html;charset=UTF-8");
PrintWriter out = response.getWriter();

//测试前端传来的数据
String name = request.getParameter("name");
String age = request.getParameter("age");

Student stu1 = new Student();
stu1.setAge(33);
stu1.setName("zs");

Student stu2 = new Student();
stu2.setAge(44);
stu2.setName("ls");

Student stu3 = new Student();
stu3.setAge(55);
stu3.setName("ww");

JSONObject json = new JSONObject();
json.put("stu1",stu1);
json.put("stu2",stu2);
json.put("stu3",stu3);

out.print(json);    //返回json对象 {"stu1":stu1, "stu2":stu2, "stu3":stu3}
out.close();
```



###### $("#xxx").load

- 直接将服务端的返回值加载到$(xxx)所选择的元素中

```
<script type="text/javascript" src="js/jquery-1.8.3.js"></script>
<script type="text/javascript">
    function register()
    {
        var $mobile = $("#mobile").val();
        $("#tip").load(
            "MobileServlet",        //服务器地址
            "mobile="+$mobile,          //请求数据
        );
    }

<body>
    手机：<input id="mobile" /><br/>
    <input type="button" value="注册" onclick="register()" />
    <span id="tip">  </span>
</body>
```

MobileServlet.java
```
String mobile = request.getParameter("mobile");
PrintWriter out = response.getWriter();
//假设此时数据库中只有一个号码18888888888
if("18888888888".equals(mobile)){
    //out.writer("true");     //servlet以输出流的方式将信息返回给客户端
    out.write("该号码已存在，请更换！");
}else{
    //out.write("false");
    out.write("注册成功！");
}
out.close();
```