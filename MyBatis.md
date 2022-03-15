# <center>MyBatis框架
## 1. 框架概述
### 1.1 三层架构
mvc：web开发中使用mvc架构模式

+ c控制器：接收请求，调用serlvet对象，显示请求的处理结果。当前使用servlet作为控制器
+ v视图：现在使用jsp，html，css，js。显示请求的处理结果，把m中的数据显示出来
+ m数据：来自数据库，来自文件，来自网络

mvc的作用：

+ 实现解耦合
+ 使系统的拓展更好，更容易维护

三层架构：

+ 界面层（视图层）：接收用户请求，调用service，显示请求的处理结果。
对应的包为：**controller**。
对应的框架为：**springMVC**

+ 业务逻辑层：处理业务逻辑，把数据返回给界面层。
对应的包为：**service**。
对应的框架为：**spring**
+ 持久层（数据库访问层）：访问数据库，或读取文件，访问网络。获取数据。
对应的包为：**dao**
对应的框架为：**MyBatis**

### 1.2 三层架构请求处理的流程
用户发起请求--->界面层--->业务逻辑层--->持久层--->数据库
## 2 MyBatis
### 2.1 第一个例子
1. 创建student表  id(pk),name,email,age
2. 新建maven项目
3. 修改pom.xml
	+ 加入mybatis依赖，mysql驱动，junit
	+ 在\<build\>中加入资源插件
4. 创建实体类student
5. 创建dao接口，定义操作数据库的方法
6. 创建xml文件（mapper文件），写sql语句
	mybatis推荐把sql语句和java代码分离
	mapper文件：定义在和dao接口在同一个目录，一个表一个mapper文件
7. 创建mybatis主要配置文件（xml文件）：只有一个，放在resources目录下
	1） 定义创建连接实例的数据源（dataSource）对象
	2）指定其他mapper文件的位置
8. 创建测试内容
