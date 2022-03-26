# 一	基础入门

## 1.1 自动装配

+ 自动配好Tomcat

	- 引入Tomcat依赖。
	- 配置Tomcat

+ 自动配好SpringMVC

	- 引入SpringMVC全套组件
	- 自动配好SpringMVC常用组件（功能）
	- 自动配好Web常见功能，如：字符编码问题
	- SpringBoot帮我们配置好了所有web开发的常见场景

+ 默认的包结构

	- **主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来**
	- 无需以前的包扫描配置
	- 想要改变扫描路径，@SpringBootApplication(scanBasePackages=**"com.atguigu"**)
	- 或者@ComponentScan 指定扫描路径
```java
    @SpringBootApplication
    等同于
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan("com.yq.springboot")
```

+ 各种配置拥有默认值

  + 默认配置最终都是映射到某个类上，如：MultipartProperties
  + 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

+ 按需加载所有自动配置项
  + 非常多的starter
  + 引入了哪些场景这个场景的自动配置才会开启
  + SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面



# 二    容器功能

## 2.1 组件添加

### 2.1.1. @Configuration

功能：告诉SpringBoot这是一个**配置类（配置文件）**，就相当于之前在spring配置文件中写的bean。

Full模式Full(**proxyBeanMethods = true**)与Lite模式Lite(**proxyBeanMethods = false**)

+ 配置 类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断 
+ 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

使用示例

+ 配置类里面使用**@Bean**标注在方法上给容器注册组件，默认也是单实例的

+ 配置类本身也是组件
+ 默认是ture

```java
@Configuration
public class HelloConfig {
    @Bean("user")
    public User getUser(){
        return new User("yuqian", "yuqian");
    }
}
```

```java
@RestController
public class HelloController {
    @Autowired
    private User user;

    @RequestMapping("/getUser")
    public String getUser(){
        return user.toString();
    }
}
```

### 2.1.2 @Bean、@Component、@Controller、@Service、@Repository @ComponentScan、

这几个注解和spring中的一样

### 2.1.3 @Import

`@Import({User.class, DBHelper.class})`
给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名

(比如有时候组件在容器之外，此时既可以使用componentScan，也可以使用import)

```java
@RestController
@EnableAutoConfiguration
@Import({com.yq.controller.HelloController.class,com.yq.config.HelloConfig.class})
public class MyApplication {}
```

### 2.1.4 @Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

`@ConditionalOnMissingBean(name = "tom")`

`@ConditionalOnBean(name = "tom")`



## 2.2 原生配置文件导入

### 2.2.1 @ImportResource

`@ImportResource("classpath:beans.xml")`

就是导入之前spring中的配置文件



## 2.3 配置绑定

### 2.3.1 @ConfigurationProperties

```java
@Component
@ConfigurationProperties(prefix = "myuser")
public class User {
    private String username;
    private String password;
}
```

application.properties 里面写上：

```properties
myuser.username = "test"
myuser.password = "test"
```



使用了这个之后，就相当于把User添加到了容器中，并且用properties中的值为User赋值

在controller中直接调用

```java
@RestController
public class HelloController {
    @Autowired
    private User user;

    @RequestMapping("/getUser")
    public String getUser(){
        return user.toString();
    }
}
```

浏览器打印

`User{username='"test"', password='"test"'}`

### 2.3.2 @EnableConfigurationProperties

**@EnableConfigurationProperties 注解的作用是:让使用了 @ConfigurationProperties 注解的类生效,并且将该类注入到 IOC 容器中,交由 IOC 容器进行管理**

说白话：@ConfigurationProperties的作用只是将类和properties绑定在一起，没有在ioc容器中，所以上面User类中添加了@Component注解将其注入到ioc容器中。@EnableConfigurationProperties 就可以让没有生效的@ConfigurationProperties生效（为什么没有生效？因为没有注入ioc容器）

所以可以这样

```java
@ConfigurationProperties(prefix = "myuser")
public class User {
    private String username;
    private String password;}
这样是不会生效的，因为没有注入ioc
```

但是

```java
@RestController
@EnableAutoConfiguration
@ComponentScan("com.yq")
@EnableConfigurationProperties(User.class)
public class MyApplication {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
这样就会生效了，因为使用了@EnableConfigurationProperties(User.class)
```



# 三 开发技巧

## 3.1 Lombok

简化javabean开发

1. 引入依赖

```xml
<dependency>
 <groupId>org.projectlombok</groupId>
 <artifactId>lombok</artifactId>
 </dependency>

```

