# Eureka

## 1. 搭建注册中心

**搭建 eureka-server**

引入 SpringCloud 为 eureka 提供的 starter 依赖，注意这里是用 **server**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**编写启动类**

注意要添加一个 `@EnableEurekaServer` **注解**，开启 eureka 的**注册中心**功能

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

**编写配置文件**

编写一个 application.yml 文件，内容如下：

```yml
server:
  port: 10086
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url: 
      defaultZone: http://127.0.0.1:10086/eureka
```

其中 `default-zone` 是因为前面配置类开启了注册中心所需要配置的 eureka 的**地址信息**，因为 eureka 本身也是一个微服务，这里也要将自己注册进来，当后面 eureka **集群**时，这里就可以填写多个，使用 “,” 隔开。

启动完成后，访问 http://localhost:10086/

## 2. 服务注册

> 将 user-service、order-service 都注册到 eureka

引入 SpringCloud 为 eureka 提供的 starter 依赖，注意这里是用 **client**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

在启动类上添加注解：`@EnableEurekaClient`

在 application.yml 文件，添加下面的配置：

```yml
spring:
  application:
      #name：orderservice
    name: userservice
eureka:
  client:
    service-url: 
      defaultZone: http:127.0.0.1:10086/eureka
```

3个项目启动后，访问 http://localhost:10086/

## 3.  服务拉取

> 在 order-service 中完成服务拉取，然后通过负载均衡挑选一个服务，实现远程调用

下面我们让 order-service 向 eureka-server 拉取 user-service 的信息，实现服务发现。

首先给 `RestTemplate` 这个 Bean 添加一个 `@LoadBalanced` **注解**，用于开启**负载均衡**。（后面会讲）

1. 修改orderservice代码，修改访问的Url路径

```java
String url = "http:userservice/user/"+order.getUserId();
```

2. 添加负载均衡注解

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

# Ribbon负载均衡

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530155940.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530160637.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530160831.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530161914.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530163019.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530163403.png)

# Nacos

## 1. 服务注册到Nacos

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530173203.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530173242.png)

只用修改配置文件，不用修改里面的sqlurl

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530173801.png)

## 2. 服务跨集群调用问题

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530175149.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530175302.png)

## 3. 环境隔离

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531171230.png)

增加命名空间

![image-20220531171444312](C:\Users\yuqian\AppData\Roaming\Typora\typora-user-images\image-20220531171444312.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531171547.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531171721.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531171749.png)

## 4. nacos注册中心细节分析

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531174044.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531174215.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531174549.png)