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

+ @PathVariable、

  + rest风格

  + ```java
    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)  
    public String getUserByID( @PathVariable("id")Integer id ){  
        System.out.println("根据id查询用户信息,传入的id为："+id);  
        return "success";  
    }  
    ```

    

+ @RequestHeader、

  + 是将请求头信息和控制器方法的形参创建映射关系

+ @ModelAttribute、

+ @RequestParam、

  + 当形参的参数名和请求参数的参数名不一致的时候，可以使用`@RequestParam` 

+ @MatrixVariable、

+ @CookieValue、

  + 是将cookie数据和控制器方法的形参创建映射关系，

+ @RequestBody
  + @RequestBody可以获取请求体，需要在控制器方法中设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

还有一些springmvc中的api之类



### 5.2.2 POJO封装过程

### 5.2.3 参数处理原理

## 5.3 响应数据与内容协商

## 5.4 视图解析与模板引擎

## 5.5 拦截器

## 5.6 跨域

## 5.7 异常处理

## 5.8 原生组件注入

## 5.9 嵌入式web容器

## 5.10 定制化原理

# 六 数据访问

# 七 单元测试

# 八 指标监控

# 九 原理解析



