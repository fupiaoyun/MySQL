# MySQL
MySQL重要概念与常用语句

## 目录
1. [安装](#安装)
    - [Ubuntu](#Ubuntu)
    + [CentOS](#CentOS)
2. [启动服务与连接](#启动服务与连接)
3. [数据库管理](#数据库管理)
    - [用户管理](#用户管理)
    + [权限设置](#权限设置)
    - [日志](#日志)
    + [备份](#备份)
4. [库操作](#库操作)
    - [查看数据库](#查看数据库)
    + [选择数据库](#选择数据库)
    - [新建数据库](#新建数据库)
    + [删除数据库](#删除数据库)
5. [表操作](#表操作)
    - [查看表结构](#查看表结构)
    + [数据模型](#数据模型)
    - [新建表](#新建表)
    + [删除表](#删除表)
    - [修改表结构](#修改表结构)
6. [数据操作](#数据操作)
    - [插入数据](#插入数据)
    + [修改数据](#修改数据)
    - [删除数据](#删除数据)
7. [视图和索引](#视图和索引)
    - [视图](#视图)
    + [索引](#索引)
8. [SQL语法](#SQL语法)
9. [数据库引擎](#数据库引擎)
10. [MySQL及SQL注入](#SQL注入)

---

## <span id="安装">一、安装</span>

### <span id="Ubuntu">Ubuntu</span>
>Ubuntu（友帮拓、优般图、乌班图）：是一个以桌面应用为主的开源GNU/Linux操作系统。Ubuntu 是基于Debian GNU/Linux，支持x86、amd64（即x64）和ppc架构，由全球化的专业开发团队（Canonical Ltd）打造的。

**在Ubuntu下安装mysql**

1.首先执行下面三条命令：

```shell
$ sudo apt-get install mysql-server
$ sudo apt install mysql-client
$ sudo apt install libmysqlclient-dev
```

2.执行完成后可以通过下面的命令测试是否安装成功：

```shell
$ sudo netstat -tap | grep mysql
```

3.通过如下命令进入mysql服务：

```shell
$ sudo service mysql start
$ mysql -uroot -p密码
```

详见[Ubuntu安装mysql教程](https://blog.csdn.net/xiangwanpeng/article/details/54562362
)

---

### <span id="CentOS">CentOS</span>
>CentOS（Community Enterprise Operating System，社区企业操作系统）：是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS完全开源。

**在CentOS下安装mysql**

1.首先检查系统是否装有mysql：

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

2.下载mysql的repo源：

```
# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

安装`mysql-community-release-el7-5.noarch.rpm包`：

```
# sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

安装这个包后，会获得两个mysql的`yum repo源`：/etc/yum.repos.d/mysql-community.repo,/etc/yum.repos.d/mysql-community-source.repo。

3.安装mysql：

```
# sudo yum install mysql-server
```

详见[CentOS安装mysql教程](https://blog.csdn.net/a774630093/article/details/79270080)

---

## <span id="启动服务与连接">二、启动服务与连接</span>
1.启动mysql服务器：

```shell
$ sudo service mysql start
```

2.连接服务器：

```shell
$ mysql -u root 密码
```

---

## <span id="数据库管理">三、数据库管理</span>
### <span id="用户管理">用户管理</span>
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

### <span id="权限设置">权限设置</span>
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

### <span id="日志">日志</span>
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

`purge master logs before 'yyyy-mm-dd hh:mm:ss'`将会删除指定日期之前的所有日志。但是若指定的时间处在正在使用中的日志文件中，将无法进行purge。

```
mysql>purge master logs before '2017-03-29 07:36:40';
mysql>show warnings;
```

③使用`--expire_logs_days=N`选项指定过了多少天日志自动过期清空

详见[MySQL日志](https://blog.csdn.net/eagle89/article/details/80969565)

---

### <span id="备份">备份</span>
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

## <span id="库操作">四、库操作</span>
### <span id="查看数据库">查看数据库</span>

查看当前服务器上存在什么数据库：**SHOW语句**

```SQL
SHOW DATABASES;
```

### <span id="选择数据库">选择数据库</span>

选定使用数据库：**USE语句**

```SQL
USE test;
```

### <span id="新建数据库">新建数据库</span>

新建数据库：**CREATE语句**

```SQL
CREATE DATABASE test;
```

### <span id="删除数据库">删除数据库</span>

删除数据库：**DROP语句**

```SQL
DROP test;
```
---

## <span id="表操作">五、表操作</span>
### <span id="查看表结构">查看表结构</span>
**1.describe(desc) 表名**

describe用于查看特定表的详细设计信息

```SQL
desc empt;
//或者
describe empt;
```

**2.show columns from 表名**

用于查询出表的列信息

```SQL
show columns from empt;
```

**3.show create table 表名**

用于查询建表语句

```SQL
show create table empt;
```

**4.使用`information_schema`数据库**

`information_schema`数据库是MySQL自带的，它提供了访问数据库元数据的方式。

> 元数据：是关于数据的数据，如数据库名或表名、列的数据类型或访问权限等。用于表述该信息的其他术语包括“数据词典”和“系统目录”。

```SQL
use information_schema;
select * from columns where table_name='empt';
```
---

### <span id="数据模型">数据模型</span>
#### 数据模型的分类
最常用的数据模型是概念数据模型和结构数据模型：
- 概念数据模型（信息模型）：面向用户的，按照用户的观点进行建模，典型代表：E-R图
+ 结构数据模型：面向计算机系统的，用于DBMS的实现，典型代表有：层次模型、网状模型、关系模型、面向对象模型。

#### 数据模型的三要素
数据结构、数据操作、数据约束

#### E-R图（实体-联系图）
E-R实体联系图是直观表示概念模型的工具，其中包含了实体、联系、属性三个成分，联系的方法为一对一（1:1）、一对多（1：N）、多对多（M：N）三种方式，联系属于哪种方式取决于客观实际本身

E-R模型图，既表示实体，也表示实体之间的联系，是现实世界的抽象，与计算机系统没有关系，是可以被用户理解的数据描述方式。通过E-R模型图也可以使用户了解系统设计者对现实世界的抽象是否符合实际情况，从某种程度上说，E-R模型图也是用户与系统设计者进行交流的工具，E-R模型图已经成为概念模型设计的一个重要设计方法。

实体用矩形框表示，联系用菱形表示，属性用椭圆表示。

#### 层次模型
层次模型采用树形结构表示数据与数据之间的关系。

层次模型不能直接表示多对多的联系、

#### 网状模型
用网络结构表示数据与数据之间的联系的模型。

网状模型子节点与父节点联系不唯一，需要为联系命名。

网状模型的优点是更直观、性能更好，缺点是结构复杂。

#### 关系模型
关系模型是目前最常见的数据模型之一，主要采用表格结构表达实体集以及实体之间的联系，最大的特色就是描述的一致性。

关系是一张表，关系数据模型由若干个表组成。

可以存在1对1，1对多，多对多的关系。

##### 基本术语
- **关系（Relation）**：一个关系对应着一个二维表，二维表就是关系名。
+ **元组（Tuple）**：在二维表中的一行，称为一个元组。
- **属性（Attribute）**：在二维表中的列，称为属性。属性的个数称为关系的元或度。列的值为属性值。
+ **域（值，Domain）**：属性值的取值范围为值域。
- **分量**：每一行对应的列的属性值，即元组中的一个属性值。
+ **关系模式**：在二维表中的行定义，即对关系的描述称为关系模式。一般表示为（属性1，属性2，……，属性n），如老师的关系模型可以表示为教师（教师号，姓名，性别，出生年月，职称，所在系）。
- **键（码）**：如果在一个关系中存在唯一标识一个实体的一个属性或属性集，则称为实体的键，即使得在该关系上的任何一个关系状态的两个元组，在该属性上的值的组合都不同。
+ **候选键（候选码）**：若关系中的某一属性能唯一标识一个元组，则称这个属性为该关系的候选键或候选码。

---

### <span id="新建表">新建表</span>
CREATE TABLE语句

```SQL
CREATE TABLE teacher(t_no int(10),t_name varchar(20),sex char(1),birth DATE,title varchar(20),dept varchar(20));
```

### <span id="删除表">删除表</span>
DROP TABLE语句

```SQL
DROP TABLE teacher;
```

### <span id="修改表结构">修改表结构</span>
ALTER TABLE语句
#### 添加字段：ADD
FIRST：放到表的首位
AFTER：放到某个字段后面

```SQL
ALTER TABLE tabName ADD 字段名称 字段属性 [完整性约束条件] [FIRST|AFTER 字段名称];
```
示例

```SQL
ALTER TABLE user ADD addr VARCHAR(50);
```

#### 删除字段：DROP

```SQL
ALTER TABLE tabName DROP 字段名称;
```
示例

```SQL
ALTER TABLE user DROP addr;
```

#### 给字段添加默认值：SET DEFAULT

```SQL
ALTER TABLE tabName ALTER 字段名称 SET DEFAULT 默认值;
```
示例

```SQL
ALTER TABLE user ALTER name SET DEFAULT 'xiaoming';
```

#### 删除默认值：DROP DEFAULT

```SQL
ALTER TABLE tabName ALTER 字段名称 DROP DEFAULT;
```
示例

```SQL
ALTER TABLE user ALTER name DROP DEFAULT;
```

#### 修改字段类型和字段属性：MODIFY

```SQL
ALTER TABLE tabName MODIFY 字段名称 字段类型 [字段属性] [FIRST|ALTER 字段名称];
```
示例

```SQL
//修改字段的字段类型和字段属性
ALTER TABLE user MODIFY id INT AUTO_INCREMENT KEY;
```

#### 修改字段名称、字段类型、字段属性：CHANGE
与MODIFY相比，CHANGE可以修改字段的名称

```SQL
ALTER TABLE tabName CHANGE 原字段名称 新字段名称 字段类型 字段属性 [FIRST|ALTER 字段名称];
```
示例

```SQL
//修改字段名称、字段类型和字段属性
ALTER TABLE user CHANGE name username CHAR(20) NOT NULL FIRST;
```

#### 添加主键：ADD PRIMARY KEY

```SQL
ALTER TABLE tabName ADD PRIMARY KEY（字段名称）;
```
示例

```SQL
//添加主键
ALTER TABLE user ADD PRIMARY KEY（id);
```

#### 删除主键：DROP PRIMARY KEY

```SQL
ALTER TABLE tabName DROP PRIMARY KEY;
```
示例

```SQL
//删除主键
ALTER TABLE user DROP PRIMARY KEY;
```

#### 修改表名称：RENAME

```SQL
ALTER TABLE tabName RENAME [TO|AS] newTabName;
//或
RENAME TABLE tabName TO newTabName;
```
示例

```SQL
//修改表名称
RENAME TABLE user TO user1;
```
---

## <span id="数据操作">六、数据操作</span>
### <span id="插入数据">插入数据</span>
#### 将文本文件装载到表中：LOAD DATA语句

```
mysql>LOAD DAATA INFILE '文件路径' INTO TABLE teacher;
```

**注**：若使用Windows中的编辑器（使用\r\n作为行的换行符）创建文件，应使用：

```SQL
LOAD DATA INFILE '文件路径' INTO TABLE pet LINES TERMINATED BY '\r\n';
//在运行 OS X 的苹果电脑上，应使用行结束符\r
```

自定义字段间分隔符为逗号：

```SQL
LOAD DATA INFILE '文件路径' INTO TABLE pet FIELDS TERMINATED BY ',';
```

#### 一次增加一个新纪录：INSERT语句

```SQL
INSERT INTO teacher VALUES('2220152470','xiaoming','M','1974-09-26','NULL','History');
```

**注**：这里字符串和日期均为引号括起来的字符串。另外，可以直接用`INSERT`语句插入`NULL`代表不存在的值。

### <span id="修改数据">修改数据：UPDATE语句</span>

```SQL
UPDATE table_name SET field1=new-value1,field2=new-value2 [WHERE Clause];
```

**注**：可以同时更新一个或多个字段。

### <span id="删除数据">删除数据：DELETE语句</span>

```SQL
DELETE FROM table_name [WHERE Clause];
```

**注**：如果没有指定WHERE子句，MySQL表中所有记录将被删除。

---

## <span id="视图和索引">七、视图和索引</span>
### <span id="视图">视图</span>
> 视图（View）：是从一个或多个表中导出的表，是一种虚拟存在的表。通过视图，用户可以不用看到整个数据库中的数据，而只关心对自己有用的数据。

**注**：
- 数据库只存放了视图的定义，而没有存放视图中的数据，这些数据存放在原来的表中。
+ 使用视图查询数据时，数据库系统会从原来的表中取出对应的数据。
- 视图中的数据依赖于原来表中的数据，一旦表中数据发生改变，显示在视图中的数据也会发生改变。
+ 在使用视图时，可以把它当做一张表。

**创建视图格式：**

```SQL
CREATE VIEW 视图名（列a,列b,列c） AS SELECT 列1，列2，列3 FROM 表名;
```
---

### <span id="索引">索引</span>
> 索引（Index）：在关系数据库中，索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。

**注**：
- 用户无法看到索引，它们只能被用来加速搜索/查询。
+ 更新一个包含索引的表比更新一个没有索引的表需要更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上创建索引。

**创建索引格式：**

```SQL
CREATE INDEX index_name ON table_name(column_name);
```
---

## <span id="SQL语法">八、SQL语法</span>
### SQL DML和DDL
可以把SQL分为两个部分：数据操作语言（DML）和数据定义语言（DDL）。

SQL（结构化查询语言）是用于执行查询的语法。但是SQL语言也包含用于更新、插入和删除记录的语法。

#### DML
查询和更新指令构成了SQL的DML部分：
- SELECT：从数据库表中获取数据
+ UPDATE：更新数据库表中的数据
- DELETE：从数据库表中删除数据
+ INSERT INTO：向数据库表中插入数据

#### DDL
SQL的数据定义语言（DDL）部分使我们有能力创建或删除表格。也可以定义索引（键），规定表之间的连接，以及施加表间的约束。

SQL中最重要的DDL语句：
- CREATE DATABASE：创建新数据库
+ ALTER DATABASE：修改数据库
- CREATE TABLE：创建新表
+ ALTER TABLE：变更（改变）数据库表
- DROP TABLE：删除表
+ CREATE INDEX：创建索引（搜索键）
- DROP INDEX：删除索引

**注**：
- SQL对大小写不敏感：SELECT与select是相同的
+ 要求在每条SQL语句的末端使用分号

---

## <span id="数据库引擎">九、数据库引擎</span>

> **数据库引擎**：数据库引擎是用于**存储、处理和保护数据的核心服务**。利用数据库引擎可控制访问权限并快速处理事务，从而满足企业内大多数处理大量数据的应用程序的要求。使用数据库引擎创建用于联机事务处理或联机分析处理的关系数据库，这包括创建用于存储数据的表和用于查看、管理和保护数据安全的数据库对象（如索引、视图和存储过程）。

### 数据库引擎任务
在数据库引擎文档中，各主题的顺序遵循用于实现数据库引擎进行存储的系统的任务的主要顺序。

- 设计并创建数据库以保存系统所需的关系或XML文档
+ 实现系统以访问和更改数据库中存储的数据。包括实现网站或使用数据的应用程序，还包括生成使用SQL Server工具和实用工具以使用数据的过程。
- 为单位或客户部署实现的系统
+ 提供日常管理支持以优化数据库的性能

### MySQL数据库引擎类别
能用的数据库引擎取决于mysql在安装时是如何被编译的。要添加一个新的引擎，就必须重新编译MySQL。在缺省情况下，MySQL支持三个引擎：ISAM、MYISAM和HEAP。另外两种类型为INNODB和BERKLEY（BDB），也常常可以使用。

1. ISAM

ISAM是一个定义明确且历经时间考验的数据表格管理方法，它在设计之时就考虑到数据库被查询的次数要远大于更新的次数。因此，ISAM执行读取操作的速度很快，而且不占用大量的内存和存储资源。ISAM的两个主要不足之处在于，它不支持事务处理，也不能容错：如果硬盘崩溃，数据文件无法恢复；如果正在把ISAM用在关键任务应用程序里，就必须经常备份所有的实时数据，通过其复制特性，MySQL能够支持这样的备份应用程序。

优点：执行读取操作的速度快；不占用大量的内存和存储资源。

缺点：不支持事务处理；不能容错。

2. MYISAM

MYISAM是MySQL的ISAM扩展格式和缺省的数据库引擎。除了提供ISAM里所没有的索引和字段管理的功能，MYISAM还使用一种表格锁定的机制，来优化多个并发的读写操作。其代价是需要经常运行OPTIMIZE TABLE命令，来回复被更新机制所浪费的空间。MYISAM还有一些有用的扩展，例如用来修复数据库文件的MYISAMCHK工具和用来恢复浪费空间的MYISAMPACK工具。

MYISAM强调了快速读取操作，这就是为什么MySQL受到了WEB开发如此青睐的主要原因：在WEB开发中所进行的大量数据操作都是读取操作。所以，大多数虚拟主机提供商和INTERNET平台提供商只允许使用MYISAM格式。MYISAM格式的一个重要缺陷就是不能在表损坏后恢复数据。

3. HEAP

HEAP允许只驻留在内存里的临时表格。驻留在内存里让HEAP要比ISAM和MYISAM都快，但是它所管理的数据是不稳定的，而且如果在关机之前没有进行保存，那么所有的数据都会丢失。在数据行被删除的时候，HEAP也不会浪费大量的空间。HEAP表格在需要使用SELECT表达式来选择和操控数据的时候非常有用，但是在用完表格后要记得删除表格。

4. INNODB和BERKLEYDB

INNODB和BERKLEYDB（BDB）数据库引擎都是造就MySQL灵活性的技术的直接产品，这项技术就是MySQL+API。在使用MySQL时，面对的每一个挑战几乎都源于ISAM和MYISAM数据库引擎不支持事务处理（transaction process），也不支持外来键。尽管要比ISAM和MYISAM引擎慢很多，但是INNODB和BDB包括了对事务处理和外来键的支持。

5. MEMORY

MEMORY是MySQL中一类特殊的存储引擎。它使用存储在内存中的内容来创建表，而且数据全部放在内存中。这些特性和前面的引擎很不同。每个基于MEMORY存储引擎的表实际对应一个磁盘文件。该文件的文件名与表名相同，类型为frm类型。该文件只存储表的结构，而其数据文件都是存储在内存中，这样有利于数据的快速处理，提高整个表的效率。值得注意的是，服务器需要有足够的内存来维持MEMORY存储引擎的表的使用。如果不需要了，可以释放内存，甚至删除不需要的表。MEMORY默认使用哈希索引，速度比使用B型树索引快。若想使用B型树索引，可以在创建索引时指定。

注：MEMORY用到的很少，因为它是把数据存到内存中，如果内存出现异常就会影响数据。如果重启或关机，所有数据都会丢失。因此，基于MEMORY的表的生命周期很短，一般是一次性的。

### 数据库引擎比较
在实际工作中，选择一个合适的存储引擎是一个比较复杂的问题。每种存储引擎都有自己的优缺点，不能笼统地说谁比谁好。

1. InnoDB：支持事务处理，支持外键，支持崩溃修复能力和并发控制。如果需要对事务的完整性要求比较高（比如银行），要求实现并发控制（比如售票），那选择InnoDB有很大的优势。如果需要频繁的更新、删除操作的数据库，也可以选择InnoDB，因为支持事务的提交（commit）和回滚（rollback）。 

2. MyISAM：为MySQL默认的插件式存储引擎，插入数据快，空间和内存使用比较低。如果表主要是用于插入新记录和读出记录，那么选择MyISAM能实现处理高效率。如果应用的完整性、并发性要求比较低，也可以使用。

3. MEMORY：所有的数据都在内存中，数据的处理速度快，但是安全性不高。如果需要很快的读写速度，对数据的安全性要求较低，可以选择MEMOEY。它对表的大小有要求，不能建立太大的表。所以，这类数据库只使用在相对较小的数据库表。

注意，同一个数据库也可以使用多种存储引擎的表。如果一个表要求比较高的事务处理，可以选择InnoDB。这个数据库中可以将查询要求比较高的表选择MyISAM存储。如果该数据库需要一个用于查询的临时表，可以选择MEMORY存储引擎。

---

## <span id="SQL注入">十、MySQL及SQL注入</span>

如果通过网页获取用户输入的数据并将其插入一个MySQL数据库，就有可能发生SQL注入安全的问题。

> **SQL注入**：所谓SQL注入，就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终欺骗服务器执行恶意的SQL命令。

永远不要信任用户的输入，必须认定用户输入的数据都是不安全的，需要对用户输入的数据进行过滤处理。

**实例**：

1. 以下代码中，输入的用户名必须为字母、数字及下划线的组合，且用户名长度为8到20个字符之间：

```php
if (preg_match("/^\w{8,20}$/", $_GET['username'], $matches))
{
   $result = mysqli_query($conn, "SELECT * FROM users 
                          WHERE username=$matches[0]");
}
 else 
{
   echo "username 输入异常";
}
```

2. 没有过滤特殊字符时：

```php
// 设定$name 中插入了我们不需要的SQL语句
$name = "Qadir'; DELETE FROM users;";
 mysqli_query($conn, "SELECT * FROM users WHERE name='{$name}'");
```

以上的注入语句中，没有对`$name`变量进行过滤，`$name`中插入了不需要的SQL语句，将删除users表中的所有数据。

在PHP中的`mysqli_query()`是不允许执行多个SQL语句的，但是在SQLite和PostageSQL中是可以同时执行多条SQL语句的，所以要对这些用户的数据进行严格的验证。

防止SQL注入，需要注意的要点：

- 永远不要信任用户的输入。对用户的输入进行校验，可以通过正则表达式，或限制长度；对单引号和“--”进行转换等
+ 永远不要使用动态瓶装sql，可以使用参数化的sql或者直接使用存储过程进行数据查询存取
- 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限、有限的数据库连接
+ 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息
- 应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装
  sql注入的检测方法一般采取辅助软件或网站平台来检测，软件一般采用sql注入检测工具jsky，网站平台有亿思网站安全平台检测工具、MDCSOFT SCAN等。采用MDCSOFT-IPS可以有效的防御SQL注入、XSS攻击。
  
 ### 防止SQL注入
 
 在脚本语言，如Perl和PHP中，可以对用户输入的数据进行转义来防止SQL注入。
 
 PHP的MySQL扩展提供了`mysqli_real_escape_string()`函数来转义特殊的输入字符。
 
 ```php
 if (get_magic_quotes_gpc()) 
{
  $name = stripslashes($name);
}
$name = mysqli_real_escape_string($conn, $name);
 mysqli_query($conn, "SELECT * FROM users WHERE name='{$name}'");
 ```
 
 ### Like语句中的注入
 
 LIKE查询时，如果用户输入的值有“_”和“%”，会出现这种情况：用户本来只是想查询“abcd_”，查询结果中却有“abcd_”、“abcde”、“abcdf”等待；用户要查询“30%”（百分之三十）时也会出现问题。
 
 在PHP脚本中可以使用`addcslashes()`函数来处理以上情况：
 
 ```php
 $sub = addcslashes(mysqli_real_escape_string($conn, "%something_"), "%_");
// $sub == \%something\_
 mysqli_query($conn, "SELECT * FROM messages WHERE subject LIKE '{$sub}%'");
 ```
 
 > addcslashes()函数：在指定的字符前添加反斜杠
 
 语法格式：`addcslashes(string,characters)`
 
 参数 | 描述
 ----|----
 string | 必需。规定要检查的字符串
 characters | 可选。规定受其影响的字符或字符范围