2. idea添加Lombok插件

2021的idea好像不支持...

## 3.2 dev-tools

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-devtools</artifactId>
 <optional>true</optional>
 </dependency>

```

项目或者页面修改以后：Ctrl+F9；

## 3.3、Spring Initailizr（项目初始化向导）

0.0

# 四 配置文件

## 4.1文件类型

### 4.1.1 properties

同以前的properties kv的形式

### 4.1.2 yaml

YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。

非常适合用来做以**数据为中心的配置文件**



**基本语法：**

+ key: value；kv之间有空格
+ 大小写敏感 
+ 使用缩进表示层级关系 
+ 缩进不允许使用tab，只允许空格 
+ 缩进的空格数不重要，只要相同层级的元素左对齐即可
+  '#'表示注释 
+ 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义



**数据类型**

+ 字面量：单个的、不可再分的值。date、boolean、string、number、null

```yaml
k: v   k:(空格) v
```

+ 对象 ：键值对的集合。map、hash、set、object

```yaml
行内写法： k: {k1:v1,k2:v2,k3:v3}
#或
k:
 k1: v1
 k2: v2
 k3: v3
```

+ 数组：一组按次序排列的值。array、list、queue

 ```yaml
 行内写法： k: [v1,v2,v3]
 #或者
 k:
  - v1
  - v2
  - v3
 
 ```

```java
@Data
public class Person {

 private String userName;
 private Boolean boss;
 private Date birth;
 private Integer age;
 private Pet pet;
 private String[] interests;
 private List<String> animal;
 private Map<String, Object> score;
 private Set<Double> salarys;
 private Map<String, List<Pet>> allPets;
}
@Data
public class Pet {
 private String name;
 private Double weight;
}

```

aplication.yaml

```yaml
person:
 userName: zhangsan
 boss: false
 birth: 2019/12/12 20:12:33
 age: 18
 pet:
 name: tomcat
 weight: 23.4
 interests: [篮球,游泳]
 animal:
 - jerry
 - mario
 score:
 english:
 first: 30
 second: 40
 third: 50
 math: [131,140,148]
 chinese: {first: 128,second: 136}
 salarys: [3999,4999.98,5999.99]
 allPets:
 sick:
 - {name: tom}
 - {name: jerry,weight: 47}
 health: [{name: mario,weight: 47}]
```

## 4.2 配置提示

```XML
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-configuration-processor</artifactId>
     <optional>true</optional>
</dependency>

<build>
     <plugins>
         <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
         <configuration>
             <excludes>
                 <exclude>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-configuration-processor</artifactId>
                 </exclude>
             </excludes>
         </configuration>
         </plugin>
     </plugins>
 </build>

```



# 五 web开发

## 5.1 简单功能分析

### 5.1.1静态资源访问

#### 1 静态资源目录

只要静态资源放在类路径下： called /static (or /public or /resources or /META-INF/resources 访问 ： 当前项目根路径/ + 静态资源名

原理： 静态映射/**。 

请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

#### 2 静态资源访问前缀

默认无前缀

```yaml
spring:
 mvc:
 static-path-pattern: /res/**

```

自定义静态资源访问前缀

localhost:8080/res/test.jpg

如果设定了自定义static文件夹

```yaml
spring:
	web:
    	resources:
      		static-locations: classpath:/ha/
```

则springboot会在/ha这个文件夹下找文件



#### 3 webjar

自动映射

网址：webjars.org

```xml
<dependency>
 <groupId>org.webjars</groupId>
 <artifactId>jquery</artifactId>
 <version>3.5.1</version>
 </dependency>

```

访问静态资源的地址 : localhost:8080/webjars**/jquery/3.5.1/jquery.js**

### 5.1.2 欢迎页支持

静态资源路径下 index.html

自动匹配

### 5.1.3 自定义Favicon

把facicon.ico放在静态资源目录下即可

### 5.1.4 静态配置原理

+ SpringBoot启动默认加载 xxxAutoConfiguration类（自动配置类）
+ SpringMVC功能的自动配置类 WebMvcAutoConfiguration，生效
+ （插入一个知识：如果一个类只有有参构造，则所有参数的值都会从容器中找）

这部分知识后面补充把，现在不想看了

## 5.2 请求参数处理

### 5.2.0请求映射

#### 1. rest使用与原理

