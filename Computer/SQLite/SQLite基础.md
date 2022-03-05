<h1 align="center">SQLite基础</h1>

[toc]

## 01数据库概念

```sql
* A: 什么是数据库
		数据库就是存储数据的仓库，其本质是一个文件系统，数据按照特定的格式将数据存储起来，用户可以对数据库中的数据进行增加，修改，删除及查询操作。
* B: 什么是数据库管理系统
		数据库管理系统（DataBase Management System，DBMS）：指一种操作和管理数据库的大型软件，用于建立、使用和维护数据库，
		对数据库进行统一管理和控制，以保证数据库的安全性和完整性。用户通过数据库管理系统访问数据库中表内的数据。
```

## 02常见的数据库

```sql
* A: 常见的数据库
		MYSQL	：开源免费的数据库，小型的数据库.已经被Oracle收购了.MySQL6.x版本也开始收费。
		Oracle	：收费的大型数据库，Oracle公司的产品。Oracle收购SUN公司，收购MYSQL。
		DB2		：IBM公司的数据库产品,收费的。常应用在银行系统中.
		SQLServer：MicroSoft 公司收费的中型的数据库。C#、.net等语言常使用。
		SyBase	：已经淡出历史舞台。提供了一个非常专业数据建模的工具PowerDesigner。
		SQLite	: 嵌入式的小型数据库，应用在手机端。
		Java相关的数据库：MYSQL，Oracle．
		这里使用MySQL数据库。MySQL中可以有多个数据库，数据库是真正存储数据的地方
```

## 03数据库和管理系统

```sql
* A: 数据库管理系统
		----数据库1
			----数据表1a
			----数据表1b
		----数据库2
			-----数据表2a
			-----数据表2b
```

## 04数据表和Java中类的对应关系

```sql
* A:数据库中以表为组织单位存储数据。
	表类似我们的Java类，每个字段都有对应的数据类型。
	那么用我们熟悉的java程序来与关系型数据对比，就会发现以下对应关系。
		类----------表
		类中属性----------表中字段
		对象----------记录
```

## 05数据表和Java中类的对应关系用户表举例

```sql
* A:举例:
账务表
id		name		age	
1		lisi		23
2		wang		24

每一条记录对应一个User的对象
[user1  id = 1 name = lisi  age = 23]
[user2	id = 2 name = wang	age = 24]
```

## 06MySQL数据库安装

```sql
A: 安装步骤参见 day28_source《MySQL安装图解.doc》
B: 安装后，MySQL会以windows服务的方式为我们提供数据存储功能。开启和关闭服务的操作：右键点击我的电脑→管理→服务→可以找到MySQL服务开启或停止。
```

## 07数据库在系统服务

```sql
* A：开启服务和关闭服务
方式1: 我的电脑-----> (右键)管理---->服务和应用程序---->服务----找到MySQL服务右键启动或关闭 
方式2: 进入dos窗口 使用命令: net start mysql 开启MySQL服务;  命令:net stop mysql 关闭MySql服务
```

## 08MySQL的登录

```sql
* A: MySQL是一个需要账户名密码登录的数据库，登陆后使用，它提供了一个默认的root账号，使用安装时设置的密码即可登录。
	格式1：cmd>  mysql –u用户名 –p密码
	例如：mysql -uroot –proot
	
	格式2：cmd>  mysql --host=ip地址 --user=用户名 --password=密码
	例如：mysql --host=127.0.0.1  --user=root --password=root
```

## 10SQL语句介绍和分类

