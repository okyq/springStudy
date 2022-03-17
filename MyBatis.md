

# 	MyBatis框架

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

**第一种用法**：自定义列名和java对象属性的对应关系,也可以在查询表的时候设置别名

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

同时**可以使用级联查询和分布查询**

**分布查询**：

一个订单对应一个学生，先查出订单，再根据订单查询学生

思考方式：

+ 订单中包含了学生信息，所以在查询订单的时候需要返回学生信息，使用resultMap

```xml
<select id="selectByStepOne" resultMap="selectByStep">
        select * from ordertable where order_id=#{id}    
</select>
```

+ 从订单中查询出来的stu_id，根据这个id查询学生信息

```xml
<select id="selectById" resultType="student" parameterType="int">
        select * from student where id = #{id}
</select>
```

+ 在resultMap中将stu_id 给 selectById

```xml
<resultMap id="selectByStep" type="com.yq.entity.Order_Student">
        <id property="order_id" column="order_id"/>
        <result property="order_name" column="order_name"/>
        <result property="stu_id" column="stu_id"/>
        <association property="student" column="stu_id" select="com.yq.dao.StudentDao.selectById"/>
</resultMap>
<!-- association定义关联对象的封装规则
	 		select:表明当前属性是调用select指定的方法查出的结果
	 		column:指定将哪一列的值传给这个方法
	 		流程：使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
	 	 -->
```

>分布查询的优点：可以实现延迟加载，但是必须在核心配置文件中设置全局配置信息
>
>lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载（即为分布查询的第二步）
>
>aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载  ，为了延迟加载应该关闭该对象，默认关闭
>
>此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。此时可通过association和collection中的**fetchType**属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加载)|eager(立即加载)"
>
>```java
>@Test
>    public void selectByStepOne(){
>        try(SqlSession session = MyBatisUtils.getSqlSession()) {
>            OrderDao mapper = session.getMapper(OrderDao.class);
>            Order_Student os = mapper.selectByStepOne(1);
>            System.out.println(os.getOrder_name());
>        }
>    }
>```
>
>当最后只需要Order_name时，就不会执行分布查询的第二步



****

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



同样也可以使用分布查询

**分布查询**

思考：一个学生有多个订单，先查询学生，再根据学生的id查询订单，根据学生的id查询订单的时候，可以有多个订单

通过学生id查询学生信息

```java
 Student_Order selectByStep(@Param("id") int id);
```

```java
<select id="selectByStep" resultMap="selectStep">
        select * from student where id = #{id}
</select>
```

通过学生id查询订单信息

```java
List<Order> selectByStuId(@Param("stu_id") int id);
```

```xml
<select id="selectByStuId" resultType="com.yq.entity.Order">
        select * from ordertable where stu_id = #{stu_id}
</select>
```

resultMap

```xml
<resultMap id="selectStep" type="com.yq.entity.Student_Order">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="email" column="email"/>
        <result property="age" column="age"/>
        <collection property="orders" select="com.yq.dao.OrderDao.selectByStuId"
                    column="id"/>
</resultMap>
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

### 4.3 添加功能获取自增的主键

```java
//    添加功能获取自增的主键
    int insertStudentGetId(Student student);
