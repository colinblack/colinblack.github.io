---
title: mysql
date: 2020-07-04 22:10:21
categories:
    - 中间件
tags: mysql
---

### 基本操作  
```
    #新建数据库
    CREATE DATABASE 名
    #显示当前使用的数据库
    select database();
    # 查询数据库支持的存储引擎  
    show engines;
    #删除表  
    DROP TABLES 名
```

#### 创建表
```
    #语法  
    CREATE [TEMPORARY] TABLE [IF NOT EXISTS] table_name [(create_definition,…)] [table_options]
    [select_statement] 
    #TEMPORARY：表示创建临时表，在当前会话结束后将自动消失
    #IF NOT EXISTS：在建表前，先判断表是否存在，只有该表不存在时才创建
    #create_definition：建表语句的关键部分，用于定义表中各列的属性
    #table_options：表的配置选项，例如：表的默认存储引擎、字符集
    #select_statement：通过select语句建表

    eg:
    create table contacts(
        id int not null auto_increment primay key,
        name varchar(30),
        phone varchar(20)
    ) engine=innodb default charset=utf8;
```

#### 字段操作  
```
    #添加
    alter table stat_online_players add time char(12);
    #删除, 如果是主键, 必须先删掉主键约束
    alter table stat_online_players drop column hour, drop column minute;
    #修改字段  
    alter table base modify recover_time int(10) NOT NULL DEFAULT '0';

    #修改字段名 
    alter table base change barrier archive_chip binary(24) ;
```

* 删除带主键约束的行  
![p1](/images/mysql_20200705_1.png)


* 删除主键约束
```
    alter table stat_online_players drop primary key; #增加主键约束（要先删掉数据吗?)
    alter table stat_online_players add primary key(zone_id, date);#主键要先删再加才行
```



#### 数据操作 
##### 插入数据  
```sql
    #INSERT 插入单条数据：
    INSERT INTO table_name (field1, field2, ..., fieldN) VALUES (value1, value2, ..., valueN)
    eg:
    insert into contacts(name, sex, phone) values('张三', 1, '11111111111');

    #INSERT 插入多条数据：
    INSERT INTO table_name (field1, field2, ..., fieldN) VALUES (valueA1, valueA2, ..., valueAN), (valueB1,
    valueB2, ..., valueBN), …, (valueN1, valueN2, ..., valueNN);

    eg:
    insert into contacts(name, sex, phone) values('李四', 1, '22222222222'), ('lily\'s cat', 2, '33333333333'), ("jane's", 2, '44444444444');
```
* 注意事项:  
如果字段是字符型，值必须使用单引号或者双引号，如”value”;如果值本身带单引号或双引号，需要转义   
如果所有列都要添加数据，insert into语句可以不指定列,即   
```sql
    INSERT INTO table_name VALUES (value1, value2, ..., valueN);
```
* insert into 和 replace into 比较  
数据存在时,  replace 为替换
数据不存在时，replace 为插入且效率比insert高



##### 修改数据  
```sql
    --语法:
    UPDATE table_name SET field1=newValue1, field2=newValue2 [WHERE Clause]
    --eg:
    update contacts set sex=2;
    update contacts set sex=2, phone='55555555555' where id=3;
```  
注意事项：
可以同时更新一个或多个字段  
可以通过where子句来指定更新的范围，如果不带where，则更新数据表中的所有记录  


##### 删除数据  
```sql
    --语法:
    DELETE FROM table_name [WHERE Clause]

    --eg:
    delete from contacts where id = 4;
```
注意事项：
可以通过where子句来指定删除的范围，如果不带where，则删除数据表中的所有记录


##### 查询语句   
```sql
    -- 语法 
    SELECT column_name1, column_name2
    FROM table_name
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position} [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
```
###### where语句  
在SQL中，insert、update、delete和select后面都能带where子句，用于插入、修改、删除或查询指定条件的记录  
单条件查询  
```sql
    -- 语法  
    SELECT column_name FROM table_name WHERE column_name 运算符 value
    运算符            描述
    =                 等于
    <>                或 != 不等于
    >                 大于
    <                 小于
    >=                大于等于
    <=                小于等于
    between and       选取介于两个值之间的数据范围；在MySQL中，相当于>=并且<=

    -- eg
    select * from employee where salary between 10000 and 12000;
```
多条件查询  
在where子句中，使用and、or可以把两个或多个过滤条件结合起来  
```sql 
    -- 语法  
    SELECT column_name FROM table_name WHERE condition1 AND condition2 OR condition3
    运算符              描述
    and                表示左右两边的条件同时成立
    or                 表示左右两边只要有一个条件成立

    -- eg 
    select * from employee where sex='男' and (salary <=4000 or salary >= 10000);
```

