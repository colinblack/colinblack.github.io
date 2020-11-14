---
title: mysql基础
date: 2020-02-26 07:27:19
categories:
    - 中间件  
tags:
    - mysql  
---

## 数据类型  
* 数值   
```sql
    类型      所占字节       说明
    tinyint    1            小整数值，如状态
    smallint   2            大整数值
    mediumint  3            大整数值
    int        4            大整数值
    bigint     8            极大整数值
    float      4            单精度浮点数值
    double     8            双精度浮点数值
    decimal    Max(D+, M+)  含小数值，例如金额
```
 
* 日期    
```sql
    类型        所占字节数          说明
    date        3                  YYYY-MM-DD
    time        3                  HH:MM:SS
    year        1                  YYYY
    datetime    8                  YYYY-MM-DD HH:MM:SS
    timestamp   8                  YYYYMMDDHHMMSS
```

* 字符串  
```sql
    类型       所占字节数     说明
    char       0~255         定长字段串
    varchar    0~65535       变长字符串
    text       0~65535       长文本数据
    blob                     二进制形式的文本数据
```

## 数据完整性  
* 实体完整性  
求每张表都有唯一标识符，每张表中的主键字段不能为空且不能重复  
约束方法：唯一性约束、主键约束、标识列  

* 域完整性  
表中某些列不能输入无效的值, 如数据类型、格式、值域范围、是否允许空值等  
约束方法：限制数据类型、检查约束、默认值、非空约束  

* 参照完整性  
求关系中不允许引用不存在的实体   
约束方法：外键约束  

*  用户自定义完整性  
针对某一具体关系数据库的约束条件，它反映某一具体应用所涉及的数据必须满足的语义要求    
约束方法：规则、存储过程、触发器   

唯一性约束  
```sql
    #可以使用关键字 UNIQUE 实现字段的唯一性约束，从而保证实体的完整性  
    #UNIQUE 意味着任何两条数据的同一个字段不能有相同值。
    #一个表中可以有多个 UNIQUE 约束。

    #在创建表时添加唯一性约束
    create table person(
    id int not null auto_increment primary key comment '主键id',
    name varchar(30) comment '姓名',
    id_number varchar(18) unique comment '身份证号'
    );
```

外键约束 
```sql
    -- 外键（FOREIGN KEY）约束定义了表之间的一致性关系，用于强制参照完整性
    -- 外键约束定义了对同一个表或其他表的列的引用，这些列具有PRIMARY KEY或UNIQUE约束

    -- 学生表(主表)
    create table stu(
    stu_no int not null primary key comment '学号',
    stu_name varchar(30) comment '姓名'
    );


    --成绩表
    --在插入数据时，必须先向主表插入，再向从表插入, 删除数据时正好相反
    create table sc(
    id int not null auto_increment primary key comment '主键id',
    stu_no int not null comment '学号',
    course varchar(30) comment '课程',
    grade int comment '成绩',
    foreign key(stu_no) references stu(stu_no)
    );

```

## 函数　
```sql
--数学函数    
ABS SQRT MOD SIN COS TAN COT
--字符串函数 
LENGTH LOWER UPPER TRIM SUBSTRING
-- 日期和时间函数 
NOW CURDATE CURTIME SYSDATE DATE_FORMAT YEAR MONTH WEEK
--聚合函数 
COUNT SUM AVG MIN MAX
--条件判断函数 
IF IFNULL CASE WHEN
--系统信息函数 
VERSION DATABASE USER
-- 加密函数 
MD5 SHA1 SHA2
```

* now
```sql
-- 返回当期时间
select now()

--在实际应用中，大多数业务表都会带一个创建时间create_time字段，用于记录每一条数据的产生时间。在向表
--中插入数据时，就可以在insert语句中使用now()函数
insert into user(id, name, create_time) values(1, 'zhangsan', now());
```
* date_format
```sql
--在实际应用中，一般会按照标准格式存储日期/时间，如 2019-12-13 14:15:16 。在查询使用数据时，往往又
--会有不同的格式要求，这时就需要使用date_format()函数进行格式 转换
select name, date_format(birthday, '%Y/%m/%d') from user;
```
* 聚合函数　
聚合函数是对一组值进行计算，并返回单个值
```sql
    count 返回符合条件的记录总数
    sum 返回指定列的总和，忽略空值
    avg 返回指定列的平均值，忽略空值
    min 返回指定列的最小值，忽略空值
    max 返回指定列的最大值，忽略空值
```

