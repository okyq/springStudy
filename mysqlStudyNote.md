 , # <center>mysql学习笔记
### 1. 关系型数据库RDBMS和非关系型数据库非RDBMS
 1. 关系型数据库以行列来存储数据，表和表之间存在关系，可以用SQL语言方便在一个表及多个表之间做非常复杂的数据查询
 2. 非关系型数据库基于键值对存储数据，性能非常高
### 2. 表的四种关联关系：一对一，一对多，多对多，自我引用
### 3. mysql5.7默认字符集是拉丁
### 4. sql语言分为三类
 3. DDL：数据定义语言。
create ; alter : drop ; rename ; 
 4. DML：数据操作语言
insert ; delete ; update; select 
 5. DCL：数据控制语言 
commit rollback savepoint grant revoke
### 5.mysql语法
1. 列的别名的三种方式
	+ a as b
	+ a b
	+ a 'b'
2. 去除重复行
	 + distinct 
3. 空值参与运算
	+ null不同于0，' ' , 'null'
	+ 空值参与运算结果也是null
	+ 可以引用IFNULL( para , 0)
4. 着重号`  把关键字引用起来
5. 显示表的结构：DESCRIBE table  显示字段的详细信息
6. 比较运算符 = <=>（安全等于） != < <= > >= 为真1 为假0  安全等于<=>和等于的作用是一样的，只是安全等于可以用来比较null
7.  in（a，b，c）
	+ select * from stu where salary in (600,700,800)
8. like
	+ select * from stu where last_name like '%a%'
	+ ```_```代表一个不确定的字符
		+ 例如第二个字符是a '_a%'
9. REGEXP
	+ ``` select 'yuqian' REGEXP '^y' , 'yuqian' REGEXP ' n$' , 'yuqian' REGEXP 'qi'```   ^以什么开头，$以什么结尾
10. order by asc 升序 desc 降序
	+ 可以实用列的别名进行排序，列的别名只能在order by 使用，不能在where中使用 
	+ 强调格式，where在from后，order by之前
11. 分页操作 limit 只能放在查询语句==最后== 
	+ ``` select * from stu limit  start,pageSize```
	+ 每页显示pageSize条记录，此时显示pageNo页
	 ``` limit (pageNo - 1) * pageSize , pageSize```
	 + 假如只想显示最上面的一条查询记录
	 ```limit 0,1```
### 6. 多表查询
#### 6.1多表查询语法
1. ```select a , b from A,B```这种sql语句会出现笛卡尔积（交叉连接）错误 。
原因：缺少了多表连接的条件
2. ``` select A.a , B.b from A , B where A.a = B.b```建议：从sql优化的角度，建议多表查询的时候，每个字段前都指明其所在的表
3. 可以给表起别名，在select 和where 中使用表的别名 ```select emp.a , dept.b from A emp , B dept where emp.a = dept.b```
4. 如果给表起了别名，在select 和where中使用了别名，就必须使用别名，不能使用表名，==因为sql语句先执行from，别名会覆盖原来的表名==
5. 如果有n个表实现多表查询，则需要至少n-1个连接条件
#### 6.2 多表查询分类
1. 等值连接 vs 非等值连接
2. 自连接 vs 非自连接
	+ 自连接的例子：
	```select A.a , B.b from ex A , ex B where A.a = B . c```
3. 内连接 vs 外连接
	+ 内连接： 结果集中不包含一个表与另一个表不匹配的行
	+ 外连接分类：左外，右外，满外
### 7. 函数
#### 7.1 流程控制函数
1. if(value , value1 , value2) 如果value为true则返回value1，否则返回value2
2. ifnull（value1，value2） 如果value1不为null则返回value1，否则返回value2
3. 
	```
	select *  case id when 10 then.... 
					when 20 then...
					when 30 then ...
					from stu
	```
4. to_days(curdate())-to_days(exp)
#### 7.2 加密解密函数
1. md5,sha 不可逆
2. encode() 和 decode() 在mysql8.0弃用
### 8 聚合函数
#### 8.1 常见的聚合函数
1. avg/sum
2. max/min
3. count 个数 不计算null值
#### 8.2 group by 的使用
1. ==每一组是一列==
2. select 中出现的非主函数的字段必须申明在group by中，反之group by 中声明的字段可以不出现在select中
3. group by 申明在from后面，where后面，order by前面，limit前面
4. group by 中 使用 with rollup会在最后计算平均值
#### 8.3 having 的使用
1. 用来过滤数据的
2. 如果过滤条件中使用了聚合函数，必须使用having来替换where，否则报错
3. 当过滤条件中没有聚合函数时，则此过滤条件申明在where中或者having中都可以，建议申明在where中，效率更高
4. having必须申明在group by后面
5. 开发中使用having的前提是sql中使用了group by
6. 使用了having还是可以使用where
7. where与having的对比
	| 函数  | 优点| 缺点 |
	| :----: | :----: | :----: |
	| where | 先筛选数据再关联，执行效率高| 不能使用分组中的计算函数进行筛选
	| having | 可以使用分组中的计算函数 | 在最后的结果集中进行筛选，执行效率低 |
	

#### 8.4 sql底层执行原理
1. select 语句的完整结构
	+  sql92
		```
		select ... , ... , ...(存在聚合函数)
		from ... , ... , ...
		where 多表连接条件 and 不包含聚合函数的过滤条件
		group by ...
		having 包含聚合函数的过滤条件
		order by ... , ... (asc | desc)
		limit ... , ...
		```
	+ sql99
		```
		select ... , ... , ...(存在聚合函数)
		from ...join ...on 多表连接的条件
		where 不包含聚合函数的过滤条件
		group by ...
		having 包含聚合函数的过滤条件
		order by ... , ... (asc | desc)
		limit ... , ...
		```
2. sql语句的执行过程
	1. from... -> on ->(left/right  join ) -> where  ->group by -> having -> select -> distinct -> order by -> limit
### 9子查询
#### 9.1 子查询分类
1. 从内查询返回的结果的条目数分为
	+ 单行子查询     case when then end 也可使用单行子查询
	+ 多行子查询
		|        操作符| 含义 |
		| :--: | :--: |
		|   in| 等于列表中的任意一个  |
		|   any | 需要和单行比较操作符一起使用，和子查询返回的某一个值比较|
		|   all| 需要和单行比较操作符一起使用，和子查询返回的所有值比较 |
		|   some| 实际上是any的别名，一般使用any|

2. 内查询是否被执行多次
	+ 相关子查询（内查询执行多次，有先相关性）
		+ 查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id
		+ 方式1：使用相关子查询
		 ```
		SELECT
		last_name,salary,department_id
		FROM employees e1
		WHERE salary > (
		SELECT AVG(salary)
		FROM employees e2
		WHERE department_id =
		e1.`department_id`
		);
		 ```
		+ 方式2：在from中声明子查询
			```
			SELECT e.last_name,e.salary,e.department_id
			FROM employees e,(
				SELECT department_id,AVG(salary) avg_sal
				FROM employees
				GROUP BY department_id) t_dept_avg_sal
			WHERE e.department_id = t_dept_avg_sal.department_id
			AND e.salary > t_dept_avg_sal.avg_sal
			```
		+  ==在SELECT中，除了GROUP BY 和 LIMIT之外，其他位置都可以声明子查询！==
	+ 不相关子查询（内查询只执行一次，没有相关性）
### 10表
#### 10.1 创建表的两种方式
+ 直接创建新表
	``` create table yq (.....)```
+ 基于现有的表，同时导入数据
	``` 
	create table yq 
	as
	select .. , .. , ..
	from ..
	```
+ 如果不想导入数据，后面加一个 ```where 1=2```
#### 10.2 修改表
| 描述 | 命令 |
| :---: | :---: |
添加一个字段 | ```alter table test1 add name varchar(20) first ```
|修改一个字段：数据类型，长度，默认值|``` alter table test modify name varchar(10) default 'aaa' ```|
|重命名一个字段| ``` alter table test change old_name new_name varchar(20) ```|
|删除一个字段| ``` alter table test drop column name ```|
|重命名表 | ``` rename table old_table_name to new_table_name ```|
|删除表 | ``` drop table if exists test ```|
|清空表，保留表结构| ``` truncate table test ```|
#### 10.3 DCL中的commit和rollback
+ commit提交，一旦执行commit，则数据就被永久保存在了数据库中，意味着不可以回滚
+ 

	

```mysql
经典四表联查
1.建立一个数据库
create database work;

