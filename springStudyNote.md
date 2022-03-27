# <center> Spring学习笔记
## 1.开始

+ spring的配置文件使用xml
+ 通过bean创建对象
+ 加载配置文件：
```ApplicationContext context =  new ClassPathXmlApplicationContext("bean1.xml");```
+ 从配置文件中创建对象```User user =  context.getBean("user", User.class);```

## 2. ioc容器
### 2.1 底层原理
+ xml解析，工厂模式，反射
+ ioc过程：
	- 第一步，xml配置文件，配置创建的对象
			```
			<bean id ="dao" class="com.yq.UserDao"></bean>
			```
			- 第二步，有service类和dao类，创建工厂类 
			```
			class Userfactory{
				public static UserDao getDao(){
					String classValue = class属性值（xml解析）
					Class class = Class.forName(classValue);
					return (UserDao)class.newInstance().
				}
			}
			```
		
### 2.2 ioc接口（beanFactory）

+  IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
+ Spring提供了IOC容器实现的两种方式
	1. BeacFactory：IOC容器的基本实现方式，是Spring内部使用的接口口，不提供开发人员进行使用（==加载配置文件的时候不会创建对象，只有在获取对象或者使用对象的时候才会创建对象==）
	2. ApplicationContext：BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用（==加载配置文件的时候就会把在配置文件中的对象创建==）
		+ ApplicationContext有两个实现类
			1. FileSystemXmlApplicationContext（要写具体盘符，定位到电脑的具体路径）
			2. ClassPathXmlApplicationContext（src文件下的xml文件）

### 2.3 ioc操作 bean管理（基于xml）
1. 创建对象
	+ bean管理指的是两个操作Spring创建对象，Spring注入属性
	+ bean标签中的属性介绍：
		1. id属性：唯一标识
		2. class属性：类的全路径
		3. name属性：也是表示标识，里面可以加特殊符号
			+ 创建对象的时候默认执行无参构造
2. 注入属性
	+ DI：依赖注入，就是注入属性 
		1.  用set方法注入
			在bean标签里面用property标签注入属性（name表示类属性名称，value表示类属性值 ）
		2.  用有参构造器注入
			在bean标签中用constructor-arg标签注入属性（name表示类属性名称，value表示类属性值 ）	
		3. p名称空间注入
			```
			xmlns:p="http://www.springframework.org/schema/p"
			```
		4. xml注入其他类型属性
			+ null值 添加 null标签
			+ 属性值包含特殊符号 
			` <![CDATE[内容]]>`
		5. 注入属性 外部bean
			 ```
			 (1)创建两个类
			 (2)在service中调用dao中的方法
			 (3)在spring配置文件中
			 <property name="类中的属性名" ref="要引用的beanid"></property>
			```
		6. 注入属性，内部bean和级联属性
			+ 一对多的关系：部门和员工
			```
			<bean id="emp" class="com.yq.spring5.bean.Emp">  
				<property name="empname" value="余谦"/>  
				<property name="dept">  
					<bean id="dept" class="com.yq.spring5.bean.Dept">  
						 <property name="deptname" value="保安"/>  
					</bean> 
				</property>
			</bean>
			```
		7.  注入集合属性
			|        类型 | 代码 |
            | :----: | :----: |
            | 数组类型 | property标签里面加array标签再加value标签|
            |list类型 | property标签里面加list标签|
            |set类型| property标签里面加set标签|
            |map类型| property标签里面加map标签，再加entry标签|
            |集合里面有对象|把value标签换成ref标签|
            |用util标签注入| xmlns:util="http://www.springframework.org/schema/util"<br>xmlns:p="http://www.springframework.org/schema/p"<br>xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd<br>http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"<br>标签就是把第一行的代码做修改
         
			8.FactoryBean
			
			+ Spring有两种类型的bena，一种是普通bean，一种是工厂bean（FactoryBean）
			+ 普通bean：在配置文件中定义的类型就是返回的类型
			+ 工厂bean：在配置文件中定义的类型和返回的类型可以不一样（类实现FactoryBean，实现getObiect方法，返回需要的bean类型）
3. bean作用域
	1. 在Spring里面，设置创建bean实例是单实例还是多实例（context.getBean()获取多次，但是都是同一个bean）
	2. 在Spring里面，默认情况下是单实例的情况
	3. 如何设置单实例还是多实例
		+ bean标签里面的scope可以设置单实例还是多实例
		+  singleton 表示单实例对象，加载spring配置文件的时候就会创建单实例对象
		+ prototype表示多实例对象，不是在加载配置文件的时候创建对象，在调用getBean()方法的时候创建多实例对象
