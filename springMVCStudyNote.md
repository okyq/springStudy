# <center> Spring学习笔记
## 1. HelloWord项目
### 1.1 配置web.xml
1. 默认方式配置xml
此配置下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为`<servlet-name>-servlet.xml`，例如下面的配置文件的名字为：`SpringMVC-servlet.xml`
```
<!-- 配置springMVC前端控制器-->  
  <servlet>  
 <servlet-name>SpringMVC</servlet-name>  
 <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
 </servlet> <servlet-mapping> <servlet-name>SpringMVC</servlet-name>  
 <url-pattern>/</url-pattern>  
<!-- 斜线不能匹配.jsp的请求，它有指定的servlet处理  
 /* 匹配所有请求，包括.jsp-->  
  </servlet-mapping>
```
2. 扩展配置方式（更优）
用==init-param==标签写明配置文件的名称和路径
```
<!-- 配置springMVC前端控制器-->  
<servlet>  
	 <servlet-name>SpringMVC</servlet-name>  
	 <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
<!-- 配置SpringMVC配置文件的位置和名称-->  
	 <init-param>  
		 <param-name>contextConfigLocation</param-name>  
		 <param-value>classpath:springMVC.xml</param-value>  
	 </init-param><!-- 把前端控制器dispatcherServlet初始化时间提前到服务器启动时-->  
	 <load-on-startup>1</load-on-startup>  
 </servlet> 
 <servlet-mapping> 
	 <servlet-name>SpringMVC</servlet-name>  
	 <url-pattern>/</url-pattern>  
<!-- 斜线不能匹配.jsp的请求，它有指定的servlet处理  
 /* 匹配所有请求，包括.jsp-->  
  </servlet-mapping>
```
### 1.2 HelloWord 项目
SpringMVC.xml
```
<!--扫描组件-->  
<context:component-scan base-package="com.yq.springMVC"/>  
	 <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">  
		 <property name="order" value="1"/>  
		 <property name="characterEncoding" value="utf-8"/>  
		 <property name="templateEngine">  
			 <bean class="org.thymeleaf.spring5.SpringTemplateEngine">  
				 <property name="templateResolver">  
					 <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">  
						 <property name="prefix" value="/WEB-INF/templates/"/>  
						 <property name="suffix" value=".html"/>  
						 <property name="templateMode" value="HTML5"/>  
						 <property name="characterEncoding" value="UTF-8"/>  
					 </bean>
				</property>
			</bean>
		</property>
	</bean>
</beans>
```
HelloController.class
```
@Controller  
public class HelloController {  
  
    @RequestMapping("/")  
    public String index(){  
        return "index";  
  }  
}
```
### 1.3 访问指定页面
HelloController.class
```
@Controller  
public class HelloController {  
  
    @RequestMapping("/")  
    public String index(){  
        return "index";  
  }  
  
    @RequestMapping("/target")  
    public String toTarget(){  
        return "target";  
  }  
}
```
index.html
```
<body>  
<h1>HelloWord</h1>  
<a th:href="@{/target}">跳转到Target页面</a>  
</body>  
</html>
```
## ==总结==：
浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispathcerServlet处理。前端控制器会读取SpringMVC核心配置文件，通过组件扫描找到控制器，将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法，处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应的页面。
## 2 RequestMapping注解
### 2.1 注解的功能
将请求和处理请求的控制器方法关联起来，建立映射关系。
SpringMVC接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求
### 2.2 注解的位置
1. 标识一个类：设置映射请求的请求路径的==初始==信息
2. 标识一个方法，设置映射请求请求路径的==具体==信息
### 2.3 RequestMapping注解的属性
1. value属性
String类型的数组，可以匹配多个值，请求映射可以处理多个请求，且==value属性必须设置==
	```
	@RequestMapping(
		value={"/test1","/test2"}
	)
	```
2. method属性
	+ 通过请求方式（post或get）匹配请求映射
	+ 它是一个RequestMethod类型的数组，可以匹配多种请求方式的请求
	+ 若当前请求满足value但是不满足method，则浏览器报错405： ``Request method 'POST' not supported``
	+ 默认匹配==所有==的请求
	```
	@RequestMapping(
		value={"/test1","/test2"}
		method={RequestMethod.GET,RequestMethod.POST}
	)
	```