+ @xxxMapping；
+ Rest风格支持（使用HTTP请求方式动词来表示对资源的操作）
  + 以前：/getUser 获取用户 /deleteUser 删除用户 /editUser 修改用户 /saveUser 保存用户
  + 现在： /user GET-获取用户 DELETE-删除用户 PUT-修改用户 POST-保存用户
  + 核心Filter；HiddenHttpMethodFilter
    + 用法： 表单method=post，隐藏域 _method=put
    + SpringBoot中手动开启



```yaml
spring:
 	mvc:
		 hiddenmethod:
 			filter:
 				enabled: true #开启页面表单的Rest功能

```

Rest原理（表单提交要使用REST的时候）

+  表单提交会带上_method=PUT 

+ 请求过来被HiddenHttpMethodFilter拦截 

  + 请求是否正常，并且是POST 

  + 获取到_method的值。

  +  兼容以下请求；PUT.DELETE.PATCH 

  + 原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。

  +  过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。

#### 2. 请求映射原理

SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet-》**doDispatch（）**

  ....后面再看



+ SpringBoot自动配置了默认 的 RequestMappingHandlerMapping 
+ 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。
  +  如果有就找到这个请求对应的handler 
  + 如果没有就是下一个 HandlerMapping

### 5.2.1 普通参数与基本注解

#### 1 注解

+ **@PathVariable**、

  + rest风格

   ```java
    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)  
    public String getUserByID( @PathVariable("id")Integer id ){  
        System.out.println("根据id查询用户信息,传入的id为："+id);  
        return "success";  
    }  
   ```

    

+ **@RequestHeader**、

  + 是将请求头信息和控制器方法的形参创建映射关系

+ **@ModelAttribute**、


+ **@RequestParam**、
  + 当形参的参数名和请求参数的参数名不一致的时候，可以使用`@RequestParam` 
  
+ **@MatrixVariable**、


  + SpringBoot默认禁用了矩阵变量


  + 不使用@EnableWebMvc注解，使用@**Configuration+WebMvcConfigurer**自定义规则
    
     ```JAVA
      @Configuration
      public class HelloConfig {
      
          @Bean
          public WebMvcConfigurer webMvcConfigurer(){
              return  new WebMvcConfigurer() {
                  @Override
                  public void configurePathMatch(PathMatchConfigurer configurer) {
                      UrlPathHelper urlPathHelper = new UrlPathHelper();
                      urlPathHelper.setRemoveSemicolonContent(false);
                      configurer.setUrlPathHelper(urlPathHelper);
                  }
              };
          }
      }
     ```
    
     ```java
          @ResponseBody
          @RequestMapping("/matrix/{path}")
          public Map getMatrix(@MatrixVariable("name") String name,
                               @MatrixVariable("name2") String name2){
              Map<String,String > matrixMap = new HashMap<>();
              matrixMap.put("name" , name);
              matrixMap.put("name2",name2);
              return matrixMap;
          }
      }
     ```
    
    ```html
      /matrix/user;name=yuqian;name2=afe
    ```

  + 对于路径的处理，都是使用了UrlPathHelper进行解析，

  + removeSemicolonContent（移除分号类容）

  + 矩阵变量绑定在路径变量中

  +   ```html
      <a href:"/cars/sell;low=34;brand=byd,audi,yd"/>
      ```

      ```java
      @GetMapping("/cars/{path}")
      public Map cars(@MatrixVariable("low") Integer low,
                     	@MatrixVariable("brand") List<String> brand)
      ```

      + 不同变量要用引号分开，要与路径看作为一个整体，例如

      ```html
      <a href:"/cars/sell;low=34;brand=byd,audi,yd/aaa/bbb"/>
      此时请求的路径为：/cars/sell/aaa/bbb
      ```

+ **@CookieValue**、
  + 是将cookie数据和控制器方法的形参创建映射关系，

+ **@RequestBody**
  + @RequestBody可以获取请求体，需要在控制器方法中设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

还有一些springmvc中的api之类

+ **@RequestAttribute**

  + 获取请求域中的数据

  ```java
  public Map success(@RequestAttribute("msg") String msg)
  ```


#### 2 servletApi

+ WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、 Locale、TimeZone、ZoneId

#### 3 复杂参数

+ **Map、Model（map、model里面的数据会被放在request的请求域 request.setAttribute）**、Errors/BindingResult、**RedirectAttributes（ 重定向携带数 据**）、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

#### 4 自定义对象参数

springmvc里面的

### 5.2.2 POJO封装过程

### 5.2.3 参数处理原理