4. bean生命周期
	1. 通过构造器创建bean实例（无参构造）
	2. 为bean的属性设置值和对其他bean引用（调用set方法）
	3. **把bean传递到前置处理器处理（实现接口BeanPostProcessor，调用方法postProcessBeforeInitialization()  ）**
	4. 调用bean的==初始化==方法（在bean标签里面设置，需要进行配置初始化的方法）
	5. **把bean传递到后置处理器处理（实现接口BeanPostProcessor，调用方法postProcessAfterInitialization()  )**
	6. bean可以使用了（对象获取到了）
	7. 当容器关闭的时候，调用bean的==销毁==的方法（需要配置销毁的方法，在bean标签里面设置，并且==手动调用==）
5. xml自动装配
	+ 根据指定的装配规则（属性名称或者属性类型），spring自动将匹配的属性值进行注入
	+ autowire标签，byName, byType
6. 引入外部属性文件
### 2.4 基于注解的方式
#### 2.4.1 什么是注解
1. 注解是代码的特殊标记，格式：@注解名称（属性名称=属性值，属性名称=属性值）
2. 使用注解，注解作用在类上面，方法上面，属性上面
3. 使用注解可以简化xml配置
#### 2.4.2 针对创建对象提供注解
1. @Component（普通）
2. @Service
3. @Controller
4. @Repository（dao）
5. 四个注解功能是一样的，都可以用来创建bean
6. 注解的value值可以不写，默认首字母小写
#### 2.4.3 基于注解方式实现对象创建
1. 引入依赖： spring-aop。。。.jar
2. 开启组件扫描
	1. 引入context名称空间
	2. ```
		使用默认filters扫描
		<context:component-scan base-package="想扫描的包" ></context:component-scan>
		```
	4. 如果想扫描多个包，用逗号隔开，或者扫描上层包