3. 对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
	+ @GetMapping 处理get请求的映射
	+ @PostMapping，@PutMapping，@DeleteMapping同上
	+ 常见的请求的方式有get，post，put，delete，但是目前浏览器只支持get和post
4. params属性
	+ 通过请求参数匹配请求
	+ 必须同时满足所有值
	```
	@RequestMapping(
		value={"/test1","/test2"}
		method={RequestMethod.GET,RequestMethod.POST}
		params("username","password!=123")
	)
	```
5. headers属性（了解）
6. SpringMVC支持ant风格路径
	+ ? : 表示任意的单个字符
	+ \* : 表示0个或者多个字符
	+ \*\*: 表示任意的一层或多层目录
	+ 在使用\*\*时候，只能使用`/**/xxx`的方式
7. ==SpringMVC支持路径中的占位符== （==重点==）
	就是使用路径的方式传参
	+ 传统方式：/deleteUser?id=1
	+ rest方式：/deleteUser/1
	```
	@RequestMapping("/target/{username}/{id}")  
	public String testRest(@PathVariable("username")String username,@PathVariable("id")Integer id ){  
	    System.out.println("username:"+username+":::::"+"id"+id);  
	    return "target";  
	}
	```
## 3 SpringMVC获取请求参数
### 3.1 通过servletAPI获取(一般不用)
```
<a th:href="@{/test_para(username='admin',password='123')}">测试serlvetapi获取参数</a>

@RequestMapping("/test_para")  
public String testpara(HttpServletRequest request){  
    String username = request.getParameter("username");  
    String password = request.getParameter("password");  
    System.out.println("username"+username+",password = " + password);  
    return "target";  
}
```
### 3.2 通过控制器方法的形参来获取参数
形参的参数名和请求参数名一致，自动从将请求参数赋值给相应的形参
```
<form th:action="@{/test_many}" method="post">  
    用户名：<input type="text" name="username"/>  
    密码：<input type="text" name="password"/>  
    爱好<input type="checkbox" name="hobby" value="a">a  
    <input type="checkbox" name="hobby" value="b">b  
    <input type="checkbox" name="hobby" value="c">c  
    <input type="submit" value="提交">  
</form>

@RequestMapping("/test_many")  
public String many(String username,String password,String hobby){  
      System.out.println("username = " + username);  
	  System.out.println("password = " + password);  
	  System.out.println("hobby = " + hobby);  
	  return "target";  
}
注意：对于多个参数的hobby，既可以直接使用String，也可以使用String[]
```
### 3.3 @RequestParam注解
当形参的参数名和请求参数的参数名不一致的时候，可以使用`@RequestParam` 

+ 有三个属性：value，required，defaultValue
+ 默认必须传输数据
+  `@RequestParam(value="user_name",requirde=false)`此时user_name可以为null
+ 可以设置默认值 `@RequestParam(value="user_name",requirde=false , defaultValue="nihao")`
```
@RequestMapping("/test_many")  
public String many(
	@RequestParam("user_name") String username,
	String password,
	String hobby){  
    System.out.println("username = " + username);  
    System.out.println("password = " + password);  
    System.out.println("hobby = " + hobby);  
    return "target";  
}
```
### 3.4 @RequestHeader注解
是将请求头信息和控制器方法的形参创建映射关系，有三个属性：

+ value
+ required
+ defaultValue，用法同@RequestParam

```
@RequestMapping("/test_many")  
public String many(
	@RequestParam("user_name") String username,
	String password,
	String hobby,
	@RequestHeader("host") String host){  
    System.out.println(host)
    return "target";  
}
```

### 3.5 @CookieValue注解
是将cookie数据和控制器方法的形参创建映射关系，三个属性：
==第一次执行getSesison方法的时候，cookie会存在于相应报文，从此之后就会存在于请求报文
服务器创建并且相应到浏览器之后，以后 每一次浏览器向服务器发送请求都会携带cookie==
+ value
+ required
+ defaultValue，用法同@RequestParam

```
@RequestMapping("/test_many")  
public String many(
	@RequestParam("user_name") String username,
	String password,
	String hobby,
	@RequestHeader("host") String host
	@CookieValue("JSESSIONID" jsessionid)){  
    System.out.println(jsessionid)
    return "target";  
}
```