```sql
* A:SQL介绍
	* 前面学习了接口的代码体现，现在来学习接口的思想，接下里从生活中的例子进行说明。
	* 举例：我们都知道电脑上留有很多个插口，而这些插口可以插入相应的设备，这些设备为什么能插在上面呢？
	* 主要原因是这些设备在生产的时候符合了这个插口的使用规则，否则将无法插入接口中，更无法使用。发现这个插口的出现让我们使用更多的设备。

* B: SQL分类	
	* 数据定义语言：简称DDL(Data Definition Language)，用来定义数据库对象：数据库，表，列等。关键字：create，alter，drop等 
	* 数据操作语言：简称DML(Data Manipulation Language)，用来对数据库中表的记录进行更新。关键字：insert，delete，update等
	* 数据控制语言：简称DCL(Data Control Language)，用来定义数据库的访问权限和安全级别，及创建用户。
	* 数据查询语言：简称DQL(Data Query Language)，用来查询数据库中表的记录。关键字：select，from，where等

* C: SQL通用语法
	SQL语句可以单行或多行书写，以分号结尾
		可使用空格和缩进来增强语句的可读性
		MySQL数据库的SQL语句不区分大小写，建议使用大写，例如：SELECT * FROM user。
		同样可以使用/**/的方式完成注释
```

## 11数据表中的数据类型

```sql
* A:MySQL中的我们常使用的数据类型如下	 
	详细的数据类型如下(不建议详细阅读！)
	分类	类型名称 	说明 
	整数类型	tinyInt	很小的整数
		smallint	小的整数
		mediumint	中等大小的整数
		int(integer)	普通大小的整数
	小数类型	float	单精度浮点数
		double	双精度浮点数
		decimal（m,d）	压缩严格的定点数
	日期类型	year	YYYY  1901~2155
		time	HH:MM:SS  -838:59:59~838:59:59
		date	YYYY-MM-DD 1000-01-01~9999-12-3
		datetime 	YYYY-MM-DD HH:MM:SS 1000-01-01 00:00:00~ 9999-12-31 23:59:59
		timestamp	YYYY-MM-DD HH:MM:SS  1970~01~01 00:00:01 UTC~2038-01-19 03:14:07UTC
	文本、二进制类型	CHAR(M)			M为0~255之间的整数
		VARCHAR(M)		M为0~65535之间的整数
		TINYBLOB	允许长度0~255字节
		BLOB		允许长度0~65535字节
		MEDIUMBLOB	允许长度0~167772150字节
		LONGBLOB	允许长度0~4294967295字节
		TINYTEXT	允许长度0~255字节
		TEXT		允许长度0~65535字节
		MEDIUMTEXT	允许长度0~167772150字节
		LONGTEXT	允许长度0~4294967295字节
		VARBINARY(M)允许长度0~M个字节的变长字节字符串
		BINARY(M)	允许长度0~M个字节的定长字节字符串
```

## 12创建数据库操作

```sql
* A: 创建数据库
	格式:
		* create database 数据库名;
		* create database 数据库名 character set 字符集;
	例如：
	#创建数据库 数据库中数据的编码采用的是安装数据库时指定的默认编码 utf8
	CREATE DATABASE day21_1; 
	#创建数据库 并指定数据库中数据的编码
	CREATE DATABASE day21_2 CHARACTER SET utf8;
	
* B: 查看数据库
	查看数据库MySQL服务器中的所有的数据库:
	show databases;
	查看某个数据库的定义的信息:
	show create database 数据库名;
	例如：
	show create database day21_1;

* C: 删除数据库
	drop database 数据库名称;
	例如：
	drop database day21_2;
	
* D: 其他的数据库操作命令
	切换数据库：
	use 数据库名;
	例如：
	use day21_1;
	
* E: 查看正在使用的数据库:
	select database();
```

## 13创建数据表格式

```sql
* A:格式：
	create table 表名(
	   字段名 类型(长度) 约束,
	   字段名 类型(长度) 约束
	);
	例如：
	###创建分类表
	CREATE TABLE sort (
	  sid INT, #分类ID 
	  sname VARCHAR(100) #分类名称
	);
```

## 14约束

```sql
* A: 约束的作用:
	create table 表名(
		   列名 类型(长度) 约束,
		   列名 类型(长度) 约束
		);
		限制每一列能写什么数据,不能写什么数据。
	
* B: 哪些约束:
		主键约束
		非空约束（NOT NULL）
		唯一约束
		外键约束
		默认值约束（DEFAULT 0）
```

## 15SQL代码的保存

