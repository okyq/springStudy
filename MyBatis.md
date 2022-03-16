# MyBatis框架
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

详细代码：
mybatis.xml 官方文档的配置文件
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
 "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  
	 <environments default="development">  
	 <environment id="development">  
		 <transactionManager type="JDBC"/>  
		 <dataSource type="POOLED">  
			 <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
			 <property name="url" value="jdbc:mysql://localhost:13306/springdb?useUnicode=true&amp;characterEncoding=utf-8"/>  
			 <property name="username" value="root"/>  
			 <property name="password" value="root"/>  
		 </dataSource> 
	 </environment> 
	</environments>  
<!-- 指定其他mapper文件的位置  
-->  
	<mappers>  
<!-- 从target/class路径开始,用斜杠分割  
 一个mapper resource指定一个mapper文件  
-->  
		 <mapper resource="com/yq/dao/StudentDao.xml"/>  
	</mappers>
 </configuration>
```
StudentDao.xml
```
<mapper namespace="com.yq.dao.StudentDao">  
 <select id="selectStudentById" resultType="com.yq.entity.Student">  
  select * from student where id = #{id}  
  </select>  

</mapper>  

```

+  id:要执行的sql语句的唯一标识，推荐使用dao接口中的方法名称  
 + resultType：执行sql语句后，把数据赋值给哪个类型的对象  
 使用java对象的全限定的名称  
+  #{id}:占位符  


+ namespace: 命名空间，必须有值，推荐使用Dao接口的全限定名称  
作用：参与识别sql语句  
在mapper中可以写：\<insert\>\<update\>\<delete\>\<select\>  

DemoTest.class
```
public void testSelectStudentByid() throws IOException {  
    //1. 定义mybatis主配置文件的位置，从类路径开始的相对路径  
  String config = "mybatis.xml";  
  
  //2. 读取主配置文件  
  InputStream inputStream = Resources.getResourceAsStream(config);  
  
  //3. 创建SqlSessionFactory对象  
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);  
  
  //4. 获取SqlSession对象  
  SqlSession session = factory.openSession();  
  
  //5. 指定要执行的sql语句的id  
 //sql的id： namespace+ . + 标签的id值  
  String sqlID = "com.yq.dao.StudentDao.selectStudentById";  
  
  //6. 通过SqlSession的方法执行sql语句  
  Student student = session.selectOne(sqlID,1);  
  System.out.println(student);  
  
  //7. 关闭sqlSession对象  
  session.close();  
}
```
### 2.2 添加日志功能
在mybatis.xml的配置文件中的\<configuration\>标签中加入
```
<settings>  
	 <setting name="logImpl" value="STDOUT_LOGGING"/>  
</settings>
```
### 2.3 自动提交和手动提交事务
mybatis默认执行sql语句是手工提交事务，在insert，update，delete之后需要提交事务
使用官方推荐的写法进行书写
```
<insert id="insertStudent">  
	  insert into student value (#{id},#{name},#{email},#{age})  
</insert>
```
```
try(SqlSession session = factory.openSession()){  
    StudentDao mapper = session.getMapper(StudentDao.class);  
    int a =mapper.insertStudent(new Student(2,"yuqian","efaef",5));  
    session.commit();  
}
```
因为SqlSession继承了Closeable，Closeable继承了AutoCloseable
### 2.4 MyBatis的一些重要对象
1. **Resources**：mybatis框架中的对象，一个作用：读取主要配置信息，返回输入流
`InputStream inputStream = Resources.getResourceAsStream("mybatis.xml");`
2.  **SqlSessionFactoryBuilder**:负责创建SqlSessionFactory对象
	`SqlSessionFactory factory  = new SqlSessionFactoryBuilder().build(inputStream)`
3. **SqlSessionFactory** :重要对象，创建此对象需要更多的资源和时间，创建一个就可以了
4. **SqlSessionFactory中的方法**
	+ **openSession()** ： 获取一个默认的SqlSession对象，默认需要手工提交事务
	+ **openSession(boolean)**：boolean参数表示是否自动提交事务
5. **SqlSession对象**
	SqlSession的作用是提供了大量的执行sql的方法
	1. selectOne：执行sql语句最多得到一行记录
	2. selectList：执行sql语句，返回多行记录
	3. selectMap：执行sql语句，得到一个Map集合
	4. insert：执行insert语句
	5. update：执行update语句
	6. delete：执行delete语句
	7. commit：提交事务
	8. rollback：回滚事务
## 3 MyBatis框架dao代理
### 3.1 动态代理
mybatis根据dao接口，创建一个内存中的接口实现类对象，由**mybatis**创建StudentDao接口的实现类Proxy（StudentDaoImpl），使用框架创建的StudentDaoImpl代替手工创建的StudentDaoImpl。不用开发人员创建Dao接口的实现类
```
StudentDao mapper = session.getMapper(StudentDao.class);
```
**使用动态代理的要求**

1. mapper中的namespace必须是dao接口的全限定名称
2. mapper中的sql语句的id必须是dao中的方法名
### 3.2 深入理解参数
通过java程序把数据传入mapper文件，主要是指dao方法的形参
#### 3.2.1 parameterType
parameterType：表示参数类型，指定dao方法的形参数据类型。这个形参的数据类型是给mybatis在给sql语句的参数赋值的时候使用。PreparedStatement.setXXX（位置，值）
**可以不写**，但是为了代码的易读性还是写上
可以使用**别名**，否则使用**全类名**
```
<select id="selectById" resultType="com.yq.entity.Student" parameterType="int">  
	  select * from student where id = #{id}  
</select>
```
#### 3.2.2一个简单类型的参数
Dao接口中的方法的参数只有一个简单类型（java基本类型和String），占位符 **#{任意字符 }**，和方法名称无关
```
<select id="selectById" resultType="com.yq.entity.Student" parameterType="int">  
  select * from student where id = #{id}  
</select>
```
**和**
```
<select id="selectById" resultType="com.yq.entity.Student" parameterType="int">  
  select * from student where id = #{abc}  
</select>
```
**是一样的**


#### 3.2.3dao接口方法由多个简单类型的参数
@Param：命名参数，在方法的形参前面使用，在mapper中使用自定义的value值代替`#{}`中的值
```
List<Student> selectStudentByNameOrAge(@Param("myname")String name,@Param("myage")int age);
```
```
<select id="selectStudentByNameOrAge" resultType="com.yq.entity.Student">  
  select * from student where name=#{myname} or age=#{myage}  
</select>
```

### 3.3 封装MyBatis输出结果
### 3.4 模糊like