### 3.6 通过pojo获取请求参数
```
<form th:action="@{/testpojo}" method="post">  
  用户名：<input type="text" name="username"><br>  
  性别：<input type="text" name="sex"><br>  
 <input type="submit" value="提交，测试pojo">  
</form>
```
```
@RequestMapping("/testpojo")  
public String pojo(UserPojo userPojo){  
    System.out.println(userPojo.getUsername()+":::"+userPojo.getSex());  
    return "target";  
}
```
解决乱码：
```
<filter>  
	 <filter-name>filter</filter-name>  
	 <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  
		 <init-param> 
			 <param-name>encoding</param-name>  
			 <param-value>UTF-8</param-value>  
		 </init-param>
</filter>  
<filter-mapping>  
		 <filter-name>filter</filter-name>  
		 <url-pattern>/*</url-pattern>  
</filter-mapping>
```

## 4 域对象共享数据
### 4.1 使用servletAPI向request域对象共享数据
```
@RequestMapping("/testServletApi")  
public String api(HttpServletRequest request){  
    request.setAttribute("username","yuqian");  
    return "success";  
}
```
```
<p th:text="${username}"></p>
```

### 4.2 使用ModelAndView向request域对象共享数据
+ ModelAndView有Model和View两个功能
+ model主要用于向请求域共享数据
+ view主要用于设置视图，实现页面跳转
```
@RequestMapping("/testModerlAndView")  
public ModelAndView modelAndView(){  
     ModelAndView mad = new ModelAndView();  
     mad.addObject("username","yuqian");  
     mad.setViewName("success");  
     return mad;  
}
```

### 4.3 使用Model向request域对象共享数据
```
@RequestMapping("/testModel")  
public String model(Model model){  
    model.addAttribute("username","yuqian");  
    return "success";  
}
```
### 4.4 使用Map向request域对象共享数据
```
@RequestMapping("/testMap")  
public String map(Map<String,Object> map){  
    map.put("username","yuqian");  
    return "success";  
}
```
### 4.5 使用ModelMap向request域对象共享数据
```
@RequestMapping("/testModelMap")  
public String testModelMap(ModelMap modelMap){  
    modelMap.addAttribute("username","yuqian");  
    return "success";  
}
```
### 4.6 Model，ModelMap，Map之间的关系
本质上都是BindingAwareModelMap类型
```
public  interface model{}
public class ModelMap extends LinkedHashMap<String,Object>{}
public class ExtendedModelMap extends ModelMap implements Model{}
public class BindingAwareModelMap extends ExtendedModelMap{}
```

### 4.7 向session（浏览器开启和关闭）域共享数据
```
@RequestMapping("/testSession")  
public String testSession(HttpSession session){  
    session.setAttribute("username","yuqian");  
    return "success";  
}
```
```
<p th:text="${session.username}"></p>
```


### 4.8  向application（serlvetcontext整个应用）域共享数据
```
@RequestMapping("/testApplication")  
public String testApplication(HttpSession session){  
    ServletContext application = session.getServletContext();  
    application.setAttribute("username","yuqian");  
    return "success";  
}
```
```
<p th:text="${application.username}"></p>
```

# 5. SpringMVC视图
+ SpringMVC的视图是view接口，视图的作用是渲染数据，将模型model中的数据展示给用户
+ SpringMVC视图的种类有很多，默认有视图转发InternalResourceView和重定向视图RedirectView
+ 当工程引入jstl的依赖，转发视图会自动转换成jstlVIew
+ 若使用的视图技术为Thymeleaf，在springmvc的配置文件中配置了Teymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView
## 5.1 ThymeleafView
当控制器方法中设置的视图名称没有任何前缀时，此时的视图名称为被SpringMVC配置文件中配置的视图解析器解析，视图名称拼接视图前缀和视图后缀得到最终路径，会通过转发的方式实现跳转
```
@RequestMapping("/")  
public String index(){  
    return "index";  
}
```
## 5.2 转发视图
SpringMVC中默认的转发视图是InternalResourceView
SpringMVC中创建转发视图的情况：
当控制器方法中设置的视图名称以“forward：”为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是将前缀去掉，剩余部分作为最终路径通过转发的方式实现跳转
```
@RequestMapping("/testApplication")  
public String testApplication(HttpSession session){  
    ServletContext application = session.getServletContext();  
    application.setAttribute("username","application");  
    return "success";  
}  
  
@RequestMapping("/testForward")  
public String testForward(){  
    return "forward:/testApplication";  
}
```
此时地址栏显示
`http://localhost:8080/spring/testForward`
## 5.3 重定向视图
SpringMVC中默认的重定向视图是RedirectView
当控制器方法中所设置的视图名称以" redircet:" 为前缀时，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀去掉，剩余部分最为最终路径通过重定向的方式实现跳转
例如："redirect:/aaaa"
| 区别 | 转发foward|重定向redirect|
| :---------: | :----------: | :-----------: |
|根目录|包含项目访问地址|没有项目访问地址|
|地址栏|不会发生变化|会发生变化|
|哪里跳转|服务器端进行的跳转|浏览器端进行的跳转|
|请求域中的数据|不会丢失|会丢失|
|请求次数|1次|2次|
```
@RequestMapping("/testApplication")  
public String testApplication(HttpSession session){  
    ServletContext application = session.getServletContext();  
    application.setAttribute("username","application");  
    return "success";  
}
@RequestMapping("/testRedircet")  
public String testRedircet(){  
    return "redirect:/testApplication";  
}

```
`<a th:href="@{/testRedircet}">测试redirect</a>`
此时地址栏显示
`http://localhost:8080/spring/testApplication`

