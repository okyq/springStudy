# MyBatis框架
## 一. 框架概述
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
## 二 MyBatis
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
## 三 MyBatis框架dao代理
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
```java
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
```java
List<Student> selectStudentByNameOrAge(@Param("myname")String name,@Param("myage")int age);
```
```java
<select id="selectStudentByNameOrAge" resultType="com.yq.entity.Student">  
  select * from student where name=#{myname} or age=#{myage}  
</select>
```

#### 3.2.4   dao接口方法使用一个对象作为参数

方法的形参是一个java对象。这个java对象表示多个参数。使用对象的属性值作为参数使用

+ 简单的语法：**`#{属性名}`**,bytatis会调用此属性的getXXX()方法赋值，也可以使用**自定义**对象，即为**vo层**

```java
List<Student> selectByObject(Student student);
```

```java
<select id="selectByObject" resultType="com.yq.entity.Student">
        select * from student where name=#{name} or age = #{age}
</select>
```

+ 复杂的写法：#{name,javaType=java.lang.String , jdbcType=VARCHAR}

#### 3.2.5 dao接口中多个简单类型的参数，使用位置

参数位置：从左往右，依次是：0  1  2  3... 

agr0, arg1 , 依次类推....

```java
<select id="selectByPosition" resultType="com.yq.entity.Student">
        select * from student where name=#{arg0} or age=#{arg1}
</select>
```

```java
List<Student> students =mapper.selectByPosition("yuqian",5);
```

#### 3.2.6 dao接口是个map

mapper中resultType是map，查询出来的结果是map（字段名，字段对应的值），只能有一条记录

```java
Map<Object, Object> getUserToMap(@Param("id") int id);
```

```xml
<select id="getUserToMap" resultType="map">
	select * from t_user where id = #{id}
</select>
```



#### 3.2.7 查询多条数据是map集合

```java
List<Map<String, Object>> getAllUserToMap();
```

```xml
<select id="getAllUserToMap" resultType="map">  
	select * from t_user  
</select>
```





### 3.3 #和$的区别



#### 3.3.1 #占位符



语法：#{字符}

mybatis处理#{} 使用的jdbc对象是**PrepareStatment**

```java
<select id="selectById" resultType="com.yq.entity.Student" parameterType="int">  
  select * from student where id = #{id}  
</select>
    
mybatis创建出PrepareStatement对象，执行sql语句

String sql = "select * from student where id = ?  "

PrepareStatement pst = conn.PrepareStatement(sql)

pst.setInt(位置,参数)即为：pst.setInt(1,1)

ResultSet rs = pst.executeQuery(); 执行sql语句
```



特点：

+ 使用PrepareStatement执行sql语句，效率高
+ PrepareStatement能够避免sql注入
+ #{} 常常作为列值使用，位于等号右侧，#{}位置的值和数据类型有关

+ id为字符型，那么#{id}表示的就是'12'，如果id为整型，那么id就是12



#### 3.3.2 $ 占位符



语法： ${字符}

```java
<select id="selectById" resultType="com.yq.entity.Student" parameterType="int">  
  select * from student where id = ${id}  
</select>

 ${字符} 表示字符串的连接，把sql语句和其他的内容使用 字符串 的方式连接在一起
    
mybatis创建statement对象，执行sql语句
    Statement stmt = conn.createStatement(sql);
	ResultSet rs = stmt.ecuteQuery();
```



特点：

+ select * from tablename where id = ${id}
  如果字段id为整型，sql语句就不会出错，但是如果字段id为字符型， 那么sql语句应该写成select * from table where id = '${id}'。

+ 使用Statement对象，执行sql语句，效率低
+ 使用字符串连接，由sql注入的风险
+ 是原样使用的，不会区分数据
+ 常用作 **表名或列名**，**可以用来排序**	

```sql
使用列名排序
select * form student order by ${ColName}

既可以查询又可以排序
select * from student where name=#{name} order by ${ColName}

查询表名
select * from ${TableName} where name=#{name} order by ${ColName}
```

+ 需要使用**@Parm**命名参数

完整代码：

```sql
<select id="selectByTableNameAndColName" resultType="com.yq.entity.Student">
        select * from ${tableName} where name=#{name} order by ${colName} desc
 </select>
```

```java
List<Student> selectByTableNameAndColName(@Param("tableName")String tableName,
                                          @Param("colName")String colName,
                                          @Param("name")String name);
```



### 3.4 封装MyBatis输出结果

mybatis执行sql语句，得到ResultSet，转为jav对象

#### 3.4.1 resultType

+ 两种值：1. java全限定名称    2. 或者别名

```sql
<select id="selectByTableNameAndColName" resultType="com.yq.entity.Student">

使用全限定名称，意思是mybatis执行sql，把ResultSet中的数据转为Student类型的对象，mybatis会执行以下操作

1. 调用student的无参构造方法，创建对象
	Student student = new Student() //使用反射创建对象
2. 同名的列赋值给同名的属性
	student.setId(rs.getInt("id"));
	student.setName(rs.getString("name"));
3. 得到java对象，如果dao接口返回的是List集合，mybatis把student放入到L	ist集合

值得注意一点：mybatis在默认情况下调用无参构造和setter，当无参构造不存在的时候（即创建了有参构造），mybatis会调用有参构造，当setter不存在的时候，也会用一种方法为对象赋值
```

+ 返回简单类型

dao.class

```java
Integer selectNum();
```

mapper

```xml
<select id="selectNum" resultType="java.lang.Integer">
        select count(*) from student
</select>
```

+ 返回map

返回的map ，key是列名，value是值，sql语句只能获取一行记录，多余一行报错

```xml
<select id="selectReturnMap" resultType="java.util.HashMap">
        select * from student where id = #{id}
</select>
```





