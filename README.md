# MySQL
MySQL重要概念与常用语句

## 一、安装
### Ubuntu
>Ubuntu（友帮拓、优般图、乌班图）：是一个以桌面应用为主的开源GNU/Linux操作系统。Ubuntu 是基于Debian GNU/Linux，支持x86、amd64（即x64）和ppc架构，由全球化的专业开发团队（Canonical Ltd）打造的。

**在Ubuntu下安装mysql**

1.首先执行下面三条命令：
```
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
2.执行完成后可以通过下面的命令测试是否安装成功：

```
sudo netstat -tap | grep mysql
```

3.通过如下命令进入mysql服务：

```
mysql -uroot -p密码
```

详见[Ubuntu安装mysql教程](https://blog.csdn.net/xiangwanpeng/article/details/54562362
)

---

### CentOS
>CentOS（Community Enterprise Operating System，社区企业操作系统）：是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS完全开源。

**在CentOS下安装mysql**

1.首先检查系统是否装有mysql:

```
rpm -qa | grep mysql
```

若返回空值，说明没有安装；若已经装有mysql，通过以下命令删除可用：

```
yum remove mysql
```

注：这里执行安装命令是无效的，因为centos默认是Mariadb，执行以下命令只是更新Mariadb数据库

```
yum install mysql
```
2.下载mysql的repo源:

```
# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

安装`mysql-community-release-el7-5.noarch.rpm包`：

