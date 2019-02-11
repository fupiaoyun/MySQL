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

`sudo netstat -tap | grep mysql`

3.通过如下命令进入mysql服务：

`mysql -uroot -p密码`

详见[Ubuntu安装mysql教程](https://blog.csdn.net/xiangwanpeng/article/details/54562362
)


### CentOS
>CentOS（Community Enterprise Operating System，社区企业操作系统）：是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS完全开源。

**在CentOS下安装mysql**
1.首先检查系统是否装有mysql:

`rpm -qa | grep mysql`

若返回空值，说明没有安装；若已经装有mysql，通过以下命令删除可用：

`yum remove mysql`

注：这里执行安装命令是无效的，因为centos默认是Mariadb，执行以下命令只是更新Mariadb数据库

`yum install mysql`
2.下载mysql的repo源:

`# wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm`

安装`mysql-community-release-el7-5.noarch.rpm包`：

`# sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm`

安装这个包后，会获得两个mysql的`yum repo源`：/etc/yum.repos.d/mysql-community.repo,/etc/yum.repos.d/mysql-community-source.repo。

3.安装mysql

`# sudo yum install mysql-server`

详见[CentOS安装mysql教程](https://blog.csdn.net/a774630093/article/details/79270080
)

## 二、启动服务与连接
1.启动mysql服务器：

`sudo service mysql start`

2.连接服务器：

`mysql -u root 密码`


## 三、数据库管理
### 用户管理
1.选择数据库

`mysql>use mysql;`

2.查看该数据库中所有用户

`mysql>select host,user,password from user;`

3.创建新用户

`mysql>create user test IDENTIFIED by 'xxxxx';//identified by会将纯文本密码加密作为散列值存储`

4.修改用户名

`mysql>rename user feng to newuser;//mysql5之后可以使用，之前需要使用update更新user表`

5.删除用户

`mysql>drop user newuser;//mysql5之前删除用户时必须先使用revoke删除用户权限，然后删除用户`

### 权限设置
1.查看用户权限

`mysql>show grants for test;`

2.赋予用户新权限

`mysql>grant select on dmc_db.* to test;`

3.回收权限

`mysql>revoke select on dmc_db.* from test;//如果权限不存在会报错`

**注：上面的命令也可以使用多个权限同时赋予和回收，权限之间使用逗号分隔**

`mysql>grant select,update,delete,insert on dmc_db.* to test;`

4.刷新权限

`mysql>flush privileges;`

### 日志
1.日志分类

在MariaDB/MySQL中，主要有5种日志文件：
- 错误日志（error log）：记录mysql服务的启停时正确和错误的信息，还记录启动、停止、运行过程中的错误信息。
+ 查询日志（general log）：记录建立的客户端连接和执行的语句。
- 二进制日志（bin log）：记录所有更改数据的语句，可用于数据复制。
+ 慢查询日志（slow log）：记录所有执行时间超过long_query_time的所有查询或不使用索引的查询。
- 中继日志（relay log）：主从复制时使用的日志。
除了这5种日志，在需要时还会创建DDL日志。

2.日志刷新操作
以下操作会刷新日志文件，刷新日志文件时会关闭旧的日志文件并重新打开日志文件。对于有些日志类型，如二进制日志，刷新日志会滚动日志文件，而不仅仅是关闭并重新打开。
```
mysql>FLUSH LOGS;
shell>mysqladmin flush-logs
shell>mysqladmin refresh
```

3.错误日志
错误日志是最重要的日志之一，它记录了MariaDB/MySQL服务启动和停止正确和错误的信息，还记录了mysqld实例运行过程中发生的错误事件信息。

可以使用`--log-error=[file_name]`来指定mysqld记录的错误日志文件，若没有指定file_name，则默认的错误日志文件在datadir目录下的'hostname'.err，hostname表示当前的主机名。

也可以在MariaDB/MySQL配置文件中的mysqld配置部分，使用log-error指定错误日志的路径。

若不知道错误日志的位置，可以使用变量`log_error`来查看:

`mysql>show variables like 'log_error';`

在MySQL5.5.7之前，刷新日志操作（如flush logs）会被备份旧的错误日志（以_old结尾），并创建一个新的错误日志并打开。在MySQL5.5.7之后，执行刷新日志的操作时，错误日志会关闭并重新打开。如果错误日志不存在，则会先创建。

4.一般查询日志
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

`mysql>set @@global.general_log=1;`

一般查询日志记录的不止是SELECT语句，几乎所有的语句都会记录。

5.慢查询日志
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

6.二进制日志
（1）二进制日志文件
二进制日志包含了引起或可能引起数据库改变（如delete语句但没有匹配行）的事件信息，但绝不会包括select和show这样的查询语句。语句以“事件”的形式保存，所以包含了时间、事件开始和结束的位置等信息。

二进制日志是**以事件形式记录的**，不是事务日志（但可能是基于事务来记录二进制日志），不代表它只记录innodb日志，myisam表也一样有二进制日志。

二进制日志**只在事务提交的时候一次性写入**。

MariaDB/MySQL默认不启动二进制日志，要启动二进制日志使用`--log-bin=[on|off|file_name]`选项指定，若没有给定file_name,则默认为

### 备份

## 四、库操作
### 查看、选择
### 新建数据库
### 删除数据库

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
