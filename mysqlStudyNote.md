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

	

