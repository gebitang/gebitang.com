+++
title = "PostgreSQL"
description = "postgresql basic"
tags = [
]
date = "2021-12-09"
topics = [
    "sql",
    "postgres",
    "tech"
]
toc = true
+++


聚合整理一下现有的内容。 


## 安装PostgreSQL debian

```
# 安装
$ apt -y install postgresql postgresql-contrib
#查看版本
$ /usr/lib/postgresql/10/bin/postgres -V
postgres (PostgreSQL) 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1)

```

安装完成后，默认会增加一个postgres用户，需要对该用户设置密码，并利用这个用户创建新的数据库用户提供给sonar使用——

## 安装PostgreSQL centOS

[官方手册](https://www.postgresql.org/download/linux/redhat/) 根据环境选择对应的配置，会自动提示对应的脚本——

```
# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL:
sudo yum install -y postgresql10-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
sudo systemctl enable postgresql-10
sudo systemctl start postgresql-10
```

执行到 初始化时出现权限问题——( ~~切换到root用户执行初始化以及systemctl命令~~ )这种方式有效，但不推荐。

```
 /usr/pgsql-10/bin/postgresql-10-setup initdb
Initializing database ... mkdir: cannot create directory ‘/var/lib/pgsql’: Permission denied
failed, see /var/lib/pgsql/10/initdb.log

#log 显示
runuser: may not be used by non-root users

# 初始化成功后，查看initdb.log可以看到以下内容

Success. You can now start the database server using:
    /usr/pgsql-10/bin/pg_ctl -D /var/lib/pgsql/10/data/ -l logfile start

```

正确的做法，根据官方文档[initdb](https://www.postgresql.org/docs/10/app-initdb.html)说明:

>Although `initdb` will attempt to create the specified data directory, it might not have permission if the parent directory of the desired data directory is root-owned. To initialize in such a setup, create an empty data directory as root, then use `chown` to assign ownership of that directory to the database user account, then `su` to become the database user to run initdb.

>initdb must be run as the user that will own the server process, because the server needs to have access to the files and directories that initdb creates. Since the server cannot be run as root, you must not run initdb as root either. (It will in fact refuse to do so.)

默认的数据库目录为`/var/lib/pgsql/10/data/`可以先使用root用户创建一个空的文件夹，再使用`chown`将此文件夹权限赋值给`postgres`用户，然后再执行initdb目录指定`-D `参数值为上述目录

[Custom PGDATA with systemd](https://pgstef.github.io/2018/02/28/custom_pgdata_with_systemd.html)

```
# mkdir -p /pgdata/10/data
# chown -R postgres:postgres /pgdata
```

Then, customize the systemd service: `systemctl edit postgresql-10.service` Add the following content:
```
[Service]
Environment=PGDATA=/pgdata/10/data
```

This will create a
`/etc/systemd/system/postgresql-10.service.d/override.conf ` file which will be merged with the original service file.

To check its content:
```
# cat /etc/systemd/system/postgresql-10.service.d/override.conf
[Service]
Environment=PGDATA=/pgdata/10/data
```

Reload systemd:
```
# systemctl daemon-reload
```

Initialize the PostgreSQL data directory:
```
# /usr/pgsql-10/bin/postgresql-10-setup initdb
```
Start and enable the service:
```
# systemctl enable postgresql-10
# systemctl start postgresql-10
```


## PostgreSQL远程登录

使用正确的用户名、密码却一直无法登陆。[因为——]((https://confluence.atlassian.com/bitbucketserverkb/fatal-ident-authentication-failed-for-user-unable-to-connect-to-postgresql-779171564.html))

>By default PostgreSQL uses IDENT-based authentication and this will never allow you to login via -U and -W options.


[Connecting to a Remote PostgreSQL Database](https://www.netiq.com/documentation/identity-manager-47/setup_windows/data/connecting-to-a-remote-postgresql-database.html)

默认，PostgreSQL只监听本地连接，不允许远程通过TCP/IP链接，运行远程连接需要在Server端修改两个配置文件；` postgresql.conf `和` pg_hba.conf `文件。

- ` postgresql.conf `中修改监听配置，修改为 `listen_addresses = '*'` 
- ` pg_hba.conf `修改权限配置，`host all all 0.0.0.0/0 md5`
- 修改完成后，需要重启数据库才能生效。 `sudo systemctl restart postgresql-10.service`

这两个配置文件的位置可以在数据库中使用postgres用户通过以下sql进行查询：

- `show config_file; `显示 ` postgresql.conf `文件的具体位置
- `show hba_file; `显示 ` pg_hba.conf  `文件的具体位置

```
postgres=# show config_file;
              config_file
----------------------------------------
 /var/lib/pgsql/10/data/postgresql.conf
(1 row)

postgres=# show hba_file;
              hba_file
------------------------------------
 /var/lib/pgsql/10/data/pg_hba.conf
(1 row)

postgres=#

```


## PostgreSQL基础用法

- 安装数据库

```
$ apt -y update
$ apt -y install postgresql postgresql-contrib
...
..
.
$ /usr/lib/postgresql/10/bin/postgres -V
postgres (PostgreSQL) 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1)
```

- 数据库配置

`vim /etc/postgresql/10/main/postgresql.conf`
重启数据库 `systemctl restart postgresql`

- 基础用法


### 创建新用户

```
#为postgres用户设置密码
passwd postgres
# 切换到postgres用户
su - postgres
# 新建postgres数据库用户 sonar
createuser sonar
# 登录数据库
psql
# 执行数据库操作：设置用户密码
ALTER USER sonar WITH ENCRYPTED password 'sonar';
# 创建数据库sonar
CREATE DATABASE sonar WITH ENCODING 'UTF8' OWNER sonar TEMPLATE=template0;
# 退出数据库连接
\q
```

### 修改用户密码

[修改postgres用户密码：](https://stackoverflow.com/a/12721095/1087122) 

```
# switch to user to login
sudo -u user_name psql db_name

#change password
ALTER USER user_name WITH PASSWORD 'new_password';
```


### 常用操作

[How to List Databases and Tables in PostgreSQL Using psql](https://chartio.com/resources/tutorials/how-to-list-databases-and-tables-in-postgresql-using-psql/)

1. 切换到默认数据库管理员用户 postgres ` sudo -i -u postgres`；
2. `psql`命令连接进入数据库
3. 默认提示符为 `username=#`；
4. 使用`createdb dbname`创建数据库
5. `\conninfo`查看连接信息
6. `\i paht\to\schema.sql` 执行sql语句
7. `\l`查看当前数据库下的表
8. `\c dbname`连接到数据库
9. `\dt`显示当前数据的表
10. `SELECT pg_size_pretty( pg_database_size('dbname') );` 查看数据库大小，或者使用 `\l+ dbname`， 使用 `\l`默认[显示所有数据库](https://stackoverflow.com/a/23990410/1087122)
11. `SELECT pg_size_pretty(pg_relation_size('table_name'));` 查看当前连接的数据库里的表的大小
12. `\d+` 显示当前数据库的表的大小
13. 按照大小排列当前数据库的表大小

```
# https://stackoverflow.com/a/21738505/1087122
#  the size of all tables in the schema public 
select table_name, pg_relation_size(quote_ident(table_name))
from information_schema.tables
where table_schema = 'public'
order by 2 desc
```

```
➜  ~ sudo -i -u postgres
postgres@iZ2zebob74margeal4esovZ:~$
postgres@iZ2zebob74margeal4esovZ:~$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.


postgres=# help
You are using psql, the command-line interface to PostgreSQL.
Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit
postgres=#

```


## postgresql数据库备份恢复

使用默认管理员账号`postgres`

备份：  
- dump数据库：`pg_dump -F c -b -v -f backup/sonar.backup sonar` 

另外一台机器：  
- 删除旧数据库，新建同名数据库： `dropdb sonar; createdb sonar`
- 恢复数据库: `pg_restore -d sonar sonar.backup` 

恢复时提示`could not execute query: ERROR:  must be owner of extension plpgsql`，log信息类似如下——

```
sonar@stf:~$ pg_restore -d sonar sonarmain.backup
pg_restore: [archiver (db)] Error while PROCESSING TOC:
pg_restore: [archiver (db)] Error from TOC entry 4332; 0 0 COMMENT EXTENSION plpgsql
pg_restore: [archiver (db)] could not execute query: ERROR:  must be owner of extension plpgsql
    Command was: COMMENT ON EXTENSION plpgsql IS 'PL/pgSQL procedural language';

WARNING: errors ignored on restore: 1

```

参考[Seeing error "Must be owner of extension plpgsql" during a Postgres database restore.](https://knowledge.broadcom.com/external/article/4307/seeing-error-must-be-owner-of-extension.html)，大部分场景下可以忽略这个错误（实测上述报错不影响导入结果，包含1213个项目的数据库重新连接后ES基于数据库重建索引成功）。因为尝试导入自身无权限的数据，可以使用`-n public`参数忽略。参考[PostgreSQL 9.1 pg_restore error regarding PLPGSQL](https://stackoverflow.com/a/11776053/1087122)

```
# pg_restroe --help查看参数含义： -c 对应的数据库如果存在内容，先删除内容； -n 只导入指定的schema；
pg_restore -U username -c -n public -d database_name

```

提示这个错误的原因是因为执行`pg_dump`导出时，创建了schema`plpgsql`，参考[How to solve privileges issues when restore PostgreSQL Database
](https://stackoverflow.com/questions/13410631/how-to-solve-privileges-issues-when-restore-postgresql-database/15452741)

```
# actual content of pg_dump
CREATE SCHEMA public;
COMMENT ON SCHEMA public IS 'standard public schema';
CREATE EXTENSION IF NOT EXISTS plpgsql WITH SCHEMA pg_catalog;
COMMENT ON EXTENSION plpgsql IS 'PL/pgSQL procedural language';
```


如果版本不一致，可能涉及到升级问题，查看sonar.log即可
```
...
2020.10.21 11:14:27 ERROR web[][o.s.s.p.PlatformImpl] Web server startup failed: Database was upgraded to a more recent version of SonarQube. A backup must probably be restored or the DB settings are incorrect.
2020.10.21 11:14:27 INFO  web[][o.s.s.a.EmbeddedTomcat] HTTP connector enabled on port 9000
2020.10.21 11:14:27 INFO  web[][o.s.p.ProcessEntryPoint] Hard stopping process
...

```
备份数据库在SQ8.4版本，恢复在8.3版本时报错：` Database was upgraded to a more recent version of SonarQube. A backup must probably be restored or the DB settings are incorrect.`

使用相同的SQ，可正常启动——备份恢复成功。


## 卸载PostgreSQL

[How To Completely Uninstall PostgreSQL](https://kb.objectrocket.com/postgresql/how-to-completely-uninstall-postgresql-757)

- Debian Linux, 例如Ubuntu

卸载主程序——
```
sudo apt-get --purge remove postgresql
sudo apt-get purge postgresql*
sudo apt-get --purge remove postgresql postgresql-doc postgresql-common
```
查找其他相关包：`dpkg -l | grep postgres`，然后再进行删除 `sudo apt-get --purge remove related-pkg-name`

删除对应的数据库、log文件夹：

```
sudo rm -rf /var/lib/postgresql/
sudo rm -rf /var/log/postgresql/
sudo rm -rf /etc/postgresql/
```

- Fedora Linux，例如 CentOS

直接删除：`yum remove postgresql`  
删除相关：`yum remove postgres\* `  
删除数据目录：`rm /var/lib/pgsql`

查找安装的其他相关包： `yum list installed | grep postgres`（也可以使用`rpm -qa | grep postgres`进行查找）然后进行删除 `yum remove -p related-pkg-name`


应用包的卸载操作都是类似的——，例如卸载java11 

```
# 卸载java11 
sudo yum remove -y java-11-openjdk-devel 
```
