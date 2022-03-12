# <center>学习笔记，准备上班
## 每日复习
1. java基本知识
2. 数据结构（牛客题库刷题）
3. java8
4. javaweb
5. spring5
6. springmvc
7. springboot
8. spring security


## 复习java的一些基本知识
1.  **javac**  后面跟着的是java文件的文件名，例如 HelloWorld.java。 该命令用于将 java 源文件编译为 class 字节码文件，如：  **javac HelloWorld.java**。
运行javac命令后，如果成功编译没有错误的话，会出现一个 HelloWorld.class 的文件。
**java**  后面跟着的是java文件中的类名,例如 HelloWorld 就是类名，如: java HelloWorld。
2. throw：是java 的关键字，在方法体中使用，后面跟的是异常对象，只能跟一个异常对象。用于在方法体中抛出一个异常对象。
throws：是 java 的关键字，在方法的参数列表后使用，后面跟的是异常的类型。可以跟多个异常类型，使用逗号分隔。用于声明方法使用的时候可能会抛出的异常对象的类型。



## javaweb的相关知识
### xml文件
#### xml基础知识
1. 注释
2. 标签
	1. 不能包含空格
	2. 不能以数字开头
	3. 区分单标签和双标签
	4. xml属性必须被引号引起来
	5. xml文档必须且只能有一个根元素（根元素是没有父标签的顶级元素，而且没有父标签）
	6. CDAT语法内部的内容只是纯文本，不需要xml解析  
	<![CDATE[ 这里是不会解析的 ]]>

#### xml解析技术
1. 不管是html文件还是xml文件他们都是标记型文档，都可以使用w3c组织制定的dom技术来解析
2. **dom4j**
3. javaWeb是基于请求和响应开发的
	1. 请求是指客户端给服务器发送数据 request
	2. 响应是指服务器给客户端回传数据 response
	3. 请求和响应是成对出现的，有请求就有响应
#### tomcat和servlet
1. tomcat部署项目的两种方式
	1. webapp文件夹下
	2. conf/Catalina/localhost文件夹下创建xml文件`<Context path=""  docBase="" />`
path 表示工程的访问路径，docBase表示工程目录在哪
2. 什么是servlet
	1. servlet是javaee规范之一，规范就是接口
	2.   **Servlet是JavaWeb三大组件之一，三大组件分别是Servlet程序，Filter过滤器，Listener监听器**
	3. Servlet是运行在服务器上的一个java小程序，它可以接受和响应客户端的请求
3. 手动实现Servle程序
	1. 编写一个类去实现Servlet接口
	2. 实现service方法，处理请求并响应数据
	3. 到web.xml中配置servlet程序(servlet标签)和servlet访问地址(servlet-mapping)
4. servlet生命周期
	1. 执行Servlet构造器方法
	2. 执行init初始化方法
	**第12步只会在创建servlet时访问**
	3. 执行service方法
	**每次访问都回调用**
	4. 执行destroy销毁方法
	**在web工程停止是调用**
5. servlet和httpservlet的区别：**Servlet是一个接口，如何继承了这个接口，就需要实现接口中的所有的方法。而HttpServlet继承了这个接口，就代表已经实现了关于Servlet的所有接口，并且比较规范和实用。我们只需要通过HttpServlet中的doPost()和doGet()方法去实现业务逻辑既可以。**
6. servlet处理请求中文乱码的问题 
	request.setCharacterEncoding("UTF-8")
7. servlet继承关系以及service方法
	- 继承关系：HttpServlet->GenericServlet->Servlet
	-  Servlet中的核心方法: init() service() destroy()
	- Service方法： 当有请求过来时，service方法回自动响应（其实是tomcat容器调用的），在HttpServlet中我们会去分析请求的方式：到底是get，post，head还是delete等等，然后再决定调用哪个do开头的方法 。在HttpServlet中这些do方法默认都是405的实现风格，要我们子类去实现对应的方法，否则会报报405错误
8. Servlet生命周期
	 - servlet生命周期是tomcat容器帮我们维护的，tomcat只会创建一个servlet实例 
	 - 默认是第一次接受请求时，实例化，初始化，我们可以通过```<load-on-startup>```来设置servlet启动的先后顺序，数字越小，启动越靠前，最小数字0
	 - Servlet在容器中是单例的，线程不安全的，尽量不要再servlet中定义成员变量，如果不得不定义成员变量，那么不要根据成员变量的值做逻辑判断