+ HandlerMapping中找到能处理请求的Handler（Controller.method()）
+  为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter** 
+ 适配器执行目标方法并确定方法参数的每一个值



## 5.3 响应数据与内容协商

### 5.3.1 响应json

jackson.jar + @ResponseBody

引入webstart依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

会自动引入json场景

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.6.4</version>
      <scope>compile</scope>
</dependency>
```

给前端自动返回json数据



### 5.3.2 返回xml

```xml
加入xml依赖
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

```java
@Component
@ConfigurationProperties(prefix = "userxml") //这里竟然不能使用大写或者驼峰
public class UserXml {
    private String username;
    private String password;
    
```

```yaml
UserXml:
  username: yuqian
  password: yqpswd
```

此外也可以自定义xml节点

```java

@JacksonXmlRootElement(localName = "response")
public class UserXmlVO {
 
    @JacksonXmlProperty(localName = "user_name")
    private String name;
 
    @JacksonXmlElementWrapper(useWrapping = false)
    @JacksonXmlProperty(localName = "order_info")
    private List<OrderInfoVO> orderList;
 
    // get set 略
    
@JacksonXmlRootElement： 用在类上，用来自定义根节点名称；

@JacksonXmlProperty： 用在属性上，用来自定义子节点名称；

@JacksonXmlElementWrapper： 用在属性上，可以用来嵌套包装一层父节点，或者禁用此属性参与 XML 转换。

```



## 5.4 视图解析与模板引擎

### 5.4.1 视图解析

### 5.4.2 模板引擎-Thymeleaf

#### 1. 简介 

现代化、服务端Java模板引擎

#### 2. 基本语法

| 表达式名字 |  语法   |                用途                |
| :--------: | :-----: | :--------------------------------: |
|  变量取值  | ${...}  |  获取请求域、session域、对象等值   |
|  选择变量  | *{...}  |          获取上下文对象值          |
|    消息    | \#{...} |           获取国际化等值           |
|    链接    | @{...}  |              生成链接              |
| 片段表达式 | ~{...}  | jsp:include 作用，引入公共页面片段 |

**字面量**：

+ 文本值: 'one text' , 'Another one!' ,…数字: 0 , 34 , 3.0 , 12.3 ,…布尔值: true , false

+ 空值: null

+ 变量： one，two，.... 变量不能有空格

**文本操作**

+ 字符串拼接: +
+ 变量替换: |The name is ${name}|

**数学运算**

+ 运算符: + , - , * , / , %

**布尔运算**

+ 运算符: and , or 
+ 一元运算: ! , not

**比较运算**

比较: > , < , >= , <= ( gt , lt , ge , le )等式: == , != ( eq , ne )

**条件运算**

+ If-then: (if) ? (then)
+  If-then-else: (if) ? (then) : (else) 
+ Default: (value) ?: (defaultvalue)

**特殊操作**

无操作： _

**设置属性值-th:attr**

+ 设置单个值

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
	<fieldset>
 		<input type="text" name="email" />
 		<input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
 	</fieldset>
</form>
```

+ 设置多个值

```html
<img src="../../images/gtvglogo.png" th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

+ 以上两个替代写法

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
<form action="subscribe.html" th:action="@{/subscribe}">
```

**迭代**

```html
<tr th:each="prod : ${prods}">
 	<td th:text="${prod.name}">Onions</td>
	<td th:text="${prod.price}">2.41</td>
 	<td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

```html
<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
	 <td th:text="${prod.name}">Onions</td>
	 <td th:text="${prod.price}">2.41</td>
 	 <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>

```

**条件运算**

```html
<a href="comments.html"
	th:href="@{/product/comments(prodId=${prod.id})}"
	th:if="${not #lists.isEmpty(prod.comments)}">view</a>
<div th:switch="${user.role}">
 	<p th:case="'admin'">User is an administrator</p>
 	<p th:case="#{roles.manager}">User is a manager</p>
 	<p th:case="*">User is some other thing</p>
</div>

```

#### 3. thymeleaf使用

1. **引入stater**

