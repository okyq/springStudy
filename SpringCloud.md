# 感谢黑马的课程

# 一，Eureka

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

# 二，Ribbon负载均衡

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530155940.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530160637.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530160831.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530161914.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530163019.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220530163403.png)

# 三，Nacos

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

## 5. Nacos配置管理

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220531175543.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220601145008.png)

![](https://cdn.jsdelivr.net/gh/yqimg/img/20220601145023.png)

![image-20220601151453828](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601151453828.png)

![image-20220601151600137](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601151600137.png)

![image-20220601151933848](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601151933848.png)

## 6. 配置热更新

![image-20220601152333738](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601152333738.png)

![image-20220601152351903](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601152351903.png)

![image-20220601152746935](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601152746935.png)

## 7. 多环境配置共享

![image-20220601152928394](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601152928394.png)

![image-20220601154530560](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601154530560.png)

![image-20220601154542947](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601154542947.png)

## 8. Nacos集群搭建

![image-20220601154751954](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601154751954.png)

![image-20220601154818948](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601154818948.png)

![image-20220601154900136](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601154900136.png)

![image-20220601155857540](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601155857540.png)

# 四，http客户端Feign

## 1. Feign的基本使用

![image-20220601160929837](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601160929837.png)

![image-20220601161042164](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601161042164.png)

![image-20220601161253329](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601161253329.png)

用Feign调用

![image-20220601161626431](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601161626431.png)

其中Feign内部集成了Ribbon，直接可以负载均衡

![image-20220601161744854](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601161744854.png)



## 2. 自定义Feign配置

![image-20220601161900621](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601161900621.png)

![image-20220601164926259](https://cdn.jsdelivr.net/gh/okyq/img/image-20220601164926259.png)

## 3. Feign性能优化

![image-20220602094503535](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602094503535.png)

![image-20220602094642357](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602094642357.png)

![image-20220602095005664](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602095005664.png)

## 4. 最佳实践

![image-20220602095443180](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602095443180.png)

![image-20220602095636345](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602095636345.png)

![image-20220602095703965](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602095703965.png)

![image-20220602102121325](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602102121325.png)

这样抽取之后，以后所有的微服务中，只要有需要UserClient的微服务就可以通过引用feign-api来调用。

![image-20220602102258355](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602102258355.png)

![image-20220602102758360](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602102758360.png)

# 五，统一网关Gateway

## 1. 网关的作用

![image-20220602103248954](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602103248954.png)

![image-20220602103541204](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602103541204.png)

![image-20220602103735845](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602103735845.png)

## 2. 搭建网关

![image-20220602103816927](https://cdn.jsdelivr.net/gh/okyq/img/image-20220602103816927.png)

![image-20220608163903913](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608163903913.png)

## 3. 路由断言工厂

![image-20220608164619017](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608164619017.png)

![image-20220608164802992](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608164802992.png)

![image-20220608164819136](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608164819136.png)

## 4. 路由过滤器

![image-20220608165524811](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608165524811.png)

![image-20220608165643841](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608165643841.png)

eg

![image-20220608165816320](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608165816320.png)

![image-20220608165918216](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608165918216.png)

![image-20220608170033237](https://cdn.jsdelivr.net/gh/okyq/img/image-20220608170033237.png)

## 5. 全局过滤器

![image-20220609133525186](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609133525186.png)

![image-20220609133541445](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609133541445.png)

![image-20220609134157286](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609134157286.png)

![image-20220609134120826](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609134120826.png)

定义顺序的时候还可以实现Order接口并实现Order方法

![image-20220609134338888](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609134338888.png)

![image-20220609134415327](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609134415327.png)

## 6. 过滤器执行顺序

![image-20220609134847473](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609134847473.png)

![image-20220609135116198](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609135116198.png)

## 7. 跨域问题解决

![image-20220609140515446](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609140515446.png)

![image-20220609140652026](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609140652026.png)

# 六，Docker

## 1. 初识Docker

![image-20220609142021493](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609142021493.png)

**镜像和容器**

![image-20220609142649360](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609142649360.png)

![image-20220609142927440](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609142927440.png)

## 2. 安装

## 3. 常用命令

![image-20220609154704736](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609154704736.png)

![image-20220609155948152](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609155948152.png)

![image-20220609162744390](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609162744390.png)

![image-20220609163212839](https://cdn.jsdelivr.net/gh/okyq/img/image-20220609163212839.png)

![image-20220610162008653](https://cdn.jsdelivr.net/gh/okyq/img/image-20220610162008653.png)