```

```xml
<insert id="insertStudentGetId" useGeneratedKeys="true" keyProperty="id">
        insert into student values(null,#{name},#{email},#{age})
</insert>
```

在insert标签中声明useGeneratedKeys和keyProperty

## 五 动态sql

Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串时的痛点问题

### 5.1 if

- if标签可通过test属性（即传递过来的数据）的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中的内容不会执行
- 在where后面添加一个恒成立条件`1=1`，防止sql语句变为：`select * from t_emp where and age = ? and sex = ? and email = ?`，此时`where`会与`and`连用，SQL语句会报错

```xml
<select id="selectSynamic" resultType="com.yq.entity.Student">
        select * from student where 1=1
        <if test="name != null and name !=''">
           and  name = #{name}
        </if>
</select>
```

### 5.2 where

- where和if一般结合使用：
 - 若where标签中的if条件都不满足，则where标签没有任何功能，即不会添加where关键字  

+ 若where标签中的if条件满足，则where标签会自动添加where关键字，并将条件最前方多余的and/or去掉  
+ 注意：where标签不能去掉**条件后**多余的and/or

```xml
<select id="selectSynamic" resultType="com.yq.entity.Student">
        select * from student
        <where>
            <if test="name != null and name !=''">
                and  name = #{name}
            </if>
        </where>
</select>
```

### 5.3 trim

trim用于去掉或添加标签中的内容  

**常用属性**

 - prefix：在trim标签中的内容的前面添加某些内容  
 - suffix：在trim标签中的内容的后面添加某些内容 
 - prefixOverrides：在trim标签中的内容的前面去掉某些内容  
 - suffixOverrides：在trim标签中的内容的后面去掉某些内容
- 若trim中的标签都不满足条件，则trim标签没有任何效果，也就是只剩下`select * from t_emp`

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select * from t_emp
	<trim prefix="where" suffixOverrides="and|or">
		<if test="empName != null and empName !=''">
			emp_name = #{empName} and
		</if>
		<if test="age != null and age !=''">
			age = #{age} and
		</if>
		<if test="sex != null and sex !=''">
			sex = #{sex} or
		</if>
		<if test="email != null and email !=''">
			email = #{email}
		</if>
	</trim>
</select>
```

### 5.4 choose,when,otherwise

相当于 if。。elseif。。else

+ choose相当于只是一个父标签,when相当于if，otherwise相当于else
+ when只要有一个满足，其余都不会执行
+ 如果都不满足，就执行otherwise

```xml
<select id="getEmpByChoose" resultType="Emp">
	select * from t_emp
	<where>
		<choose>
			<when test="empName != null and empName != ''">
				emp_name = #{empName}
			</when>
			<when test="age != null and age != ''">
				age = #{age}
			</when>
			<when test="sex != null and sex != ''">
				sex = #{sex}
			</when>
			<when test="email != null and email != ''">
				email = #{email}
			</when>
			<otherwise>
				did = 1
			</otherwise>
		</choose>
	</where>
</select>
```



### 5.5 foreach

+ 通过foreach实现批量删除

```java
int deleteByArray(@Param("ids") int[] ids);
```

```xml
<delete id="deleteByArray">
        delete from student where id in 
        <foreach collection="ids" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>
</delete>
```

+ 通过foreach实现批量添加

两种方法都可以

第一种

```java
int insertByArray(@Param("students") List<Student> students);
```

```xml
<insert id="insertByArray" keyProperty="id" useGeneratedKeys="true">
        insert into student values
        <foreach collection="students" item="student" separator=",">
            (null,#{student.name},#{student.email},#{student.age})
        </foreach>
</insert>
```

第二种

```java
int insertByArrayTest(@Param("students") Student[] students);
```

```xml
<insert id="insertByArrayTest" keyProperty="id" useGeneratedKeys="true">
        insert into student values
        <foreach collection="students" item="student"  separator=",">
            (null,#{student.name},#{student.email},#{student.age})
        </foreach>
</insert>
```



### 5.6 sql片段

- sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入
- 声明sql片段：`<sql>`标签

```xml
<sql id="empColumns">eid,emp_name,age,sex,email</sql>
```

+ 引用sql片段：`<include>`标签

```xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
	select <include refid="empColumns"></include> from t_emp
</select>
```

## 六 mysql缓存

### 6.1 MyBatis一级缓存，默认开启

+ 一级缓存是SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问  
+ 使一级缓存失效的四种情况：

```
1. 不同的SqlSession对应不同的一级缓存  
2. 同一个SqlSession但是查询条件不同
3. 同一个SqlSession两次查询期间执行了任何一次增删改操作
4. 同一个SqlSession两次查询期间手动清空了缓存 sqlsession.clearCach()
```



### 6.2 MyBatis二级缓存

- 二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取  
- 二级缓存开启的条件:

```
1. 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
2. 在映射文件中设置标签 <cache/>
3. 二级缓存必须在SqlSession关闭或提交之后有效
4. 查询的数据所转换的实体类类型必须实现序列化的接口
```

+ 使二级缓存失效的情况：两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

### 6.3 二级缓存相关配置

在mapper配置文件中添加的cache标签可以设置一些属性

- eviction属性：缓存回收策略  

  - LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。  
  - FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。  
  - SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。  
  - WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
  - 默认的是 LRU

- flushInterval属性：刷新间隔，单位毫秒

  默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句（增删改）时刷新

- size属性：引用数目，正整数

  代表缓存最多可以存储多少个对象，太大容易导致内存溢出

- readOnly属性：只读，true/false

  - true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。  
  - false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false

### 6.3 缓存查询顺序

- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用  
- 如果二级缓存没有命中，再查询一级缓存  
- 如果一级缓存也没有命中，则查询数据库  
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

## 七 逆向工程

逆向工程的maven配置 pom.xml

```xml
<dependencies>
	<!-- MyBatis核心依赖包 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.9</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.13.2</version>
		<scope>test</scope>
	</dependency>
	<!-- MySQL驱动 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.27</version>
	</dependency>
	<!-- log4j日志 -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
</dependencies>
<!-- 控制Maven在构建过程中相关配置 -->
<build>
	<!-- 构建过程中用到的插件 -->
	<plugins>
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
			<!-- 插件的依赖 -->
			<dependencies>
				<!-- 逆向工程的核心依赖 -->
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.2</version>
				</dependency>
				<!-- 数据库连接池 -->
				<dependency>
					<groupId>com.mchange</groupId>
					<artifactId>c3p0</artifactId>
					<version>0.9.2</version>
				</dependency>
				<!-- MySQL驱动 -->
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>8.0.27</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```

MyBatis核心配置文件 mybatis.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <typeAliases>
        <package name=""/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name=""/>
    </mappers>
</configuration>
```

创建逆向工程的配置文件 generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
    targetRuntime: 执行生成的逆向工程的版本
    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
    MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.atguigu.mybatis.pojo" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.atguigu.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.atguigu.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```

举个例子：

```java
@Test
public void mytest(){
        try(SqlSession session = MyBatisUtils.getSqlSession()) {
            StudentMapper mapper = session.getMapper(StudentMapper.class);
            StudentExample example = new StudentExample();
            example.createCriteria().andAgeBetween(5,7).andNameEqualTo("yuqian");
            example.or().andNameEqualTo("aa");
            List<Student> studentList  = mapper.selectByExample(example);
            System.out.println(studentList);
        }
}
```



## 八 分页操作

pom.xml中添加依赖

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.2.0</version>
</dependency>
```

在mybatis.xml中配置插件

```xml
<plugins>
	<!--设置分页插件-->
	<plugin interceptor="com.github.pagehelper.PageInterceptor">	     </plugin>
</plugins>
```

开启分页功能

- 在查询功能之前使用`PageHelper.startPage(int pageNum, int pageSize)`开启分页功能
 - pageNum：当前页的页码  
 - pageSize：每页显示的条数

```java
public void mytest(){
        try(SqlSession session = MyBatisUtils.getSqlSession()) {
            StudentMapper mapper = session.getMapper(StudentMapper.class);
            StudentExample example = new StudentExample();
            example.createCriteria().andAgeBetween(5,7).andNameEqualTo("yuqian");
            example.or().andNameEqualTo("aa");
            PageHelper.startPage(1,1);
            List<Student> studentList  = mapper.selectByExample(example);
            System.out.println(studentList);
        }
}
```