```sql
* A: 当sql语句执行了，就已经对数据库进行操作了，一般不用保存操作
	在SQLyog 中Ctrl + S 保存的是写sql语句。
```

## 16创建用户表

```sql
* A: 创建用户表:
	需求:创建用户表,用户编号,姓名,用户的地址
	
	* B: SQL语句
	CREAT TABLE users (
		uid INT,
		uname VARCHAR(20),
		uaddress VARCHAR(200)
	);
```

## 17主键约束

```sql
* A: 主键是用于标识当前记录的字段。它的特点是非空，唯一。
	在开发中一般情况下主键是不具备任何含义，只是用于标识当前记录。
* B: 格式：
	1.在创建表时创建主键，在字段后面加上  primary key.
	create table tablename(	
	id int primary key,
	.......
	)
	
	2. 在创建表时创建主键，在表创建的最后来指定主键	
	create table tablename(						
	id int，
	.......，
	primary key(id)
	)
	
	3.删除主键：alter table 表名 drop primary key;
	alter table sort drop primary key;
	
	4.主键自动增长：一般主键是自增长的字段，不需要指定。
	实现添加自增长语句,主键字段后加auto_increment(只适用MySQL)
```

## 18常见表的操作

```sql
* A: 查看数据库中的所有表：
	格式：show tables;
		查看表结构：
	格式：desc 表名;
	例如：desc sort;

* B: 格式：drop table 表名;
	例如：drop table sort;
```

## 19修改表结构

```sql
 * A: 修改表添加列
	alter table 表名 add 列名 类型(长度) 约束;	
	例如：
	#1，为分类表添加一个新的字段为 分类描述 varchar(20)
	ALTER TABLE sort ADD sdesc VARCHAR(20);

* B: 修改表修改列的类型长度及约束
	alter table 表名 modify 列名 类型(长度) 约束; 
	例如：
	#2, 为分类表的分类名称字段进行修改，类型varchar(50) 添加约束 not null
	ALTER TABLE sort MODIFY sname VARCHAR(50) NOT NULL;

* C: 修改表修改列名
	alter table 表名 change 旧列名 新列名 类型(长度) 约束; 
	例如：
	#3, 为分类表的分类名称字段进行更换 更换为 snamesname varchar(30)
	ALTER TABLE sort CHANGE sname snamename VARCHAR(30);

* D: 修改表删除列
	alter table 表名 drop 列名;	
	例如：
	#4, 删除分类表中snamename这列
	ALTER TABLE sort DROP snamename;

* E: 修改表名
	rename table 表名 to 新表名; 
	例如：
	#5, 为分类表sort 改名成 category
	RENAME TABLE sort TO category;

* F: 修改表的字符集
	salter table 表名 character set 字符集;
	例如：
	#6, 为分类表 category 的编码表进行修改，修改成 gbk
	ALTER TABLE category CHARACTER SET gbk;
```

## 20数据表添加数据_1

```sql
 * A: 语法：
	insert into 表 (列名1,列名2,列名3..) values  (值1,值2,值3..); -- 向表中插入某些列

 * 举例:
	INSERT INTO product (id,pname,price) VALUES (1,'笔记本',5555.99);
	INSERT INTO product (id,pname,price) VALUES (2,'智能手机',9999);
 * 注意:
	列表,表名问题
	对应问题,个数,数据类型
```

## 21数据表添加数据_2

```sql
 * A: 添加数据格式,不考虑主键
	insert into 表名 (列名) values (值)
 * 举例:
	INSERT INTO product (pname,price) VALUE('洗衣机',800);
	
	
 * B: 添加数据格式,所有值全给出
	格式
	insert into 表名 values (值1,值2,值3..); --向表中插入所有列
	INSERT INOT product VALUES (4,'微波炉',300.25)
 * C: 添加数据格式,批量写入
	格式:
	insert into 表名 (列名1,列名2,列名3) values (值1,值2,值3),(值1,值2,值3)
 举例:
	INSERT INTO product (pname,price) VALUES
	('智能机器人',25999.22),
	('彩色电视',1250.36),
	('沙发',58899.02)
```