* ifnull
函数ifnull()用于处理NULL值。
ifnull(v1,v2)，如果 v1 的值不为 NULL，则返回 v1，否则返回 v2
```sql
    select ifnull(1/0, 0); --0
    select ifnull(1, 0); --1
```
* case when 
流程控制语句，可以在SQL语句中使用case when来获取更加准确和直接的结果
SQL中的case when类似于编程语言中的if else或者switch
```sql
    -- 语法
    CASE [col_name] WHEN [value1] THEN [result1]…ELSE [default] END
    CASE WHEN [expr] THEN [result1]…ELSE [default] END

    -- as 取别名　
    select id, name case sex when '男'　then 'F' when '女' then 'M' else '' end as sex
```


## 索引
### 慢查询日志　　
#### mysql的日志类型　

| 日志                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 重做日志（redo log）         | 重做日志是一种物理格式的日志，记录的是物理数据页面的修改的信息，其redo log是顺序 写入redo log file的物理文件中去的。 |
| 回滚日志（undo log）         | 回滚日志是一种逻辑格式的日志，在执行undo的时候，仅仅是将数据从逻辑上恢复至事务 之前的状态，而不是从物理页面上操作实现的，这一点是不同于redo log的。 |
| 二进制日志（binlog）         | 二进制日志是一种逻辑格式的日志，以二进制文件的形式记录了数据库中的操作，但不记录 查询语句。 |
| 错误日志（errorlog）         | 错误日志记录着mysqld启动和停止，以及服务器在运行过程中发生的错误的相关信息。 |
| 慢查询日志（slow query log） | 慢查询日志记录执行时间过长和没有使用索引的查询语句           |
| 一般查询日志（general log）  | 记录了服务器接收到的每一个查询或是命令，无论这些查询或是命令是否正确甚至是否包含 语法错误，general log都会将其记录下来 |
| 中继日志（relay log）        | 中继日志类似二进制；可用于复制架构中，使从服务器和主服务器的数据保持一致 |

#### 慢查询日志属性　　

| 参数                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| slow_query_log                | 是否开启慢查询日志，1表示开启，0表示关闭。                   |
| slow_query_log_file           | 慢查询日志存储路径，可选。 注意：MySQL 5.6之前的版本，参数名为 log-slow-queries |
| long_query_time               | 阈值，当SQL语句的响应时间超过该阈值就会被记录到日志中。      |
| log_queries_not_using_indexes | 未使用索引的查询也被记录到慢查询日志中，可选。               |
| log_output                    | 日志存储方式，默认为FILE。 log_output=‘FILE’表示将日志存入文件 log_output=‘TABLE’表示将日志存入数据库 log_output=‘FILE,TABLE’表示同时将日志存入文件和数据库 |

#### 开启慢查询日志　
慢查询日志可以通过命令临时设置，也可以修改配置文件永久设置
```
#查看是否开启慢查询日志
show variables like 'slow%';
#临时开启慢查询日志
set slow_query_log='ON';
set long_query_time=1;
#慢查询日志文件所在位置
show variables like '%datadir%';
```

### 查询分析器
explain命令可以查看SQL语句的执行计划。当explain与SQL语句一起使用时，MySQL将显示来自
优化器的有关语句执行计划的信息。也就是说，MySQL解释了它将如何处理语句，包括有关如何联
接表以及以何种顺序联接表的信息

#### explain功能　
分析出表的读取顺序
数据读取操作的操作类型
哪些索引可以使用
哪些索引被实际使用
表之间的引用
每张表有多少行被优化器查询

#### explain 使用　
<img src="https://i.loli.net/2020/08/20/QBWxtJcN8Lq2iDg.png"/>   

#### 结果分析　

| 参数          | 描述                                                        |
| ------------- | ----------------------------------------------------------- |
| id            | 执行select子句或操作表的顺序                                |
| select_type   | 查询的类型，如SIMPLE、PRIMARY、SUBQUERY、DERIVED、UNION等   |
| table         | 当前行使用的表名                                            |
| partitions    | 匹配的分区                                                  |
| type          | 连接类型，如system、const、eq_ref、ref、range、index、all等 |
| possible_keys | 可能使用的索引                                              |
| key           | 实际使用的索引，NULL表示未使用索引                          |
| key_len       | 查询中使用的索引长度                                        |
| ref           | 列与索引的比较                                              |
| rows          | 扫描的行数                                                  |
| filtered      | 选取的行数占扫描的行数的百分比，理想的结果是100             |
| extra         | 其他额外信息                                                |

### 索引种类

| 索引种类 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 普通索引 | 最基本的索引，没有任何限制，仅加速查询                       |
| 唯一索引 | 索引列的值必须唯一，但允许有空值                             |
| 主键索引 | 一种特殊的唯一索引，不允许有空值 一般是在建表的同时自动创建主键索引 |
| 复合索引 | 两个或多个列上的索引被称作复合索引                           |
| 全文索引 | 对文本内容进行分词索引       