```xml
<dependency>
 	<groupId>org.springframework.boot</groupId>
 	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2. **自动配置好thymeleaf**

```java
ThymeleafAutoConfiguration.class
@Configuration(
    proxyBeanMethods = false
)
@EnableConfigurationProperties({ThymeleafProperties.class})
@ConditionalOnClass({TemplateMode.class, SpringTemplateEngine.class})
@AutoConfigureAfter({WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class})
public class ThymeleafAutoConfiguration {}
```

**自动配好的策略：**

+ 所有thymeleaf的配置都在ThymeleafProperties

+ 配置好了SpringTemplateEngin

+ 配置好了thymeleafViewResolver

+ 只需要直接开发页面

  ```java
  public static final String DEFAULT_PREFIX = "classpath:/templates/";
  public static final String DEFAULT_SUFFIX = ".html";
  ```

+ 防止重复提交表单的方法：重定向

```java
@Controller
public class HelloController {
    //    设置登陆页面为初始页面
    @RequestMapping(value = {"/", "/login"})
    public String toLogin() {
        return "login";
    }

    //处理登陆页面发送过来的信息
    @PostMapping("index")
    public String toIndex(User user, Model model, HttpSession session) {
        System.out.println(user);
        if (user.getUsername() != null && !StringUtils.isEmpty(user.getPassword())) {
//            登陆成功重定向到index.html,然后把登陆的信息存到session
            session.setAttribute("user", user);
            return "redirect:/index.html";

        } else {
            model.addAttribute("msg", "不能为空值，请输入");
            return "login";
        }
    }

    //专门写一个controller处理index页面
    @RequestMapping("index.html")
    public String index(Model model, HttpSession session) {
        if(session.getAttribute("user")!= null){
            return "index";
        }
        return "login";

    }
}
```

**抽取公共部分**

...



## 5.5 拦截器

**登陆检查**

1. 配置好拦截器要拦截哪些东西
2. 放入容器中,实现WebMvcConfig的addInterceptors

```java
public class LoginIntercepter implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
//        登陆检查逻辑
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        System.out.println(user);
        if (user != null) {
            return true;
        }
        else {
            response.sendRedirect("/");
            return false;
        }

    }

```

```java
@Configuration
public class AdminConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginIntercepter())
                .addPathPatterns("/**")
                .excludePathPatterns("/","/login","/index","/css/**");//放行静态资源和登陆的页面
    }
}
```

## 5.6 文件上传

```html
<h2>文件上传测试</h2>
<form th:action="@{/fileUpload}" enctype="multipart/form-data" method="post">
    单个文件：<input type="file" name="file">
    多个文件：<input type="file" name="files" multiple>
    <input type="submit" value="提交">
</form>
```

```java
@RequestMapping("fileUpload")
    public String fileUpload(@RequestPart("file") MultipartFile file,
                             @RequestPart("files") MultipartFile[] files) throws IOException {
        if (!file.isEmpty()){
            file.transferTo(new File("D://test//" + file.getOriginalFilename()));
        }
        for(MultipartFile file1:files){
            if (!file1.isEmpty()){
                file1.transferTo(new File("D://test//" + file1.getOriginalFilename()));
            }
        }
        return "success";
    }
```

配置最大的上传文件大小

```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=100MB
```



## 5.7 异常处理

### 5.7.1 错误处理

1. 默认规则
   + 默认情况下，Spring Boot提供 /error 处理所有错误的映射
   + 对于**机器**客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于**浏览器客**户端，响应一个“ whitelabel”错误视图，以 HTML格式呈现相同的数据
2. 自定义错误页
   + templates/error/404.html error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页

## 5.8 原生组件注入（Servlet，Filter，Listener）

### 5.8.1 使用Serlvet API（推荐）

```java
@WebServlet(urlPatterns = "/hello")
public class ServletTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello");
    }
}

同时：可以使用@WebFilter,@WebListener 注册filter和listener，都需要添加@ServletComponentScan
```

```java
@ServletComponentScan
@SpringBootApplication
public class Web02Application {

    public static void main(String[] args) {
        SpringApplication.run(Web02Application.class, args);
    }

}
```

效果：**直接响应，没有经过spring拦截器**

**原因分析：**

此时服务器有两个servlet：1.MyServlet，2.DispatcherServlet

有多个servlet的时候会使用精确原则

### 5.8.2 使用RegistrationBean

ServletRegistrationBean，FilterRegistrationBean，ServletListenerRegistrationBean



```java
@Configuration
public class MyRegistConfig {
 	@Bean
 	public ServletRegistrationBean myServlet(){
 		MyServlet myServlet = new MyServlet();
 		return new ServletRegistrationBean(myServlet,"/my","/my02");
    }
 	@Bean
 	public FilterRegistrationBean myFilter(){
 		MyFilter myFilter = new MyFilter();
// return new FilterRegistrationBean(myFilter,myServlet());
 		FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
 		return filterRegistrationBean;
    }
 	@Bean
 	public ServletListenerRegistrationBean myListener(){
 		MySwervletContextListener mySwervletContextListener = new MySwervletContextListener();
 		return new ServletListenerRegistrationBean(mySwervletContextListener);
    }
}