## 5.4 视图控制器view-controller
当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用view-controller标签进行表示
```
<mvc:view-controller path="/" view-name="index"></mvc:view-controller>
```
当设置了view-controller之后，控制器写的所有映射将全部失效
此时需要开启mvc注解驱动
```
<mvc:annotation-driven></mvc:annotation-driven>
```
# 6. RESTFul
## 6.1 RESTFul简介
REST：Representational State Transfer，表现层资源状态转移
当项目部署到服务器上之后，万物皆资源

1. 资源
资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象概念，所以它不仅仅能代表服务器系统中的一个文件，数据库的一张表等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词，一个资源可以由一个或多个URL来标识。URL既是资源的名称，也是资源再Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URL与其进行交互
2. 资源的表述
资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移。资源的表述可以有多种格式，例如HTML/xml/JSON/纯文本/图片/视频/音频等，资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式
3. 状态转移
在客户端和服务器端转移代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的
## 6.2 RESTFul的实现
+ 在HTTP协议里面，四个表示操作方式的动词：GET获取资源，POST新建资源，PUT更新资源，DElETE删除资源
+ REST风格提倡URL地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为URL地址的一部分，以保证整体风格的统一

|操作|传统方式|REST风格|
| :---------: | :------------: | :----------------: |
|查询操作|getUerByid?id=1|user/1 --> get method|
|保存操作|saveUser|user --> post method|
|删除操作|deleteUser?id=1|user/1 --> delete method|
|更新操作| updateUser| user  --> put method|
+ 一点插曲：get和post的区别
	+ 对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；
而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。
	    也就是说，GET只需要汽车跑一趟就把货送到了，而POST得跑两趟，第一趟，先去和服务器打个招呼“嗨，我等下要送一批货来，你们打开门迎接我”，然后再回头把货送过去。
	-   GET在浏览器回退时是无害的，而POST会再次提交请求。
	    
	-   GET产生的URL地址可以被Bookmark，而POST不可以。
	    
	-   GET请求会被浏览器主动cache，而POST不会，除非手动设置。
	    
	-   GET请求只能进行url编码，而POST支持多种编码方式。
	    
	-   GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
	    
	-   GET请求在URL中传送的参数是有长度限制的，而POST么有。
	    
	-   对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
	-   GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
	-   GET参数通过URL传递，POST放在Request body中。
### 6.2.1 get和post代码实现
```
<a th:href="@{/user}">查询所有用户数据</a>  
<a th:href="@{/user/1}">根据用户id查询用户数据</a>  
<form th:action="@{/user}" method="post">  
     用户名：<input type="text" name="username">  
     <input type="submit" value="submit">  
</form>
```
```
@RequestMapping(value = "/user",method = RequestMethod.GET)  
public String getUser(){  
    System.out.println("查询所有用户信息");  
    return "success";  
}  
  
@RequestMapping(value = "/user/{id}",method = RequestMethod.GET)  
public String getUserByID( @PathVariable("id")Integer id ){  
    System.out.println("根据id查询用户信息,传入的id为："+id);  
    return "success";  
}  
  
@RequestMapping(value = "/user" , method = RequestMethod.POST)  
public String AddUser(String username){  
    System.out.println("添加用户的用户名为："+username);  
    return "success";  
}
```
### 6.2.2  put和delete请求