## 22更新数据

```sql
* A: 用来修改指定条件的数据，将满足条件的记录指定列修改为指定值
	语法：
	update 表名 set 字段名=值,字段名=值;
	update 表名 set 字段名=值,字段名=值 where 条件;
* B: 注意：
		列名的类型与修改的值要一致.
		修改值得时候不能超过最大长度.
		值如果是字符串或者日期需要加’’.

* C: 例如：
	#1，将指定的sname字段中的值 修改成 日用品
	UPDATE sort SET sname='日用品';
	#2, 将sid为s002的记录中的sname改成 日用品
	UPDATE sort SET sname='日用品' WHERE sid='s002';
	UPDATE sort SET sname='日用品' WHERE sid='s003';
```

## 23删除数据

```sql
* A: 语法：
	delete from 表名 [where 条件];
	或者
	truncate table 表名;

 * B: 面试题：
	删除表中所有记录使用delete from 表名; 还是用truncate table 表名;
	删除方式：delete 一条一条删除，不清空auto_increment记录数。
	truncate 直接将表删除，重新建表，auto_increment将置为零，从新开始。

 * C: 例如：
	DELETE FROM sort WHERE sname='日用品';
	#表数据清空
	DELETE FROM sort;
```

## 24命令行乱码问题

```sql
A: 问题
	我们在dos命令行操作中文时，会报错
	insert into user(username,password) values(‘张三’,’123’);		
	ERROR 1366 (HY000): Incorrect string value: '\xD5\xC5\xC8\xFD' for column 'username' at row 1
B: 原因:因为mysql的客户端编码的问题我们的是utf8,而系统的cmd窗口编码是gbk
	解决方案（临时解决方案）:修改mysql客户端编码。
	show variables like 'character%'; 查看所有mysql的编码
	client connetion result 和客户端相关
	database server system 和服务器端相关 
	将客户端编码修改为gbk.
	set character_set_results=gbk; / set names gbk;
	以上操作，只针对当前窗口有效果，如果关闭了服务器便失效。如果想要永久修改，通过以下方式:
	在mysql安装目录下有my.ini文件
	default-character-set=gbk 客户端编码设置						
	character-set-server=utf8 服务器端编码设置
	注意:修改完成配置文件，重启服务
```

## 25数据表和测试数据准备

```sql
* A: 查询语句，在开发中使用的次数最多，此处使用“zhangwu” 账务表。
	创建账务表：
	CREATE TABLE zhangwu (
	  id INT PRIMARY KEY AUTO_INCREMENT, -- 账务ID
	  name VARCHAR(200), -- 账务名称
	  money DOUBLE, -- 金额
	);
* B: 插入表记录：
	INSERT  INTO zhangwu(id,name,money) VALUES (1,'吃饭支出',247);
	INSERT  INTO zhangwu(id,name,money) VALUES (2,'工资收入',12345);
	INSERT  INTO zhangwu(id,name,money) VALUES (3,'服装支出',1000);
	INSERT  INTO zhangwu(id,name,money) VALUES (4,'吃饭支出',325);
	INSERT  INTO zhangwu(id,name,money) VALUES (5,'股票收入',8000);
	INSERT  INTO zhangwu(id,name,money) VALUES (6,'打麻将支出',8000);
	INSERT  INTO zhangwu(id,name,money) VALUES (7,null,5000);
```

## 26数据的基本查询