```
# sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

安装这个包后，会获得两个mysql的`yum repo源`：/etc/yum.repos.d/mysql-community.repo,/etc/yum.repos.d/mysql-community-source.repo。

3.安装mysql

```
# sudo yum install mysql-server
```

详见[CentOS安装mysql教程](https://blog.csdn.net/a774630093/article/details/79270080)

---

## 二、启动服务与连接
1.启动mysql服务器：

```
sudo service mysql start
```

2.连接服务器：

```
mysql -u root 密码
```

---

## 三、数据库管理
### 用户管理
1.选择数据库

```
mysql>use mysql;
```

2.查看该数据库中所有用户

```
mysql>select host,user,password from user;
```

3.创建新用户

```
mysql>create user test IDENTIFIED by 'xxxxx';//identified by会将纯文本密码加密作为散列值存储
```

4.修改用户名

```
mysql>rename user feng to newuser;//mysql5之后可以使用，之前需要使用update更新user表
```

5.删除用户

```
mysql>drop user newuser;//mysql5之前删除用户时必须先使用revoke删除用户权限，然后删除用户
```

---

### 权限设置
1.查看用户权限

```
mysql>show grants for test;
```

2.赋予用户新权限

```
mysql>grant select on dmc_db.* to test;
```

3.回收权限

```
mysql>revoke select on dmc_db.* from test;//如果权限不存在会报错
```

**注：上面的命令也可以使用多个权限同时赋予和回收，权限之间使用逗号分隔**

```
mysql>grant select,update,delete,insert on dmc_db.* to test;
```

4.刷新权限

```
mysql>flush privileges;
```

---

### 日志
#### 日志分类

在MariaDB/MySQL中，主要有5种日志文件：
- 错误日志（error log）：记录mysql服务的启停时正确和错误的信息，还记录启动、停止、运行过程中的错误信息。
+ 查询日志（general log）：记录建立的客户端连接和执行的语句。
- 二进制日志（bin log）：记录所有更改数据的语句，可用于数据复制。
+ 慢查询日志（slow log）：记录所有执行时间超过long_query_time的所有查询或不使用索引的查询。
- 中继日志（relay log）：主从复制时使用的日志。
除了这5种日志，在需要时还会创建DDL日志。

#### 日志刷新操作
以下操作会刷新日志文件，刷新日志文件时会关闭旧的日志文件并重新打开日志文件。对于有些日志类型，如二进制日志，刷新日志会滚动日志文件，而不仅仅是关闭并重新打开。
```
mysql>FLUSH LOGS;
shell>mysqladmin flush-logs
shell>mysqladmin refresh
```

#### 错误日志
错误日志是最重要的日志之一，它记录了MariaDB/MySQL服务启动和停止正确和错误的信息，还记录了mysqld实例运行过程中发生的错误事件信息。

可以使用`--log-error=[file_name]`来指定mysqld记录的错误日志文件，若没有指定file_name，则默认的错误日志文件在datadir目录下的'hostname'.err，hostname表示当前的主机名。

也可以在MariaDB/MySQL配置文件中的mysqld配置部分，使用log-error指定错误日志的路径。

若不知道错误日志的位置，可以使用变量`log_error`来查看:

```
mysql>show variables like 'log_error';
```

在MySQL5.5.7之前，刷新日志操作（如flush logs）会被备份旧的错误日志（以_old结尾），并创建一个新的错误日志并打开。在MySQL5.5.7之后，执行刷新日志的操作时，错误日志会关闭并重新打开。如果错误日志不存在，则会先创建。

#### 一般查询日志
查询日志氛围一般查询日志和慢查询日志，他们是通过查询是否超出变量long_query_time指定时间的值来判定的。在超时时间内完成的查询是一般查询，可以将其记录到一般查询日志中，**但是建议关闭这种日志（默认是关闭的）**，超出时间的查询是慢查询，可以将其记录到慢查询日志中。

使用`--general_log={0|1}`来决定是否启用一般查询日志，使用`--general_log_file=file_name`来指定查询日志的路径。不给定路径是默认的文件名以'hostname'.log命名。

和查询日志有关的变量有：
```
long_query_time=10 # 指定慢查询超时时长，超出此时长的属于慢查询，会记录到慢查询日志中
log_output={TABLE|FILE|NONE} # 定义一般查询日志和慢查询日志的输出格式，不指定时默认为file
```

Table表示记录日志到表中，FILE表示记录日志到文件中，NONE表示不记录日志。只要这里指定为NONE，即使开启了一般查询日志和慢查询日志，也都不会有任何记录。

和一般查询日志有关的变量有：
```
general_log=off # 是否启用一般查询日志，为全局变量，必须在global上修改
sql_log_off=off # 在session级别控制是否启用一般查询日志，默认为off，即启用
general_log_file=/mydata/data/hostname.log # 默认是库文件路径下主机名加上.log
```
默认没有开启一般查询日志，也不建议开启一般查询日志
开启一般查询日志使用以下命令：

```
mysql>set @@global.general_log=1;
```

一般查询日志记录的不止是SELECT语句，几乎所有的语句都会记录。

#### 慢查询日志
查询超出（等于不会记录）变量long_query_time指定时间值的为慢查询，但是查询获取锁（包括锁等待）的时间不计入查询时间内。

mysql记录慢查询日志是在查询执行完毕且已经完全释放锁之后才记录的，因此慢查询日志记录的顺序和执行的SQL查询语句顺序可能会不一致（例如语句1先执行，查询速度慢，语句2后执行，但查询速度快，则语句2先记录）

和慢查询有关的变量：
```
long_query_time=10 # 指定慢查询超时时长（默认10秒），超出此时长的属于慢查询
log_output={TABLE|FILE|NONE} # 定义一般查询日志和慢查询日志的输出格式，不指定时默认为file
log_slow_queries={yes|no} # 是否启用慢查询日志，默认不启用
slow_query_log={1|ON|0|OFF} # 是否启用慢查询日志，此变量和log_slow_queries修改一个 另一个同时变化
slow_query_log_file=/mydata/data/hostname-slow.log # 默认路径为库文件目录下主机名加上-slow.log
log_queries_not_using_indexes=OFF # 查询没有使用索引的时候是否也记入慢查询日志
```
随着时间的推移，慢查询日志文件中的记录可能会变得非常多，这对于分析查询来说是非常困难的 => 专门归类慢查询日志的工具mysqldumpslow。

参数 | 解释
----|----
-d | debug
-v | verbose:显示详细信息
-t NUM | just show the top n queries:仅显示前n条查询
-a | don't abstract all numbers to N and strings to 'S':归类是不要使用N替换数字，S替换字符串
-g PATTERN | grep:only consider stmts that include this string:通过grep来筛选select语句

该工具归类时，默认会将同文本但变量值不同的查询语句视为同一类，并使用N代替其中的数值变化，使用S代替其中的字符串变量，可以使用`-a`来禁用这种替换。

归类后的结果只是精确到0.01秒，如果要显示极其精确的秒数，使用`-d`参数启用调试功能

#### 二进制日志

（1）二进制日志文件

二进制日志包含了引起或可能引起数据库改变（如delete语句但没有匹配行）的事件信息，但绝不会包括select和show这样的查询语句。语句以“事件”的形式保存，所以包含了时间、事件开始和结束的位置等信息。

二进制日志是**以事件形式记录的**，不是事务日志（但可能是基于事务来记录二进制日志），不代表它只记录innodb日志，myisam表也一样有二进制日志。

二进制日志**只在事务提交的时候一次性写入**。

MariaDB/MySQL默认不启动二进制日志，要启动二进制日志有两种方法：

①使用`--log-bin=[on|off|file_name]`选项指定，若没有给定file_name,则默认为datadir下的主机名加`-bin`，并在后面跟上一串数字表示日志序列号，如果给定的日志文件中包含了后缀（logname.suffix）将忽略后缀部分。

②在配置文件中的[mysqld]部分设置`log-bin`。

注：对于mysql5.7，直接启动binlog可能会导致mysql服务启动失败，这是需要在配置文件中的mysqld为mysql实例分配server_id：
```
[mysqld]
# server_id=1234
log-bin=[on|filename]
```
mysqld还创建一个**二进制日志索引文件**，当二进制日志文件滚动的时候会向该文件写入对应的信息。所以该文件包含所有使用的二进制日志文件的文件名。默认情况下，该文件与二进制日志文件的文件名相同，扩展名为'.index'。要制定该文件的文件名使用`--log-bin-index[=file_name]`选项。当mysqld在运行时不应手动编辑该文件，免得mysqld变得混乱。

当重启mysql服务或刷新日志或达到日志最大值时，将滚动二进制日志文件，滚动日志时只修改日志文件名的数字序列部分。

二进制日志文件的最大值通过变量`max_binlog_size`设置（默认值为1G）。但由于二进制日志可能是基于事务来记录的（如innodb表类型），二十五是绝对不可能也不应该跨文件记录的，如果正好二进制日志文件达到了最大值但事务还没有提交，则不会滚动日志，而是继续增大日志，所以`max_binlog_size`指定的值和实际的二进制日志大小不一定相等。

因为二进制日志文件增长迅速，但官方说明因此而损耗的性能小于1%，且二进制目的是为了恢复定点数据库和主从复制，所以出于安全和功能考虑，**既不建议将二进制日志和datadir放在同一磁盘上**。

（2）查看二进制日志
MySQL中查看二进制日志的方法主要有以下几种：
①使用`mysqlbinlog`工具
②使用`show`显示对应的信息
```
SHOW {BINARY | MASTER} LOGS # 查看使用了哪些日志文件
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] # 查看日志中进行了哪些操作
SHOW MASTER STATUS # 显示主服务器中的二进制日志信息
```

（3）删除二进制日志
删除二进制日志有几种方法，不管哪种方法，都会将删除后的信息同步到二进制index文件中。
①`reset master`将会删除所有日志，并让日志文件重新从000001开始

```
mysql>reset master;
```

②`PURGE {BINARY|MASTER}LOGS{TO 'log_name'|BEFORE datetime_expr}`

`purge master logs to "binlog_name.00000X"`将会清空00000X之前的所有日志文件。例如删除000006之前的日志文件：

```
mysql>purge master logs to "mysql-bin.000006";
mysql>purge binary logs to "mysql-bin.000006";
```

master和binary是同义词

`purge master logs before 'yyyy-mm-dd hh:mi:ss'`将会删除指定日期之前的所有日志。但是若指定的时间处在正在使用中的日志文件中，将无法进行purge。

```
mysql>purge master logs before '2017-03-29 07:36:40';
mysql>show warnings;
```

③使用`--expire_logs_days=N`选项指定过了多少天日志自动过期清空

详见[MySQL日志](https://blog.csdn.net/eagle89/article/details/80969565)

---

### 备份
#### 逻辑备份
MySQL中的逻辑备份试讲数据库中的数据备份为一个文本文件，备份的文件可以被查看和编辑。在MySQL中，可以使用mysqldump工具来完成逻辑备份：

```
//备份指定的数据库或者数据库中的某些表
shell>mysqldump [options] db_name [tables]