2.进入数据库work
use work;

3.数据库默认编码可能不支持中文,可以在这里设置下
set names gbk;

4.建立student表
属性有:编号：id (主键，自动增长),姓名:sname,出生年月:sage,性别:ssex(枚举)
create table student(sid int primary key auto_increment,
sname varchar(20),
sage date,
ssex enum('男','女'));

5.第二个课程表中使用了外键教师标号，因而需要先建立教师表
create table teacher(tid int primary key auto_increment,
tname varchar(20));

6.建立课程表
create table course(cid int primary key auto_increment,
cname varchar(20),
tid int,
foreign key(tid) references teacher(tid));

7.建立成绩表
create table sc(sid int,
cid int,
score int);

8.show tables; //可查看建立的四个表格

9.插入数据，因为里面有主键链接，表格插入数据也要有顺序(注意题目图片上都是字节引号，应该为int，不要单引号)

a，先给student表插入数据
insert into student values(1,'赵雷','1990-01-01','男'),
    (2,'钱电','1990-12-21','男'),
    (3,'孙风','1990-05-20','男'),
    (4,'李云','1990-08-06','男'),
    (5,'周梅','1991-12-01','女'),
    (6,'吴兰','1992-03-01','女'),
    (7,'郑竹','1989-07-01','女'),
    (8,'王菊','1990-01-20','女');
 