```sql
	* A: 查询指定字段信息
		select 字段1,字段2,...from 表名;
		例如：
		select id,name from zhangwu;

	* B: 查询表中所有字段
		select * from 表名; 
		例如：
		select * from zhangwu; 
		注意:使用"*"在练习、学习过程中可以使用，在实际开发中，不推荐使用。原因，要查询的字段信息不明确，若字段数量很多，会导致查询速度很慢。

	* C: distinct用于去除重复记录
		select distinct 字段 from 表名;			
		例如：
		select distinct money from zhangwu;

	* D: 别名查询，使用的as关键字，as可以省略的.
		别名可以给表中的字段，表设置别名。 当查询语句复杂时，使用别名可以极大的简便操作。
		表别名格式: 
		select * from 表名 as 别名;
		或
		select * from 表名 别名;
		列别名格式：
		select 字段名 as 别名 from 表名;
		或
		select 字段名 别名 from 表名;
		例如
		表别名：
			select * from zhangwu as zw;
		列别名：
			select money as m from zhangwu;
			或
			select money m from zhangwu;

		我们在sql语句的操作中，可以直接对列进行运算。
		例如：将所有账务的金额+10000元进行显示.
		select pname,price+10000 from product;
```

## 27数据的条件查询_1

```sql
 * A:条件查询
		where语句表条件过滤。满足条件操作，不满足不操作，多用于数据的查询与修改。
	
 * B : 格式 :
		select 字段  from 表名  where 条件;	
 
 * C: while条件的种类如下：
	比较运算符	
		>  <  <=   >=   =  <>	---------- 大于、小于、大于(小于)等于、不等于
		BETWEEN  ...AND...      -----------	显示在某一区间的值(含头含尾)
		IN(set) 	            -----------显示在in列表中的值，例：in(100,200)
		LIKE 通配符	   			-----------模糊查询，Like语句中有两个通配符：
											% 用来匹配多个字符；例first_name like ‘a%’;
											_ 用来匹配一个字符。例first_name like ‘a_’;
		IS NULL 	判断是否为空
								------------is null; 判断为空
											is not null; 判断不为空
 * D 逻辑运算符	
		and	                    ------------ 多个条件同时成立
		or						------------ 多个条件任一成立
		not						------------ 不成立，例：where not(salary>100);

 * E: 例如：
	查询所有吃饭支出记录
	SELECT * FROM zhangwu WHERE name = '吃饭支出';

	查询出金额大于1000的信息
	SELECT * FROM zhangwu WHERE money >1000;

	查询出金额在2000-5000之间的账务信息
	SELECT * FROM zhangwu WHERE money >=2000 AND money <=5000;
	或
	SELECT * FROM zhangwu WHERE money BETWEEN 2000 AND 5000;

	查询出金额是1000或5000或3500的商品信息
	SELECT * FROM zhangwu WHERE money =1000 OR money =5000 OR money =3500;
	或
	SELECT * FROM zhangwu WHERE money IN(1000,5000,3500);
```

## 28数据的条件查询_2

```sql
 * A 模糊查询
	查询出账务名称包含”支出”的账务信息。
	SELECT * FROM zhangwu WHERE name LIKE "%支出%";

 * B 查询出账务名称中是五个字的账务信息
	SELECT * FROM gjp_ledger WHERE ldesc LIKE "_____"; -- 五个下划线_

* C 查询出账务名称不为null账务信息
	SELECT * FROM zhangwu WHERE name IS NOT NULL;
	SELECT * FROM zhangwu WHERE NOT (name IS NULL);
	
* D 查询出账务名称包含“支出”的账务信息中的最多3条
	SELECT * FROM zhangwu WHERE name LIKE "%支出%" LIMIT 3;
```

## 29排序查询

```sql
* A: 排序查询
	 使用格式
		* 通过order by语句，可以将查询出的结果进行排序。放置在select语句的最后。
		* SELECT * FROM 表名 ORDER BY 字段ASC;
			* ASC 升序 (默认)
			* DESC 降序
		
* B: 案例代码
		/*
		  查询,对结果集进行排序
		  升序,降序,对指定列排序
		  order by 列名 [desc][asc]
		  desc 降序
		  asc  升序排列,可以不写
		*/
		-- 查询账务表,价格进行升序
		SELECT * FROM zhangwu ORDER BY zmoney ASC

		-- 查询账务表,价格进行降序
		SELECT * FROM zhangwu ORDER BY zmoney DESC

		-- 查询账务表,查询所有的支出,对金额降序排列
		-- 先过滤条件 where 查询的结果再排序
		SELECT * FROM zhangwu WHERE zname LIKE'%支出%' ORDER BY zmoney DESC
		
		-- 查询账务表,查询所有的支出,对金额降序排列,然后取条数
		SELECT * FROM zhangwu WHERE zname LIKE'%支出%' ORDER BY zmoney DESC LIMIT 3
```