//备份指定的一个或多个数据库
shell>mysqldump [options] --database DB1 [DB2,DB3...]

//备份所有数据库
shell>mysqldump [options] --all-database
```

如果没有指定数据库中的任何表，默认导出所有数据库中的所有表

**示例**

1.备份所有数据库：
```
shell>mysqldump -uroot -p --all-database > all.sql
```
2.备份数据库test
```
shell>mysqldump -uroot -p test > test.sql
```
3.备份数据库test下的表emp
```
shell>mysqldump -uroot -p test emp > emp.sql
```
4.备份数据库test下的表emp和dept
```SQL
shell>mysqldump -uroot -p test emp dept > emp_dept.sql
```
5.备份数据库test下的所有表为逗号分割的文本，备份到/tmp
```
shell>mysqldump -uroot -p -T /tmp test emp --fields-terminated-by ','
shell>more emp.txt
```
#### 物理备份
物理备份又分为冷备份和热备份两种，和逻辑备份相比，它的最大优点是**备份和恢复的速度更快**，因为物理备份的原理都是基于文件的cp。
##### 冷备份
冷备份其实就是停掉数据库服务、cp数据文件的方法（基本不考虑这种方法）。
##### 热备份
在MySQL中，对于不同的存储引擎，热备份的方法也有所不同。

1.myisam存储引擎

myisam存储引擎的热备份有很多方法，本质其实就是将要备份的表加读锁，然后再cp数据文件到备份目录。常用的有以下两种方法：

（1）使用mysqlhotcopy工具
```
//mysqlhotcopy是MySQL的一个自带的热备份工具
shell>mysqlhotcopy db_name [/path/to/new_directory]
```
（2）手工锁表copy
```
//在mysqlhotcopy使用不正常的情况下，可以用手工来做热备份
mysql>flush tables for read;
//cp数据文件到备份目录即可
```
2.innodb存储引擎

使用第三方工具ibbackup,xtrabackup,innobackupex

详见[MySQL备份](https://juejin.im/entry/5a0aa2026fb9a045132a369f)

---

## 四、库操作
### 查看、选择库
#### 查看库

查看当前服务器上存在什么数据库：**SHOW语句**

```
mysql>SHOW DATABASES
```

#### 选择库

选定使用数据库：**USE语句**

```
mysql>use test;
```

### 新建数据库

新建数据库：**CREATE语句**

```
mysql>CREATE DATABASE test;
```

### 删除数据库

删除数据库：**DROP语句**

```
mysql>DROP test;
```
---

## 五、表操作
### 查看表结构
### 数据结构
### 新建表
### 删除表
### 修改表结构

## 六、数据操作
### 插入数据
### 修改数据
### 删除数据

## 七、视图

## 八、SQL语法