b, 给teacher表插入数据,这里不可以先给course表插入数据,因为course表外链接到teacher的主键
insert into teacher values(1,'张三'),
        (2,'李四'),
        (3,'王五');
 
c, 给course表插入数据
insert into course values(1,'语文',2),
            (2,'语文',1),
            (3,'语文',3);
 
d, 最后给sc表插入数据(题目图片少了第1个学生成绩，在这加上 1,1,90;  1,2,80;  1,3,90)
insert into sc values(1,1,90),
            (1,2,80),
            (1,3,90),
            (2,1,70),
            (2,2,60),
            (2,3,80),
            (3,1,80),
            (3,2,80),
            (3,3,80),
            (4,1,50),
            (4,2,30),
            (4,3,20),
            (5,1,76),
            (5,2,87),
            (6,1,31),
            (6,3,34),
            (7,2,89),
            (7,3,98);
						
	1、查询”01”课程比”02”课程成绩高的学生的信息及课程分数
select s.sid,s.sname,s.sage,s.ssex,sc1.score,sc2.score from student s,sc sc1,sc sc2 where sc1.cid=1 and sc2.cid=2 and sc1.score>sc2.score and sc1.sid=sc2.sid and s.sid=sc1.sid;

2、查询同时存在”01”课程和”02”课程的情况
select * from course c1,course c2;

3.查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
select s.sid,s.sname,avg(score) from student s,sc group by s.sid having avg(score)>60;

4、查询名字中含有”风”字的学生信息
select * from student where sname like ‘%风%’;

5、查询课程名称为”数学”，且分数低于60的学生姓名和分数
select s.sname,score from student s,sc where s.sid=sc.sid and cid=2 and score<60;

6、查询所有学生的课程及分数情况；
select cname,score from sc,course where sc.cid=course.cid;

7、查询没学过”张三”老师授课的同学的信息
select s.* from student s where s.sid not in(select sc1.sid from sc sc1,course c,teacher t where t.tid=c.tid and sc1.cid=c.cid and t.tname=’张三’);

8.查询学过”张三”老师授课的同学的信息
select s.* from student s ,sc sc1,course c,teacher t where s.sid=sc1.sid and sc1.cid=c.cid and c.tid=t.tid and t.tname=’张三’;