###### in运算符  
运算符 IN 允许我们在 WHERE 子句中过滤某个字段的多个值  
```sql
    -- 语法 
    SELECT column_name FROM table_name WHERE column_name IN(value1, value2, …)

    -- eg
    select * from employee where id in(1, 2, 3);  
```

###### like运算符   
在where子句中，有时候我们需要查询包含xxx 字符串的所有记录，这时就需要用到运算符like  
```sql
    -- 语法  
    SELECT column_name FROM table_name WHERE column_name LIKE ‘%value%’ 
    -- eg 
    select * from employee where name like '%小%';
```
说明
LIKE子句中的%类似于正则表达式中的*，匹配任意0个或多个字符
LIKE子句中的_匹配任意单个字符
LIKE子句中如果没有%和_，就相当于运算符=的效果

##### 查询结果排序和分页　
* order by
```sql
    --语法　
    SELECT column_name1, column_name2
    FROM table_name1, table_name2
    ORDER BY column_name, column_name [ASC|DESC]

    --eg 
    select * from empolyee order by sex, salary desc;
```
说明：
ASC表示按升序排列，DESC表示按降序排列。
默认情况下，对列按升序排

* limit 
```sql
    -- 语法　
    SELECT column_name1, column_name2
    FROM table_name1, table_name2
    LIMIT [offset,] row_count

    --eg  
    select * from employee limit 0, 5;
    select * from employee limit 5, 5;
    select * from employee limit 10, 5;
```
说明：
offset指定要返回的第一行的偏移量。第一行的偏移量是0，而不是1。
row_count指定要返回的最大行数。
limit的分页公式:
limit (page-1)*row_count, row_count

##### 分组　
* group by 
示根据某种规则对数据进行分组，它必须配合聚合函数进行使用，对数
据进行分组后可以进行count、sum、avg、max和min等运算。
```sql
    -- 语法 
    SELECT column_name, aggregate_function(column_name)
    FROM table_name
    GROUP BY column_name

    --eg
    select sex, count(*) from employee group by sex;
```
说明: 
    aggregate_function表示聚合函数
    group by可以对一列或多列进行分组

* having 
WHERE 关键字无法与聚合函数一起使用。HAVING 子句可
以对分组后的各组数据进行筛选
```sql
   -- 语法　
   SELECT column_name, aggregate_function(column_name)
   FROM table_name
   WHERE column_name operator value
   GROUP BY column_name
   HAVING aggregate_function(column_name) operator value

   --eg
   select dept, count(*) from employee group by dept having count(*) < 5;
```
* group_concat  
group_concat配合group by一起使用，用于将某一列的值按指定的分隔符进行拼接
MySQL默认的分隔符为逗号
```sql
    -- 语法　
    group_concat([distinct] column_name [order by column_name asc/desc ] [separator '分隔符'])   

    --eg 
    select dept, count(*), group_concat(name order by name desc separator ';') from employee group by dept;
```
##### 去重  
* distinct 
支持单列和多列　
```sql
    -- 语法
    SELECT DISTINCT column_name, column_name
    FROM table_name;

    -- eg
    select distinct username, city from footprint;
```


#### 多表操作　　
##### 表连接　　
在多个表之间通过一定的连接条件，使表之间发生关联，进而能从多个表之间获取数据　　　
```sql
    
    -- 语法  
    SELECT table1.column, table2.column
    FROM table1, table2
    WHERE table1.column1 = table2.column2;
```
* 表连接几种方式　　
内连接　join或inner join   
自连接　同一张表内的连接　
外连接　左外连接　left join, 右外连接 right join, 全外连接 full join

* 各种表连接的区别　
![p20200714_1](/images/mysql_20200714_p1.png)
交叉连接（cross join）：没有用where子句的交叉连接将产生笛卡尔积，第一个表的行数乘以第二个表的行数等于笛卡尔积
和结果集的大小

###### 内连接　
<img src="https://i.loli.net/2020/08/18/xW1zZPBD2gAykva.png"/>
```sql
select A.stu_no, A.name, B.course, B.score from student A join score B on(A.stu_no = B.stu_no);

select A.stu_no, A.name, B.course, B.score from student A inner join score B on(A.stu_no = B.stu_no);

select A.stu_no, A.name, B.course, B.score from student A score B while A.stu_no = B.stu_no;

```