### 创建索引
* 创建普通索引 
CREATE INDEX indexName ON tableName(columnName(length))
```sql
create index name_idx on base(name);
```
CREATE UNIQUE INDEX indexName ON tableName(columnName(length))   
* 创建唯一索引
```sql
create unique index name_idx on base(name);
```

* 创建复合索引
CREATE INDEX indexName ON tableName(columnName1, columnName2, …)
```sql
create index muti_idx on base(name, sex, age);
```

### 删除索引
DROP INDEX [indexName] ON tableName;  

### 查看索引  
SHOW INDEX FROM tableName;  

* 使用索引分析器 
```sql
explain select * from base where name='cp';
```

### 复合索引   
* 前导列特性（最左前缀）  
在MySQL中，如果创建了复合索引(name, salary, dept)，就相当于创建了(name, salary, dept)、
(name, salary)和(name)三个索引，这被称为复合索引前导列特性， 因此在创建复合索引时应该将
最常用作查询条件的列放在最左边，依次递减

```sql
未使用索引
    select * from employee where salary=8800;
    select * from employee where dept='部门A';
    select * from employee where salary=8800 and dept='部门A';

    使用索引
    select * from employee where name='liufeng';
    select * from employee where name='liufeng' and salary=8800;
    select * from employee where name='liufeng' and salary=8800 and dept='部门A';
```

### 覆盖索引  
* 什么是覆盖索引  
即select的数据列只从索引中就能得到，不必读取数据行，也就是只需扫描索引就可以得到查询结果  

* 几点说明  
1. 使用覆盖索引，只需要从索引中就能检索到需要的数据，而不要再扫描数据表；  
2. 索引的体量往往要比数据表小很多，因此只读取索引速度会非常快，也会极大减少数据访问量；  
3. MySQL的查询优化器会在执行查询前判断，是否有一个索引可以覆盖所有的查询列；  
4. 并非所有类型的索引都可以作为覆盖索引，覆盖索引必须要存储索引列的值。像哈希索引、空间索引、全  
   文索引等并不会真正存储索引列的值  
5. 当一个查询使用了覆盖索引， 在查询分析器EXPLAIN的Extra列可以看到“Using index”       



### 索引优化  
* 选择区分度高的列建立索引  
区分度计算公式：count(distinct col)/count(*)，表示字段不重复的比例   
* 避免对索引列进行计算  
from_unixtime(create_time)='2014-05-29' 不会用到索引   
* 每次查询每张表仅能使用一个索引  




## 事务
* MyISAM 不支持， InnoDB
* 什么是事务
用来处理批量的mysql操作， 为了保证数据库的完整性，这些操作要么完全执行，要么完全不执行

* 几个术语    
回退(rollback) 撤销SQL语句的过程   
提交(commit) 未存储的SQL语句写入数据库表   
保留点(savepoint) 事务处理中设置临时占位符(place-holder).可以发布回退 

* 可以执行回退的语句
INSERT UPDATE和DELETE

* 事务的4个特性(ACID)
原子性: 批量的sql, 要么都发生， 要么都不发生  
一致性：事务前后的数据保持业务上的合理一致   
持久性：事务一旦发生不能取消，只能通过补偿性事务来抵消效果  
隔离性：在事务进行中，其他操作开不到此事务的任何效果   

* 操作命令  
开启事务：start transaction  
执行命令： xxx   
提交事务/回滚事务 commit/rollback  
设置隔离级别：set session transaction isolation level [read uncommitted |  read committed | repeatable read |serializable] 

* 示例  


```sql
//创建表
mysql> create table account(
    -> uname char(10),
    -> money int)
    -> engine innodb charset utf8;
//插入数据
mysql> insert into account values
    -> ('zhang', 5000),
    -> ('lisi', 3000);

//开启事务
start transaction

//更新数据
update account set money = money + 100000  where uname='lisi';

//查询
mysql> select * from account;
+-------+-------+
| uname | money |
+-------+-------+
| zhang |  5000 |
| lisi  |  3000 |

//设置隔离级别
mysql> set session transaction  isolation level read uncommitted

//提交事务
mysql> commit;

```

* 隔离级别  
read uncommit : "脏读", 读到未提交事务的内容  
read commited : 一个事务在进行的过程中， 读不到另外一个进行中事务的内容，但是能读到已经完成事务中的内容  
repeatable read: 可重复读,即在一个事务过程中,所有信息都来自事务开始那一瞬间的信息,不受其他已提交事务的影响. (大多数的系统,用此隔离级别)  
serializable: 串行化, 所有的事务,必须编号,按顺序一个一个来执行,也就取消了冲突的可能.这样隔离级别最高,但事务相互等待的等待长. 在实用,也不是很多 

