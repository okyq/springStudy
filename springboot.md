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



# 五 web开发

# 六 数据访问

# 七 单元测试

# 八 指标监控

# 九 原理解析