###### 左连接　
<img src="https://i.loli.net/2020/08/18/BtTodxEVWwKn4Gi.png"/>
```sql
select A.stu_no, A.name, B.course, B.score from student A left join score B on(A.stu_no = B.stu_no);
```

###### 笛卡尔积
```sql
select A.stu_no, A.name, B.course, B.score from student A score B
```

##### 自连接　
一种特殊的表连接，它是指相互连接的表在物理上同为一张表，但是逻辑上是多张表。自
连接通常用于表中的数据有层次结构，如区域表、菜单表、商品分类表等　
```sql
#自连接语法
SELECT A.column, B.column
FROM table A, table B
WHERE A.column = B.column;
```
<img src="https://i.loli.net/2020/08/19/NHFdTWZBmAqSYU2.png"/>  


##### 子查询　
又成为内部查询和嵌套查询　
<img src="https://i.loli.net/2020/08/19/zJZjmYDhnICQ5fq.png"/>　
select 学号 姓名 地址 from 学生表 where 学号 in (select 学号 from 成绩表 where 科目=计算机)
* 子查询in
```sql
SELECT column_name FROM table_name
WHERE column_name IN(
 SELECT column_name FROM table_name [WHERE]
);
```
<img src="https://i.loli.net/2020/08/19/VITdOj3hgZGqNpX.png"/>   

* 子查询exists
```sql
SELECT column_name1
FROM table_name1
WHERE EXISTS (SELECT * FROM table_name2 WHERE condition);
```
<img src="https://i.loli.net/2020/08/19/gLpx9WRwhtc5Hay.png"/>   


#### 用户管理　　

#####  mysql权限体系　

| 层级       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 全局层级   | 适用于一个给定服务器中的所有数据库。这些权限存储在mysql.user表中。 GRANT ALL ON *.*和REVOKE ALL ON *.*只授予和撤销全局权限 |
| 数据库层级 | 适用于一个给定数据库中的所有目标。这些权限存储在mysql.db和mysql.host表中。 GRANT ALL ON db_name.*和REVOKE ALL ON db_name.*只授予和撤销数据库权限 |
| 表层级     | 适用于一个给定表中的所有列。这些权限存储在mysql.talbes_priv表中。 GRANT ALL ON db_name.tbl_name和REVOKE ALL ON db_name.tbl_name只授予和撤销表权限 |
| 列层级     | 适用于一个给定表中的单一列。这些权限存储在mysql.columns_priv表中。当使用REVOKE时，您必须指 定与被授权列相同的列 |
| 子程序层级 | CREATE ROUTINE, ALTER ROUTINE, EXECUTE和GRANT权限适用于已存储的子程序。这些权限可以被 授予为全局层级和数据库层级。而且，除了CREATE ROUTINE外，这些权限可以被授予为子程序层级， 并存储在mysql.procs_priv表中 |

MySQL的权限信息主要存储在以下几张表中，当用户连接数据库时，MySQL会根据这些表对用户
进行权限验证

| 表名        | 描述                                       |
| ----------- | ------------------------------------------ |
| user        | 用户权限表，记录账号、密码及全局性权限信息 |
| db          | 记录数据库相关权限                         |
| table_priv  | 用户对某个表拥有的权限                     |
| column_priv | 用户对某表的某个列所拥有的权限             |
| procs_priv  | 用户对存储过程及存储函数的操作权限         |

在MySQL中，使用CREATE USER来创建用户，用户创建后没有任何权限
**MySQL的用户账号由两部分组成：用户名和主机名，即用户名@主机名，主机名可以是IP或机器名称, 主机名为%表示允许任何地址的主机远程登录MySQL数据库**

```
#创建用户
CREATE USER '用户名' [@'主机名'] [IDENTIFIED BY '密码'];
#删除用户
DROP USER '用户名' [@'主机名'];
#修改密码
ALTER USER '用户名'@'主机名' IDENTIFIED BY '新密码';
```

##### 权限管理　

在MySQL数据库中，使用grant命令授权、revoke命令撤销授权

```
#授权
grant all privileges on databaseName.tableName to '用户名' [@'主机名'] ;
#撤销授权
revoke all privileges on databaseName.tableName from '用户名' [@'主机名'] ;
#刷新权限
FLUSH PRIVILEGES;
#查看权限
show grants for '用户名' [@'主机名'] ;
```

###### 权限列表　