* c++中使用事务
```cpp
//参考brks
bool MysqlConnection::transaction(std::list<std::string> sql){
	int ret = 0;
	//手动提交
	mysql_autocommit(mysql_, 0);

	//开始事务
	mysql_query(mysql_, "start transaction");
	for(auto iter = sql.begin(), iter != sql.end(); iter++){
		ret = mysql_qurey(mysql_, (*iter).c_ctr());
		if(ret != 0){
			break;
		}
	}

	if(ret != 0){
		//回滚
		mysqL_query(mysql_, "rollback");
		LOG_ERROR("excute transaction failed.");
		return false;
	}	

	if(0 != mysql_query(mysql, "commit")){
		LOG_ERROR("commit transaction failed.");
		return false;
	}

	return true;
}
```


## 存储过程
一组为了完成特定功能的SQL语句集， 经编译后存储在数据库中， 用户通过存储过过程名并传参来调用它

* 优点
增强的sql语音的功能和灵活性   
存储过程允许标准组件式编程   
较快的执行速度(存储过程是预编译的)    
减少网络流量    
可被作物一种安全机制充分利用   

* 创建步骤   
选择一个数据库    
改变分割符：delimiter $$(避免使用;作为结束标记)  
```sql
mysql> use test;
mysql> delimiter $$

mysql> create procedure p_hello()
    -> begin
    -> select 'hello';
    -> select 'world';
    -> end
    -> $$;

mysql> delimiter ;

mysql> call p_hello;

```

* 存储过程中的参数  
in: 输入参数     
必须在调用存储过程之前指定  
out: 输出参数  
可以在存储过程内部改变并返回  
inout:输入输出参数  
可以在调用时指定， 并可修改和返回  
in 在存储过程中修值是传入值得一份拷贝，传入的值不会改变
```sql
mysql> create procedure v_test1(in p_int int)
    -> begin
    -> select p_int;
    -> set p_int = p_int +1;
    -> select p_int;
    -> end;
    -> $$;

mysql> delimiter 
mysql> set @p_int =3;


//存储过程里的值被修改
mysql> call v_test1(@p_int);
+-------+
| p_int |
+-------+
|     3 |
+-------+
1 row in set (0.00 sec)

+-------+
| p_int |
+-------+
|     4 |
+-------+

//外面的值没变
mysql> select @p_int;
+--------+
| @p_int |
+--------+
|      3 |
+--------+
```
out 不认可传入的值，修改后值会变
```sql
mysql> create procedure p_test_out(out v_out_int int)
    -> begin
    -> select v_out_int;
    -> set v_out_int=15;
    -> select v_out_int;
    -> end
    -> $$;

mysql> set @v_out_int=10;
    -> $$;
mysql> call p_test_out(@v_out_int);
    -> $$;
+-----------+
| v_out_int |
+-----------+
|      NULL |
+-----------+
1 row in set (0.00 sec)

+-----------+
| v_out_int |
+-----------+
|        15 |
+-----------+
```
inout 类似于引用，认可传入的值， 修改后会改变


在存储过程里面定义变量
```sql
mysql> create procedure p_vartest()
    -> begin
    -> declare a varchar(20) default 'abc';
    -> select a;
    -> end;
    -> $$;
Query OK, 0 rows affected (0.00 sec)

mysql> call p_vartest;
    -> $$;
+------+
| a    |
+------+
| abc  |
+------+
1 row in set (0.00 sec)

```

* c++中使用存储过程  
* 创建存储过程  
* 初始化mysql: mysql_init   
* 链接mysql: mysql_real_connect   
* 调用存储过程: mysql_real_qurey, 第二个参数，"call xxx(xxx)"
* 释放资源与连接：mysql_free_result， mysql_close   

## 视图
由查询结果形成的一张虚拟表， 如果某个查询结果出现的非常频繁， 要经常拿这个查询结果来做子查询  
视图本身不包含数据， 它返回的数据是从其他表中检索出来的

* 作用   
1. 权限控制：如某几个列允许用户查询，其他的列不允许， 可以通过视图开放其中一列或几列           
2. 简化复杂的查询

* 视图能否更新，删除，添加  
如果视图的每一行是和物理表一一对应的才可以   
如果view的行是由物理表多行经过计算的到的结果，view不可以更新  

* 视图放在哪儿（视图算法） 
1. 对于简单的查询形成的view, 再对view查询时，如where, order等， 可以把建视图的语句+查询视图的语句合并成=》查物理的语句，这种视图算法叫merge(合并)   
2. 视图的语句本身比较复杂，很难再和查询视图的语句合并，mysql先执行视图的创建语句，把结果集形成内存中的临时表，然后再去查临时表(temptable)

 




