#### 3.4.2 resultMap

resultMap:结果映射。

**第一种用法**：自定义列名和java对象属性的对应关系

1. 先定义resultMap标签，指定列名和属性名的关系
2. 在select标签使用result属性，指定上面定义的resultMap的id值

vo层

```java
public class StudentVo {
    private int stu_id;
    private String stu_name;
}
```

mapper

```xml
<!--    使用ResultMap,当vo层字段和表字段不一样-->
    <select id="selectByResultMap" resultMap="stuResultMap">
        select id,name from student where id = #{id}
    </select>
<!--    定义resultMap-->
    <resultMap id="stuResultMap" type="com.yq.vo.StudentVo">
        <id property="stu_id" column="id"/>
        <result property="stu_name" column="name"/>
    </resultMap>
```

| 属性 | 描述 |
| :----------: | :--------------------------------------------------: |
|property	|需要映射到JavaBean 的属性名称。|
|column|	数据表的列名或者标签别名。|
|javaType|	一个完整的类名，或者是一个类型别名。如果你匹配的是一个JavaBean，那MyBatis 通常会自行检测到。然后，如果你是要映射到一个HashMap，那你需要指定javaType 要达到的目的。|
|jdbcType|	数据表支持的类型列表。这个属性只在insert,update 或delete 的时候针对允许空的列有用。JDBC 需要这项，但MyBatis 不需要。如果你是直接针对JDBC 编码，且有允许空的列，而你要指定这项。|
|typeHandler	|使用这个属性可以覆写类型处理器。这项值可以是一个完整的类名，也可以是一个类型别名。|


dao

```java
StudentVo selectByResultMap(@Param("id")int id);
```





**********

**********





**第二种用法：** 一对一关联查询

一个学生（student）对应多个订单，一个订单（order）对应一个学生

所以应该这样建表

在order表中加入 stu_id 并设置外键，表示一个订单对应一个学生

`student表的属性：id(pk),  name,  email,  age`

`ordertabel表的属性：order_id(pk),  stu_id(fk),  order_name`



Order_Student.class

因为一个订单对应一个学生，所以在order表中加入student

```java
public class Order_Student {
    private int order_id;
    private int stu_id;
    private String order_name;
    private Student student;
 }
```

Student.class

```java
public class Student {
    private int id;
    private String name;
    private String email;
    private int age;
}
```

dao

```java
Order_Student selctByOneToOne( @Param("id")int id );
```

mapper

```xml
<!--    使用resultMap一对一-->
   <select id="selctByOneToOne" resultMap="oneToOne">
       select o.*, s.* from ordertable as o left join student as s  on o.stu_id = s.id where order_id=#{id}
   </select>
    
    <resultMap id="oneToOne" type="com.yq.entity.Order_Student">
        <id property="order_id" column="order_id"/>
        <result property="order_name" column="order_name"/>
        <result property="stu_id" column="stu_id"/>
        <association property="student" javaType="com.yq.entity.Student">
            <id property="id" column="id"/>
            <result property="name" column="name"/>
            <result property="email" column="email"/>
            <result property="age" column="age"/>
        </association>
    </resultMap>
注意： 使用 association 和  javaType！！！
```



****

****



**第三种用法：** 一对多关联查询

一个学生可以有多个订单

Student_Order.class

```java
public class Student_Order {
    private int id;
    private String name;
    private String email;
    private int age;
    private List<Order> orders;
}
```

dao

```java
Student_Order selectByOneToMany(@Param("id")int id);
```

mapper

```xml
<!--    使用resultMap一对多查询-->
    <select id="selectByOneToMany" resultMap="ontToMany">
        select s.*, o.* from student as s left join ordertable as o  on o.stu_id = s.id where s.id = #{id}
    </select>

    <resultMap id="ontToMany" type="com.yq.entity.Student_Order">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="email" column="email"/>
        <result property="age" column="age"/>
        <collection property="orders" ofType="com.yq.entity.Order">
            <id property="order_id" column="order_id"/>
            <result property="stu_id" column="stu_id"/>
            <result property="order_name" column="order_name"/>
        </collection>
    </resultMap>

注意 ：使用collection  和   ofType！！
```









### 3.5 自定义别名

1. 第一种:

​		优点：别名可以自定义，缺点：每个类型必须单独定义

```xml
<typeAliases>
<!--        第一种格式：
                    type: java类型全限定名称
                    alias：自定义别名-->
        <typeAlias type="com.yq.entity.Student" alias="student"/> </typeAliases>
```

2. 第二种方式

   name中写包名，mybatis会把包中所有的类名作为别名

   缺点：重名不同包

```xml
<!--        第二种格式：
                    name中写包名，mybatis会把包中所有的类名作为别名-->
<package name="com.yq.entity"/>
```

目前推荐的是**不使用别名**。



## 四 特殊sql执行



### 4.1 模糊查询

```java
 List<Student> selectByLike(@Param("name")String name);
```

```xml
<!--    模糊查询-->
    <select id="selectByLike" resultType="com.yq.entity.Student">
        select * from student where name like "%"#{name}"%"
    </select>
```

**注意**:  "%"#{name}"%" 使用最常见

 ### 4.2 批量删除

+ 只能使用\${}，如果使用#{}，则解析后的sql语句为`delete from t_user where id in ('1,2,3')`，这样是将`1,2,3`看做是一个整体，只有id为`1,2,3`的数据会被删除。正确的语句应该是`delete from t_user where id in (1,2,3)`，或者`delete from t_user where id in ('1','2','3')`

```java
 int deleteMany(@Param("ids")String ids);
```

```xml
<delete id="deleteMany">
        delete from student where id in (${ids})
</delete>
```