使用grant和revoke进行授权、撤销授权时，需要指定具体是哪些权限，这些权限大体可以分为3类，数据类、结构类和管理类

| 数据                             | 结构                                                         | 管理                                                         |
| -------------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| SELECT INSERT UPDATE DELETE FILE | CREATE ALTER INDEX DROP CREATE TEMPORARY TABLES SHOW VIEW CREATE ROUTINE ALTER ROUTINE EXECUTE CREATE VIEW EVENT TRIGGER | USAGE GRANT SUPER PROCESS RELOAD SHUTDOWN SHOW DATABASES LOCK TABLES REFERENCES REPUCATION CUENT REPUCATION SLAVE CREATE USER |

###### 禁止root远程登录
1. root是MySQL数据库的超级管理员，几乎拥有所有权限，一旦泄露后果非常严重；
2. root是MySQL数据库的默认用户，所有人都知道，如果不禁止远程登录，可以针对root用户暴	力破解密码
<img src="https://i.loli.net/2020/08/19/7C6q2On1VrbZwaM.png"/>  


###### 忘记root密码解决 
<img src="https://i.loli.net/2020/08/19/oaiRTSve8bZQpF2.png"/>  
```
#关闭权限验证
mysqld --defaults-file="./my.cnf" --console --skip-granttables --shared-memory
#参数--defaults-file的值为配置文件my.cnf的完整路径
```
MySQL关闭权限验证后，直接通过 mysql 命令即可连接到数据库，并可正常执行各类操作 
```
#刷新权限
FLUSH PRIVILEGES;
#修改root用户的密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```




* sql导出数据库  
```
＃只导整个库结构:
    mysqldump -uroot -p1234 -d sg17_s0 > sg17_s0.sql
＃导出整个库结构和数据:
    mysqldump -uroot -p1234 sg17_s0 > sg17_s0.sql
＃只导出表结构:
    mysqldump -uroot -p1234 -d sg17_s0 concern > concern.sql
```
* sql导入数据库　　
```
    source [所在的路径//*.sql]
    mysql -uabc_f -p abc < abc.sql
```

* 查看端口 
```
    show global variables like 'port'
```


* 显示建表语句  
```
    show create table mainline_task_stat_zkw
```







* 调换行的位置
```
   alter table stat_online_players modify player int(10) unsigned after time;
```


* 求和
```
   select sum(peoples) as total from tutorial_stage_stat where day_id=19;
```

* 修改表结构
```
   alter table stat_key_data_online modify `date` char(12) NOT NULL
```
* 按列显示  
```
   select * from T_Account limit 1\G
```
* pymysql 
```
   安装(centos 6)：
   安装pip: sudo yum -y install epel-release, sudo yum -y install python-pip
   安装mypython: yum install -y mysql-devel, python-devel python-setuptools
                pip install MySQL-python
   安装mysql.connector包 pip install mysql-connector
```

* 查看mysql 配置文件的方法  
```
   which mysqld
   /usr/local/mysql/bin/mysql --verbose --help | grep -A 1 'Default options'
```

* MySQL中文乱码
[彻底解决MySQL中文乱码](https://mp.weixin.qq.com/s?__biz=MzIzNjg4MTE2Ng==&idx=4&mid=100000774&sn=c4ed7a8c2fee681523c3e30600be4bf8)

* mysql修改数据库表和表中的字段的编码格式的修改
[mysql修改数据库表和表中的字段的编码格式的修改](https://blog.csdn.net/luo4105/article/details/50804148)

* mysql的latin1支持中文
[mysql的latin1支持中文](https://blog.csdn.net/congcongsuiyue/article/details/41979643)

* mysql 两表联合查询
[mysql两表联合查询的四种情况](https://blog.csdn.net/wj123446/article/details/52870114/)

* mysql 设置最大连接数
```
1.第一种：命令行查看和修改最大连接数(max_connections)。
  >mysql -uuser -ppassword(命令行登录MySQL)
  mysql>show variables like 'max_connections';(查可以看当前的最大连接数)
  msyql>set global max_connections=1000;(设置最大连接数为1000，可以再次查看是否设置成功)
  mysql>exit  
2.设置/etc/my.cnf
  注:似乎两者都要设置才会成功
```
* 查看错误日志路径  
```
   在配置中查看
   /data/mysql/var
   数据库存放路径
   /var/log/message
   系统日志    
```

* 数据库遭到攻击
[数据库遭到攻击](https://jason5.cn/blog/ji-%5B%3F%5D-ci-shu-ju-ku-bei-gong-ji-de-jing-li.html)