9、查询学过编号为”01”并且也学过编号为”02”的课程的同学的信息
student(sid) sc(sid cid tid) sc2(sid cid tid) course(cid tid cname)
select s.* from student s,sc sc1,sc sc2 where s.sid=sc1.sid and sc1.sid=sc2.sid and sc1.cid=1 and sc2.cid=2;

10、查询学过编号为”01”但是没有学过编号为”02”的课程的同学的信息
select distinct s.* from student s,sc sc1,sc sc2,sc sc3 where s.sid=sc1.sid and sc1.sid=sc2.sid and sc1.cid=1 and sc2.cid!=2;

11、查询没有学全所有课程的同学的信息
select s.* from student s where s.sid not in(select sc1.sid from sc sc1,sc sc2,sc sc3 where sc1.cid=1 and sc2.cid=2 and sc3.cid =3 and sc1.sid=sc2.sid and sc1.sid=sc3.sid) group by s.sid;

12、查询至少有一门课与学号为”01”的同学所学相同的同学的信息
select distinct s.* from student s,sc sc1 where s.sid=sc1.sid and sc1.cid in(select cid from sc where sid=1) and s.sid<> 1;

13、查询和”01”号的同学学习的课程完全相同的其他同学的信息
select s.* from student s where s.sid in(select distinct sc.sid from sc where sid<>1 and sc.cid in(select distinct cid from sc where sid=1)group by sc.sid having count(1)=(select count(1) from sc where s.sid=1));

14、查询没学过”张三”老师讲授的任一门课程的学生姓名
select s.* from student s where s.sid not in(select sc1.sid from sc sc1,course c,teacher t where sc1.cid=c.cid and c.tid=t.tid and t.tname=’张三’);

15、查询出只有两门课程的全部学生的学号和姓名
select s.* from student s,sc group by sc.sid having count(sc.sid)=2 and s.sid=sc.sid;

16、查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime)
select * from student where sage>=’1900-01-01’ and sage<=’1900-12-31’;
select s.* from student s where s.sage like ‘1900-%’;（方法2）

17、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
select sc.cid,avg(score) from sc group by sc.cid order by avg(score) DESC , sc.cid;

18、查询任何一门课程成绩在70分以上的姓名、课程名称和分数；
select s.sname,c.cname,score from student s,sc,course c where s.sid=sc.sid and sc.cid=c.cid and score>70;

19、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩
select s.sname,avg(score) from sc,student s where s.sid=sc.sid group by sc.sid having avg(score)>=85;

20、查询不及格的课程
select s.sname,c.cname,score from student s,sc,course c where s.sid=sc.sid and sc.cid=c.cid and score<60;

21、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名；
select s.sid,s.sname from student s,sc where sc.sid=s.sid and sc.cid=1 and score>80;

22、求每门课程的学生人数
select cid,count(sid) from sc group by sc.cid;

23、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
select cid,count(sid) from sc group by cid having count(sid)>5 order by count(sid),cid ASC;

24、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
select s1.sid,s2.sid,sc1.cid,sc1.score,sc2.score from student s1,student s2,sc sc1,sc sc2 where s1.sid!=s2.sid and s1.sid=sc1.sid and s2.sid=sc2.sid and sc1.cid!=sc2.cid and sc1.score=sc2.score;

25、检索至少选修两门课程的学生学号
select sid from sc group by sid having count(cid)>=2;

26、查询选修了全部课程的学生信息
select s.* from sc,student s where s.sid=sc.sid group by sid having count
(cid)=3;

27、查询各学生的年龄
select s.sname,(TO_DAYS(‘2017-09-07’)-TO_DAYS(s.sage))/365 as age from student s;

28、查询本月过生日的学生
select s.sname from student s where s.sage like ‘_____07%’;

29、查询下月过生日的学生
select s.sname from student s where s.sage like ‘_____08%’;

30、查询学全所有课程的同学的信息
select s.* from student s,sc sc1,sc sc2,sc sc3 where sc1.cid=1 and sc2.cid=2 and sc3.cid=3 and sc1.sid=sc2.sid and sc1.sid=sc3.cid and s.sid =sc1.sid group by s.sid;
						

```