9. HTTP协议
	- Http是无状态的
	- Http包含请求和响应
	- 请求：
		1. 请求行 
		包含当前请求的最基本信息（请求的方式，请求的URL，HTTP协议版本 ）
		2. 请求消息头 
		包含了很多客户端需要告诉服务器的信息，比如：我的浏览器型号，版本，我能接受的内容的类型 
		3. 请求主体
			- get方式，没有请求体，但是有一个queryString
			- post方式，有请求体，from data
			- json格式，有请求体，request payload
	-  响应
		1. 响应行
		包含三个信息，协议，响应状态码，响应状态
		2. 响应头
		服务器发送给浏览器的信息（内容的媒体类型，编码，内容的长度）
		3. 响应体
		响应的实际内容（比如请求add.html页面时，响应的内容就是```<html><head><body>```）
10. 会话
	1. http是无状态的这句话怎么理解
		- 如果有两个请求，服务器无法判断这两次请求是同一个客户端发来的还是不同的客户端发来的
		- 无状态带来的现实问题是：第一次请求是添加商品到购物车 ，第二次请求是结账，如果这两次请求服务器无法区分是同一个用户，那么就会导致混乱
		- 通过会话跟踪技术解决无状态的问题  
	2. 会话跟踪技术
		- 客户端第一次发请求给服务器，服务器获取session，获取不到，则创建新的，然后响应给客户端
		- 下次客户端给服务器发请求时，会把sessionID带给服务器，服务器就能获取到了，那么服务器就能判断这一次请求和上一次请求是同一个客户端，从而能够区分开客户端
		- 常用的api
		```request.getSession()``` 获取当前会话，如果没有则创建一个
		```request.getSession(true)``` 效果和不带参数相同
		```request.getSession(false)``` 获取当前会话，没有则返回null，不会创建新的
		```session.getId()```   获取sessionid
		```session.isNew()``` 判断当前session是否是新的
		```session.getMaxInactiveInterval``` session的非激活间隔时长，默认1800s
		```session.setMaxInactiveInterval```
	3. session保存作用域
		- session保存作用域和具体的某一个session对应的
11. 服务器内部转发以及客户端重定向
	- 服务器内部转发```request.getRequestDspather().forward(request,response)```
	一次请求响应的过程
	- 客户端重定向：```response.sendRedirect()```
	两次请求响应的过程 
12. Thymeleaf  视图模板技术
	- thymeleaf是帮助我们做视图渲染的一个技术  
13. 保存作用域
	- page（页面级别，几乎不用）
	- request（一次请求响应范围）
	 客户端重定向无法获取数据，服务器端转发可以获取数据
	- session（一次会话范围）
	客户端转发和服务器段转发都可以获取到数据，从session中获取数据
	- application（整个应用程序范围）
14. 路径问题
	- 相对路径
	- 绝对路径
15. Servlet中初始化方法有两个：init(); init(ServletConfig config)
16. MVC:
	- 视图层：用于做数据展示和用户交互的一个界面
	- 控制层：能够接受客户端的请求，具体的业务功能还是需要借助于模型组件来完成
	- 模型层：模型层分为很多钟，有比较简单的pojo/vo ，有业务模型组件，有数据访问层组件
	- 区分业务对象（BO）和数据访问对象（DAO）：
		- DAO中的方法都是单精度方法，一个方法只考虑一个操作，比如添加，那就是insert
		- BO中的方法属于业务方法，实际的业务是比较复杂的，因此业务方法的粒度是比较粗的
		- 注册这个功能属于业务功能，注册属于业务方法，这个业务方法中包含了多个DAO方法，也就是说注册这个业务功能需要通过多个DAO方法的组合调用
17. IOC
	+ 耦合/依赖
18. Filter
	+ filter也属于Servlet规范
	+ filter开发步骤：
		1.  新建类实现filter接口，然后实现其中的三个方法：init，doFilter，destroy 
		2. 配置Filter，可以用注解@WebFilter
		3. 可以实用通配符@WebFilter("*.do")表示拦截所有以.do结尾的所有请求
	+  过滤器链
		+ 如果采取的是注解的方式进行配置，那么过滤器链的拦截顺序是按照全类名的先后顺序排序的
		+ 如果采取的是xml的方式配置，那么按照配置的先后顺序进行排序
	+ 用过滤器设置编码
19. 事务管理
	+ ThreadLocal
		- get()  ,  set(obj)
		- ThreadLocal称之为本地线程，同一个线程中进行数据通信，可以通过set方法在当前线程上存储数据，通过get方法在当前线程上获取数
20. 监听器
	+  ServletContextListener 监听ServletContext对象的创建和销毁过程 
	+ HttpSessionListener
	+ ServletRequestListener
	+ ServletContextAttributeListenner监听ServletContext的保存作用域的改动
	+ HttpSessionAttributeListener 监听httpSession的保存作用域的改动
	+ ServletRequestAttributeListener监听ServletRequest的保存作用域的改动