## 30聚合函数

```sql
	* A: 聚合函数		
	* B: 函数介绍
		* 之前我们做的查询都是横向查询，它们都是根据条件一行一行的进行判断，而使用聚合函数查询是纵向查询，
			它是对一列的值进行计算，然后返回一个单一的值；另外聚合函数会忽略空值。
		* count：统计指定列不为NULL的记录行数；
		* sum：计算指定列的数值和，如果指定列；
		* max：计算指定列的最大值，如果指定列是字符串类型，那么使用字符串类型不是数值类型，那么计算结果为0排0序运算；
		* min：计算指定列的最小值，如果指定列是字符串类型，那么使用字符串排序运算；
		* avg：计算指定列的平均值，如果指定列类型不是数值类型，那么计算结果为0；

	* C: 案例代码
		/*
		   使用聚合函数查询计算
		*/

		-- count 求和,对表中的数据的个数求和  count(列名)
		-- 查询统计账务表中,一共有多少条数据
		SELECT COUNT(*)AS'count' FROM zhangwu

		-- sum求和,对一列中数据进行求和计算 sum(列名)
		-- 对账务表查询,对所有的金额求和计算
		SELECT SUM(zmoney) FROM zhangwu
		-- 求和,统计所有支出的总金额
		SELECT SUM(zname) FROM zhangwu WHERE zname LIKE'%收入%'

		INSERT INTO zhangwu (zname) VALUES ('彩票收入')

		-- max 函数,对某列数据,获取最大值
		SELECT MAX(zmoney) FROM zhangwu

		-- avg 函数,计算一个列所有数据的平均数
		SELECT AVG(zmoney)FROM zhangwu
```

## 31分组查询

```sql
* A: 分组查询
	* a: 使用格式
		* 分组查询是指使用group by字句对查询信息进行分组,例如：我们要统计出zhanguw表中所有分类账务的总数量,这时就需要使用group by 来对zhangwu表中的账务信息根据parent进行分组操作。
		* SELECT 字段1,字段2… FROM 表名 GROUP BY 字段 HAVING 条件;
		* 分组操作中的having子语句，是用于在分组后对数据进行过滤的，作用类似于where条件。
	* b: having与where的区别
		* having是在分组后对数据进行过滤.
		* where是在分组前对数据进行过滤
		* having后面可以使用分组函数(统计函数)
		* where后面不可以使用分组函数。

		
 * B: 案例代码		
		/*
			查询所有的数据
            吃饭支出 共计多少
			工资收入 共计多少
			服装支出 共计多少
			股票收入 共计多少
			打麻将支出 共计多少钱
			
			分组查询:  group by 被分组的列名
			必须跟随聚合函数
			select 查询的时候,被分组的列,要出现在select 选择列的后面
		*/
		  SELECT SUM(zmoney),zname FROM zhangwu GROUP BY zname
		  
		-- 对zname内容进行分组查询求和,但是只要支出
			SELECT SUM(zmoney)AS 'getsum',zname FROM zhangwu WHERE zname LIKE'%支出%'
			GROUP BY zname
			ORDER BY getsum DESC

		-- 对zname内容进行分组查询求和,但是只要支出, 显示金额大于5000
		-- 结果集是分组查询后,再次进行筛选,不能使用where, 分组后再次过滤,关键字 having
			SELECT SUM(zmoney)AS 'getsum',zname FROM zhangwu WHERE zname LIKE'%支出%'
			GROUP BY zname HAVING getsum>5000	
```

## 参考

https://blog.csdn.net/oschina_41544814/article/details/84592466

[MYSQL limit用法 - 就是你baby - 博客园 (cnblogs.com)](https://www.cnblogs.com/cai170221/p/7122289.html)