```



## 5.9 嵌入式web容器

原理分析套路：场景starter- xxxAutoConfiguration - 导入xxx组件 - 绑定xxxProperties - 绑定配置文件项



## 5.10 定制化原理

# 六 数据访问

## 6.1 sql

### 6.1.1 数据源的自动配置 HikariDataSource

1. 导入场景

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

没有导入驱动，导入驱动，有默认版本，但是数据库版本和驱动版本要对应

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
</dependency>
```

2. 修改配置

```yaml
spring:
 datasource:
 	url: jdbc:mysql://localhost:3306/db_account
 	username: root
 	password: 123456
 	driver-class-name: com.mysql.jdbc.Driver
 
```

3. 测试

```java
class Boot05WebAdminApplicationTests {
 @Autowired
 JdbcTemplate jdbcTemplate;
 @Test
 void contextLoads() {
// jdbcTemplate.queryForObject("select * from account_tbl")
// jdbcTemplate.queryForList("select * from account_tbl",)
 Long aLong = jdbcTemplate.queryForObject("select count(*) from account_tbl", Long.class);
 log.info("记录总数：{}",aLong);
 }
}
```

### 6.1.2 使用Druid数据源

整合第三方技术的两种方式：

+ 自定义
+ 找starter



1. 自定义方式：

```xml
引入数据源
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>

```

```java
创建配置类
@Configuration
public class DruidConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
   public DataSource dataSource(){
       DruidDataSource druidDataSource = new DruidDataSource();
       return druidDataSource;
   }
}
```

配置druid监控页面

```java
@Bean
    public ServletRegistrationBean servletRegistrationBean(){
        return new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
   }
```

2. stater方式

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```



### 6.1.3 整合mybatis操作

#### 1. 配置模式

+ 全局配置文件
+ SqlSessionFactory：自动配置好了
+ sqlSession：自动配置了SqlSessionTemplate，组合了SqlSession
+ Mapper：只要写了Mybatis的接口标注了@Mapper(也可以不写mapper，直接在启动类上方标注@MapperScan)

starter

springboot官方starter:`spring-boot-starter...`

第三方的starter： `*-spring-boot-startet`

```xml
<groupId>org.mybatis.spring.boot</groupId>
<artifactId>mybatis-spring-boot-starter</artifactId>
<version>2.2.2</version>
```

可以修改配置文件中mybatis开头的 



如使用纯配置模式，首先需要指定全局配置文件的位置和sql映射文件的位置

```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
```

然后写全局配置文件（也可以直接省略，直接在yaml中配置mybatis）和mapper文件

```yaml
mybatis:
#  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
```

dao层添加mapper注解

```java
@Mapper
public interface HelloDao {
    public int selectAcountFromTable();
}
```

#### 2. 使用纯注解,或者混合起来

```java
@Mapper
public interface HelloDao {
    public int selectAcountFromTable();

//    使用纯注解
    @Select("select count(*) from student")
    public int selectByAnno();
}
```

#### 3. 最佳实践

1. 引入mybatis-starter
2. 配置application.yaml中，指定mapper-location位置
3. 编写mapper接口并标注@Mapper
4. 简单语句直接注解，复杂语句使用mapper.xml

5. 使用MapperScan简化开发

### 6.1.4 整合MyBatis-Plus完成CRUD

+ 已经配置好了SqlSessionFactory 底层是容器中的数据源
+ 默认mapper文件：classpath*:/mapper/**/*.xml
+ 容器中也有SqlSessionTemplate
+ @Mapper标注的接口也会被自动扫描



## 6.2 NoSql

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://redis.cn/topics/lru-cache.html)，[事务（transactions）](http://redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

### 6.2.1 Redis自动配置

自动配置

+ RedisAutoConfiguration自动配置类。
+ 连接工厂是准备好的。LettuceConnectionConfiguration，JedisConnectionConfiguration
+ 自动注入RedisTemplate<object,object>：操作Redis的
+ 自动注入了StringRedisTemplate<String , String >

### 6.2.2 RedisTemplate域Lettuce

### 6.2.3 切换至jedis

# 七 单元测试

1. junit5具有spring5的功能

# 八 指标监控

# 九 原理解析