3. 组件扫描配置
```
<context:component-scan base-package="想扫描的包"  use-default-filters="false">
	<context:include-filter type="annotation"（不适用默认filter，指定扫描哪些类
		expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

```
<context:component-scan base-package="想扫描的包" >
	<context:exclude-filter type="annotation"(用默认filter，不扫描哪些类)
		expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```
#### 2.4.4基于注解方式实现属性注入
1. @AutoWired 根据属性类型自动装配
2. @Qualifier 根据属性名称进行注入  
	和上面的@AutoWired一起使用
3. @Resource 可以根据属性类型注入，也可以根据名称注入
	@Resource默认按照属性名称来注入，按照类型注入要写@Resource（type=UserDaoImpl）这个注解不是spring的注解，是==java的注解==，在java11被移除
4. @Value 注入普通类型属性
#### 2.4.5完全注解开发
1. 创建配置类，替代xml文件
	```
	@Configuration
	@ComponentScan(basePackages = {"com.yq.spring"})
	public class StringConfig {}
	```
2. 编写测试类
	```
	ApplicationContext context = new AnnotationConfigApplicationContext(StringConfig.class);
	UserServiceImpl userService = context.getBean("userServiceImpl", UserServiceImpl.class);
	```


## 3.AOP
### 3.1 AOP概念
1. aop是面向切面（方面）编程，可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各个部分之间的耦合度降低。
2. 不通过修改源代码的方式添加新的功能
### 3.2AOP底层原理
1. aop底层使用动态代理
	1. 有接口的时候使用JDK动态代理，创建接口实现类的代理对象
	2. 没有接口的情况使用CGLIB动态代理，创建当前类子类的代理对象。增强类的方法
### 3.3JDK动态代理
1. 调用newProxyInstance方法
	newProxyInstance(==ClassLoader  loader==  ,  ==[类]<?>[] interfaces==   , ==invocationHandler h== )
	第一个参数，类加载器
	第二个参数，增强方法所在的类，这个类实现的接口，支持多个接口
	第三个参数，实现这个接口InvocationHandler，创建代理对象，写增强的方法
2. 动态代理的代码
	```
	1. 创建接口，定义方法
		UserDaoImpl实现UserDao中的add方法
	2. 创建接口实现类，实现方法
	 
	3. 使用InvocationHandler的继承类实现类的代理对象
		invoke方法中增强add方法
	4.使用proxy类
		UserDao userDao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),aa,invocationHandler);
	
	```
### 3.4JDK动态代理详细学习
Java动态代理类位于java.lang.reflect包下，一般主要涉及到以下两个类：
1. Interface InvocationHandler：该接口中仅定义了一个方法
`public object invoke(Object obj,Method method, Object[] args)`
第一个参数obj一般是指代理类，method是被代理的方法,args为该方法的参数数组
2. Proxy：该类即为动态代理类，其中主要包含以下内容：
	1. `protected Proxy(InvocationHandler h)`：构造函数，用于给内部的h赋值。
	2. `static Class getProxyClass (ClassLoaderloader, Class[] interfaces)`：获得一个代理类，其中loader是类装载器，interfaces是真实类所拥有的全部接口的数组
	3. `static Object newProxyInstance(ClassLoaderloader, Class[] interfaces, InvocationHandler h)`：返回代理类的一个实例，返回后的代理类可以当作被代理类使用(可使用被代理类的在Subject接口中声明过的方法)    

所谓DynamicProxy是这样一种class：它是在运行时生成的class，在生成它时你必须提供一组interface给它，然后该class就宣称它实现了这些 interface。你当然可以把该class的实例当作这些interface中的任何一个来用。当然，这个DynamicProxy其实就是一个Proxy，它不会替你作实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作。
### 3.5 AOP术语
1. **连接点**
类里面的哪些方法可以被增强，那么这些方法就被称为连接点
2. **切入点**
实际被增强的方法称为切入点
3. **通知（增强）**
	1. 实际被增强的逻辑部分称为通知
	2. 通知有多种类型
	3. 前置通知 @Before
	4. 后置通知 @AfterReturning
	5. 环绕通知 @Around
	6. 异常通知 @AfterThrowing	
	7. 最终通知 @After(==ProceedingJoinPoint p==) 再用p调用p.proceed();
4. **切面**
是动作，把通知应用到切入点的过程
### 3.6 AOP操作
需要的额外jar包
```
com.springsource.net.sf.cglib-2.2.0.jar
com.springsource.org.aopalliance-1.0.0.jar
com.springsource.org.aspectj.weaver-1.7.2.RELEASE.jar
```
#### 3.6.1准备
1. Spring一般是基于AspectJ实现AOP操作
2. AspectJ不是Spring的组成部分，独立的AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP
3. AspcetJ可以使用xml和注解方式实现
4. 在项目工程里面引入aop相关依赖
#### 3.6.2 切入点表达式
1. 切入点表达式作用：知道对哪个类的哪个方法进行增强
2. 语法结构：
	`execution([权限修饰符] [返回类型,可以省略] [类全路径][方法名称]([参数列表])`
	举例：
	```
	execution(* com.yq.dao.BookDao.add(..) 
	对BooDao中的add方法增强
	```
	```
	execution( * com.yq.dao.*.*(..) )
	这句话的意思是，对包里面的所有类的所有方法进行增强
	```
### 3.7 AOP操作（AspectJ注解）
代码实现
==注意Around的写法==
```
1.User类（被代理类），注解生成bean
	@Component(value = "user")
2.UserProxy类（代理类），添加@Aspect注解
	注意Around的写法
	@Component  
	@Aspect  
	public class UserProxy {  
	    @Before("execution(* com.yq.spring5.aopanno.User.add(..))")  
	    public void before(){  
	        System.out.println("这是一个before方法");  
	  }
	}
	     @Around("execution(* com.yq.spring5.aopanno.User.add(..))")  
	     public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
	         System.out.println("这是一个around方法,前");  
		     proceedingJoinPoint.proceed();  
	         System.out.println("这是一个around方法，后");
}
3.bean.xml中添加context和aop标签，开启注解扫描和aspectJ生成代理对象
	<context:component-scan base-package="com.yq.spring5.aopanno"></context:component-scan>
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
==公共切入点抽取：==
```
@Pointcut("execution(* com.yq.spring5.aopanno.User.add(..))")
public void pointCut(){}

@Before("pointCut()")  
public void before(){  
    System.out.println("这是一个before方法");  
}
```
==多个增强类的对同一个方法进行增强，设置增强类的优先级==
`order(数字）`数字值越小，优先级越高


### 3.8 AOP操作（AspectJ配置文件）
了解就行。

## 4. JDBCTemplate(JDBC模板）

### 4.1 概念和准备
   Spring框架对JDBC进行封装，使用JDBCTamplate方便对数据库进行操作

准备工作
1. 引入
```
durid.jar
mysql连接驱动
spring-jdbc-5.3.9.jar
spring-tx-5.3.9.jar
spring-orm-5.3.9.jar
```
2. 在spring配置文件中配置数据库连接池
```
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"  
  destroy-method="close">  
	 <property name="url" value="jdbc:mysql://127.0.0.1:13306:/JdbcDatabase"/>  
	 <property name="username" value="root"/>  
	 <property name="password" value="root"/>  
	 <property name="driverClassName" value="com.mysql.jdbc.Driver"/>  
</bean>
```
3. 配置JDBCtemplate对象 注入DataSource
```
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">  
 <property name="dataSource" ref="dataSource"/>  
</bean>
```
4. 创建service类和dao类，在dao类中注入JdbcTemplate对象，在Service中注入Dao

### 4.2 JdbcTemplate操作数据库
<center>完整代码：


1. 实体类略

2. BookDao
```
		public interface BookDao {  
		    void add(Book book);  
		 void updateBook(Book book);  
		 void deleteBook(int id);  
		}
```
3. BookDaoImpl
```
		@Repository  
		public class BookDaoImpl implements BookDao {  
		    @Autowired  
		  private JdbcTemplate jdbcTemplate;  
		  @Override  
		  public void add(Book book) {  
		        String sql = "insert into book values(?,?)";  
		 int update =  jdbcTemplate.update(sql,book.getId(),book.getBookname());  
		  System.out.println(update);  
		  }  
		    @Override  
		  public void updateBook(Book book) {  
		        String sql = "update book set bookname=? where id = ?";  
		 int update = jdbcTemplate.update(sql,book.getBookname(),book.getId());  
		  System.out.println("升级完成");  
		  }  
		    @Override  
		  public void deleteBook(int id) {  
		        String sql = "delete from book where id = ?";  
		 int update = jdbcTemplate.update(sql,id);  
		  System.out.println("删除成功");  
		  
		  }  
		}
```
4. Service
```
@Service  
public class BookService {  
  
    @Autowired  
  private BookDao bookDao;  
  
// 添加的方法  
  public void addBook(Book book){  
        bookDao.add(book);  
  }  
  
// 升级的方法  
  public void updateBook(Book book){  
        bookDao.updateBook(book);  
  }  
  
// 删除的方法  
  public void deleteBook(int id){  
        bookDao.deleteBook(id);  
  }   
}
```
5. 测试类
```
public class BookTest {  
  
    @Test  
  public void bookTest(){  
        Book book = new Book();  
  book.setId(1);  
  book.setBookname("yuqianUpdate");  
  ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");  
  BookService bookService = context.getBean("bookService", BookService.class);  
  //   bookService.addBook(book);  
 //bookService.updateBook(book);  bookService.deleteBook(1);  
  }  
}
```

### 4.3 JdbcTemplate操作数据库(查询）
1. 查询返回某个值 
`queryForObject(sql , 类.class)`
```
public int findCount() {  
     String sql = "select count(*) from book";  
     Integer integer = jdbcTemplate.queryForObject(sql, Integer.class);  
	 return integer;  
}
```

2. 查询返回对象 
`queryForObject(sql , BeanPropertyRowMapper , sql语句中的问号值)`
```
public Book findBook(int id) {  
    String sql = "select * from book where id = ?";  
    Book book = jdbcTemplate.queryForObject(sql,new BeanPropertyRowMapper<Book>(Book.class),id);  
    return book;  
}
```
3. 查询返回集合 
`query(sql , BeanPropertyRowMapper , sql语句中的问号值)`
```
public List<Book> findAllBook() {  
    String sql = "select * from book";  
    List<Book> books = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));  
    return books;  
}
```
### 4.4 JdbcTemplate操作数据库(批量添加）
用到的方法：`batchUpdate(sql , List<Object[]> batchArgs)`
daoImpl层代码
```
@Override  
public void batchAddBook(List<Object[]> batchArgs) {  
    String sql = "insert into book values(?,?)";  
    jdbcTemplate.batchUpdate(sql,batchArgs);  
}
```
==为什么是List<Object[]> batchArgs==
相当于是List中存了数组，批量添加的时候根据set方法把数组中的值添加
### 4.5 JdbcTemplate操作数据库(批量修改和删除）
用到的方法：`batchUpdate(sql，List<Object> batchArgs)`
详细代码：
```
批量修改
@Override  
public void batchUpdate(List<Object[]> batchArgs) {  
    String sql = "update book set bookname=? where id = ?";  
    jdbcTemplate.batchUpdate(sql,batchArgs);  
}  
  批量删除
@Override  
public void batchDelte(List<Object[]> batchArgs) {  
    String sql = "delete from book where id = ?";  
    jdbcTemplate.batchUpdate(sql,batchArgs);  
}
```

## 5. 事务概念
### 5.1 什么是事务
1. 事务是数据库操作基本概念，逻辑上一组操作，要么都成功，如果有一个失败，所有操作都失败
2. 事物的四个特性（ACID）
	1. 原子性 （不可分割）
	2. 一致性 （操作前后总量不变）
	3. 隔离性  （多事物操作不会相互影响）
	4. 持久性    （表中真正发生变化）
### 5.2 事务操作
1. 事务添加到Service层
2. Spring进行事务管理的操作
	1. 有两种方式：编程式和声明式
3. 声明式事务管理
	1. 基于注解的方式
	2. 基于xml文件的方式
4. 在Spring进行声明式事务管理，底层使用AOP原理

5. 事务管理API
>PlatformTransactionManager
>>AbstracPlatformTransanctionManger
>>>==DataSourceTransactionManager==(jdbcTemplate和Mybatis都是它)
>>>HibernateTransactionManager

6. 具体代码实现：

在spring配置文件中配置事务管理器
```
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
   <property name="dataSource" ref="dataSource"></property>  
</bean>
```
在spring配置文件中开启事务注解
```
1.引入名称空间tx
xmlns:aop="http://www.springframework.org/schema/aop"  
xmlns:tx="http://www.springframework.org/schema/tx"
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd  
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd"

2.开启事务注解
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

3.在Service类上面（或方法）上面添加事务注解
@Transactional  
public class UserService {}
如果添加到类上面，则这个类所有方法都添加事务，如果添加到方法上面，则只有这一个方法添加事务
```

### 5.3 声明式事务管理参数配置
Transactional参数配置
==`@Transactional(propagation = Propagation.REQUIRED)`==
1. propagation：事务传播行为
	多事务方法直接进行调用，这个过程中事务是如何进行管理的（即事务方法调用非事务方法或者非事务方法调用事务方法或者事务方法调用事务方法）

	Spring框架事务传播行为有7种
	| 名称 | 行为|
	| :-----------------: | :-----------------------: |
	|PROPAGATION_REQUIRED|如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。|
	|PROPAGATION_SUPPORTS|支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。|
	|PROPAGATION_MANDATORY|支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。|
	|PROPAGATION_REQUIRES_NEW | 创建新事务，无论当前存不存在事务，都创建新事务。|
	| PROPAGATION_NOT_SUPPORTED|以非事务方式执行操作，如果当前存在事务，就把当前事务挂起 |
	|PROPAGATION_NEVER |以非事务方式执行，如果当前存在事务，则抛出异常。 |
	|PROPAGATION_NESTED | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。|


2. isolation：事务隔离级别
==`@Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ)`==

事务的特性：隔离性，多事务操作之间不会产生影响
有三个问题：==脏读，不可重复读，虚（幻）读==

| 名称 | 描述 |
| :---------: | :---------: |
|脏读|一个未提交事务读取到另一个未提交事务的数据|
|不可重复读|一个未提交事务读取到另一提交事务修改数据|
|幻读|一个未提交事务读取到另一提交事务添加数据

通过设置事务隔离级别，解决问题


|  | 脏读 | 不可重复读 | 幻读 |
| :----------: | :----------: | :----------: | :-----------: |
|Read Uncommitted<br>读未提交|有|有|有|
|Read Committed<br>读已提交|无|有|有|
|Repeatable Read<br>可重复读|无|无|有|
|Serializable<br>串行化|无|无|无|


4. timeout：超时时间
事务需要在一定的时间提交，如果不提交进行回滚
spring默认-1 设置时间以秒为单位
5. readOnly：是否只读
readOnly默认值是false，设置readOnly的值是true，则只能查询

6. rollbackFor：回滚
设置出现了哪些异常进行事务回滚
7. noRollbackFor：不会滚
出现哪些异常不进行事务回滚
### 5.4 完全注解开发
config代码如下
```
@Configuration  
@ComponentScan(basePackages = "com.yq.spring5")  
@EnableTransactionManagement  
public class Txconfig {  
    @Bean  
  public DruidDataSource getDruidDataSource(){  
        DruidDataSource dataSource = new DruidDataSource();  
  dataSource.setUrl("jdbc:mysql://127.0.0.1:13306/JdbcDatabase");  
  dataSource.setUsername("root");  
  dataSource.setPassword("root");  
  dataSource.setDriverClassName("com.mysql.jdbc.Driver");  
 return dataSource;  
  }  
    @Bean  
  public JdbcTemplate getJdbcTemplate(DruidDataSource dataSource){  
        JdbcTemplate jdbcTemplate = new JdbcTemplate();  
  jdbcTemplate.setDataSource(dataSource);  
 return jdbcTemplate;  
  }  
    @Bean  
  public DataSourceTransactionManager dataSourceTransactionManager(DruidDataSource dataSource){  
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();  
  dataSourceTransactionManager.setDataSource(dataSource);  
 return dataSourceTransactionManager;  
  }  
}
```
