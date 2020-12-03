+++
title = "Mysql"
description = "mysql on ubuntu"
tags = [
    "mysql",
    "tech"
]
date = "2019-12-24"
topics = [
    "work"
]
toc = true
+++

### 设置serverTimeZone

Q：遗留问题一直没处理。测试环境mysql的默认timestamp值总是跟北京时间相差了10+小时，没定位清楚原因是什么。  
A： 系统默认的时区为`CST`（utc-6），使用北京时间(utc+8)插入时，默认会进行转换。可通过jdbc的url链接参数进行指定`serverTimezone=Asia/Shanghai`

[MySQL TIMESTAMP](https://www.mysqltutorial.org/mysql-timestamp.aspx) 
>The MySQL `TIMESTAMP` is a temporal [data type](https://www.mysqltutorial.org/mysql-data-types.aspx "MySQL Data Types") that holds the combination of [date](https://www.mysqltutorial.org/mysql-date/) and [time](https://www.mysqltutorial.org/mysql-time/). The [format ](https://www.mysqltutorial.org/mysql-date_format/ "MySQL DATE_FORMAT function")of a `TIMESTAMP` is `YYYY-MM-DD HH:MM:SS` which is fixed at 19 characters.
>
>The `TIMESTAMP` value has a range from `'1970-01-01 00:00:01' UTC` to `'2038-01-19 03:14:07' UTC`.
>
>When you [insert ](https://www.mysqltutorial.org/mysql-insert-statement.aspx "MySQL INSERT statement")a `TIMESTAMP` value into a table, MySQL converts it from your connection’s time zone to UTC for storing.

官方文档[11.2.2 The DATE, DATETIME, and TIMESTAMP Types](https://dev.mysql.com/doc/refman/8.0/en/datetime.html) 

查看系统/当前session的时区：`SELECT @@GLOBAL.time_zone, @@SESSION.time_zone;`值为`system`；  
查看system对应的时区：`SELECT @@system_time_zone;` 

[5.1.15 MySQL Server Time Zone Support](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)

>`timezone` values can be given in several formats, none of which are case-sensitive:
>
>- As the value 'SYSTEM', indicating that the server time zone is the same as the system time zone.
>
>- As a string indicating an offset from UTC of the form [H]H:MM, prefixed with a + or -, such as '+10:00', '-6:00', or '+05:30'. 
>- As a named time zone, such as 'Europe/Helsinki', 'US/Eastern', 'MET', or 'UTC'.

系统设置可参考 [How do I set the time zone of MySQL?](https://stackoverflow.com/questions/930900/how-do-i-set-the-time-zone-of-mysql) 

建立mysql的url链接时可以指定serverTimeZone——`jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai&useLegacyDatetimeCode=false`

参考[Setting the MySQL JDBC Timezone Using Spring Boot Configuration](https://www.baeldung.com/mysql-jdbc-timezone-spring-boot)，或[这里](https://stackoverflow.com/a/7610174/1087122)，[更详细的问题说明](https://stackoverflow.com/a/19075291/1087122) 

[Time Zone Map](https://www.timeanddate.com/time/map/) 

![](https://upload-images.jianshu.io/upload_images/3296949-3f4652dff8c7eba0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 创建index

[mysql index](https://www.tutorialspoint.com/mysql/mysql-indexes.htm)   

 A unique index means that two rows cannot have the same index value. Here is the syntax to create an Index on a table.

`CREATE UNIQUE INDEX index_name ON table_name ( column1, column2,...);`

 omit the UNIQUE keyword from the query to create a simple index. A Simple index allows duplicate values in a table.

### 常用函数

[mysql resources](https://www.w3resource.com/mysql/mysql-functions-and-operators.php)，五星推荐：gif演示 && 视频演示。

- 拼接字段：`CONCAT`,例如`select CONCAT(git_group ,'/', project_name) as name , ut_block.* from ut_block ;`
- 计算平均值：`avg`，例如`select avg(total_time) from su_result where id > 9000;`
- 时间戳转换：`unix_timestamp`将时间格式转换为秒/毫秒，例如`unix_timestamp('2020-05-20 13:14:00') * 1000`
- 时间戳转换：`FROM_UNIXTIME`将秒转换为时间格式，例如`FROM_UNIXTIME(start_time / 1000, '%y-%m-%d %T') AS start`，start_time为毫秒值
- 计算TimeStamp之间的耗时：`TIMESTAMPDIFF`，例如`TIMESTAMPDIFF(c,created_at,updated_at) as DT`。creatte_at, updated_at为`timestamp`格式，计算两个时间戳之间多少秒(MICROSECOND)、毫秒(MICROSECOND) 

- 设置变量： `set @start = '2020-05-06 00:00:00'`，使用时使用`start_time < unix_timestamp(@start) * 1000`方式

[FROM_UNIXTIME() Examples](https://database.guide/from_unixtime-examples-mysql/) Available Specifiers

[MySQL: grouping continuous ranges](https://explainextended.com/2009/07/24/mysql-grouping-continuous-ranges/)  

[Group by price range](https://snipplr.com/view/41322) [Group by range](https://stackoverflow.com/questions/6687534/group-by-range-in-mysql)

```sql
set @start = '2020-06-12 00:00:00';
set @end = '2020-06-13 00:00:00';

-- 语言分布
SELECT count(1) as num , lang_type
FROM su_result
WHERE (id > 150000
    AND start_time > unix_timestamp(@start) * 1000 and start_time < unix_timestamp(@end) * 1000)

group by lang_type;

-- ut result
SELECT  count(1) as num,  ut_result
FROM su_result
WHERE (id > 150000 and ut_block = 1
    AND start_time > unix_timestamp(@start) * 1000 and start_time < unix_timestamp(@end) * 1000)

group by ut_result;


-- 等待分布
select case when wait_duration > 1800000 then '1 30m以上'
            when wait_duration <= 1800000 and wait_duration > 600000 then '2 10~30m'
            when wait_duration <= 600000 and wait_duration > 60000 then '3 1~10m'
            else '4 1m以下'
    end as wsec, count(*) AS num
from su_result
WHERE (id > 150000
    AND start_time > unix_timestamp(@start) * 1000 and start_time < unix_timestamp(@end) * 1000)
group by wsec;


-- sonar耗时分布
select case when sonar_duration > 600000 then '1 10m以上'
            when sonar_duration <= 600000 and sonar_duration > 60000 then '2 1~10m'
            when sonar_duration <= 60000 and sonar_duration > 30000 then '3 30~60s'
            when sonar_duration <= 30000 and sonar_duration > 10000 then '4 10~30s'
            else '5 10s以下'
           end as wsec, count(*) AS num
from su_result
WHERE (id > 150000
    AND start_time > unix_timestamp(@start) * 1000 and start_time < unix_timestamp(@end) * 1000)
group by wsec;


-- week
SELECT CASE
           WHEN num > 100 THEN '1 100次以上'
           WHEN num <= 100
               AND num > 50 THEN '2 50~100次'
           WHEN num <= 50
               AND num > 10 THEN '3 10~50次'
           ELSE '4 10次以下'
           END AS Number, COUNT(*) AS num
FROM (
         SELECT project_name, group_name, COUNT(1) AS num
         FROM su_result
         WHERE (id > 100000
             AND start_time > unix_timestamp('2020-06-01 00:00:00') * 1000
             AND start_time < unix_timestamp('2020-06-08 00:00:00') * 1000)
         GROUP BY project_name
         ORDER BY num DESC
     ) week
GROUP BY Number;

-- times
SELECT group_name, project_name, COUNT(1) AS num
FROM su_result
WHERE (id > 100000
    AND start_time > unix_timestamp('2020-06-01 00:00:00') * 1000
    AND start_time < unix_timestamp('2020-06-08 00:00:00') * 1000)
GROUP BY project_name
ORDER BY num DESC;


```

### 实现主从备份

[MySQL主从备份配置](https://www.cnblogs.com/eric-fang/p/9285093.html)、 [mysql实现主从备份](https://www.cnblogs.com/baoyi/p/mysql_master_slave.html)

### 查看数据库大小

[refer](https://chartio.com/resources/tutorials/how-to-get-the-size-of-a-table-in-mysql/)  
[official documentation](https://dev.mysql.com/doc/refman/8.0/en/tables-table.html)

- `DATA_LENGTH` is the length (or size) of all data in the table (in `bytes`).
- `INDEX_LENGTH` is the length (or size) of the index file for the table (also in `bytes`).

```
SELECT table_schema AS "Database",
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (MB)"
FROM information_schema.TABLES
GROUP BY table_schema;

# 返回结果类似——
Database  size(MB)
daxiang	    0.36
information_schema	0.16
mysql	2.43
performance_schema	0.00
sonar_db	36.00
stfmonkey	1.86
sys	0.02
```

### 查看具体的数据库中的表大小

```
SELECT table_name AS "Table",
       ROUND(((data_length + index_length) / 1024 / 1024), 2) AS "Size (MB)"
FROM information_schema.TABLES
WHERE table_schema = 'sonar_db' 
    -- without table name, then all tables will be checked
    and TABLE_NAME = 'live_measures' 
ORDER BY (data_length + index_length) DESC;

# 结果
Table           Size(MB)
live_measures	17.41

```

### 查看当前数据库都有哪些人在连接

[List of users accessing database](https://stackoverflow.com/questions/5575347/list-of-users-accessing-database) 

`SHOW PROCESSLIST` or `select * from information_schema.processlist` 
```
"ID","USER","HOST","DB","COMMAND","TIME","STATE","INFO","TIME_MS","ROWS_SENT","ROWS_EXAMINED"
```

[DDL VS. DML](https://stackoverflow.com/questions/2578194/what-is-ddl-and-dml)

**DDL** is Data Definition Language : it is used to define data structures.

For example, with SQL, it would be instructions such as `create table`, `alter table`, ...


**DML** is Data Manipulation Language : it is used to manipulate data itself.

For example, with SQL, it would be instructions such as `insert`,`update`, `delete`, ...

### install on Ubuntu 18.04

[How to Install MySQL on Ubuntu 18.04
This tutorial is also available for:](https://linuxize.com/post/how-to-install-mysql-on-ubuntu-18-04/)  

```
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'very_strong_password';
```

### 重置auto_increment初始值

[TRUNCATE TABLE 语法](https://blog.csdn.net/weixin_36210698/article/details/70176652)
```
-- 1. 将数据全部删除，而且重新定位自增的字段
truncate table 你的表名

-- 2. 直接重置autoIncrement的值
ALTER TABLE table_name AUTO_INCREMENT = 1;
-- 可以先删除表内容 
delete from table_name;
```

### 复制表结构

[复制表结构](https://www.cnblogs.com/chenmh/p/5644644.html)

```
-- like方法能一模一样的将一个表的结果复制生成一个新表，
-- 包括复制表的备注、索引、主键外键、存储引擎等。
CREATE TABLE IF NOT EXISTS taskInterveneData LIKE taskRecord;

-- select 方法将复制完整的数据和结构 
-- 其它表属性都有系统的配置文件决定；包括存储引擎、默认字符集等都是有系统的默认配置所决定
CREATE TABLE IF NOT EXISTS tb_base_select
AS
SELECT *
FROM tb_base;
```


### 查询数据库大小

[SQL命令查看Mysql数据库大小](https://blog.csdn.net/atec2000/article/details/7041352)

```
1、进入information_schema 数据库（存放了其他的数据库的信息）
use information_schema;

2、查询所有数据的大小：
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables;

3、查看指定数据库的大小：
比如查看数据库home的大小
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables
WHERE table_schema = 'home';

4、查看指定数据库的某个表的大小
比如查看数据库home中 members 表的大小
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables
WHERE table_schema = 'home'
    AND table_name = 'members';
```
 

### 查看表结构 
```
desc `tableName`

desc 表名;
show columns from 表名;
describe 表名;
show create table 表名;
```
### 查询数据库版本号 select @@Version

### CLIENT_PLUGIN_AUTH is required 报错

版本过高，降低mysql-connector-java的版本。

```
# 6.0.6 com.mysql.cj.jdbc.Driver
String driver = "com.mysql.jdbc.Driver"; //
String url = "jdbc:mysql://" + DB_ADDRESS + ":" + DB_PORT + "/" + DB_NAME
        + "?connectTimeout=10000&characterEncoding=UTF-8";
String user = DB_USER;
String password = DB_PASS;

try {
    Class.forName(driver);
    DriverManager.setLoginTimeout(10);
    SmsService.conn = DriverManager.getConnection(url, user, password);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 运行远程访问
```

# 安装 
apt-get install mysql-server mysql-client

#登录mysql
mysql -u root -p

# 运行远程访问 注意 youpassword
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
FLUSH PRIVILEGES ;

# 需要修改 /etc/mysql/my.cnf的配置 /etc/mysql/mysql.conf.d
# 修改bind-address=127.0.0.1为bind-address=0.0.0.0

# 如何查看mysql数据库的端口
# 启动，并进入mysql后，输入命令：show global variables like 'port';

# 重启服务 start stop 
service mysqld restart

#或者使用 
sudo /etc/init.d/mysql restart
```

### 备份数据库表结构

命令行下具体用法如下：  mysqldump -u用戶名 -p密码 -d 数据库名 表名 脚本名;

```
	# 1.导出整个数据库  
    # mysqldump -u用户名 -p密码  数据库名 > 导出的文件名 
　　mysqldump -uroot -pmysql sva_rec  > e:\sva_rec.sql 

　　# 2.导出一个表，包括表结构和数据 

　　# mysqldump -u用户名 -p 密码  数据库名 表名> 导出的文件名  http://zetcode.com/mysql/exportimport/
　　mysqldump -uroot -pmysql --no-create-info sva_rec date_rec_drv > e:\date_rec_drv.sql 

　　# 3.导出一个数据库结构 增加 -d 参数
　　mysqldump -uroot -pmysql -d sva_rec > e:\sva_rec.sql 

    # 4.导出一个表，只有表结构 

　　mysqldump -u用户名 -p 密码 -d数据库名  表名> 导出的文件名 
　　mysqldump -uroot -pmysql -d sva_rec date_rec_drv> e:\date_rec_drv.sql 
```
### 导入数据 

[导入\导出表结构或数据](https://www.cnblogs.com/zcw-ios/articles/3319480.html)
[mysql导入导出sql文件](https://www.cnblogs.com/yuwensong/p/3955834.html)
```
#常用source 命令
#进入mysql数据库控制台，如
mysql -u root -p
mysql>use 数据库
# 然后使用source命令，后面参数为脚本文件(如这里用到的.sql)
mysql>source path/to/dbname.sql
```

### 数据库查询时间转换

FROM_UNIXTIME
```
SELECT FROM_UNIXTIME(ii.match_time/1000), ii.* FROM `intervene_info` as ii 
	where ii.adapt_id in('xxx') and ii.tindex>0 and ii.swj_email='intervene.109@xxx.cn';
```

### 新增/修改表名、列名

[添加列，修改列，删除列](https://blog.csdn.net/ws84643557/article/details/6939846)

```
--修改表名 
alter table test rename test1; 

--添加表列 
alter table test add  column name varchar(10); 

-- 多个columns
ALTER TABLE su_result
	ADD COLUMN committer_email varchar(100) NOT NULL DEFAULT '' COMMENT '提交人email',
	ADD COLUMN bug int(4) NOT NULL DEFAULT 0 COMMENT '新增bug数',
	ADD COLUMN code_smell int(4) NOT NULL DEFAULT 0 COMMENT '新增异味数',
	ADD COLUMN vulnerability int(4) NOT NULL DEFAULT 0 COMMENT '新增漏洞数';

--删除表列 
alter table test drop  column name; 

--修改表列类型 
alter table test modify address char(10) 
--修改表列类型
alter table test change address address char(40) 

--修改表列名
alter table test change  column address address1 varchar(30)

-- 删除表
DROP TABLE  tbl_name;
-- 或者是
DROP TABLE IF EXISTS tbl_name;

-- 删除表记录
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name 

[WHERE where_definition] 

[ORDER BY ...] 

[LIMIT row_count] 

delete from friends;
```

### 拼接字段concat

[参考](https://blog.csdn.net/u012260238/article/details/70245452)
```
#将多个字段值拼接为新的值
select *, CONCAT(git_group ,'/', project_name) as name from ut_block where git_group = 'nr';
```

### 命令行 

#### 登录远程数据库

mysql -P 3306 -h 192.168.1.104 -u root -p

#### 查看表结构
show columns from customers; 

#### 查看记录
- select count(1) from table; 忽略所有列
- select count('') from table;-返回表的记录数
- select count(0) from table;-返回表的记录数
- select count(null) from table;-返回0