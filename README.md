# MySQL
MySQL重要概念与常用语句

## 安装
### Ubuntu
>Ubuntu（友帮拓、优般图、乌班图）：是一个以桌面应用为主的开源GNU/Linux操作系统。Ubuntu 是基于Debian GNU/Linux，支持x86、amd64（即x64）和ppc架构，由全球化的专业开发团队（Canonical Ltd）打造的。

**在Ubuntu下安装mysql**

1. 首先执行下面三条命令：
```
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
2. 执行完成后可以通过下面的命令测试是否安装成功：

`sudo netstat -tap | grep mysql`

3. 通过如下命令进入mysql服务：

`mysql -uroot -p密码`


### CentOS
>CentOS（Community Enterprise Operating System，社区企业操作系统）：是Linux发行版之一，它是来自于Red Hat Enterprise Linux依照开放源代码规定释出的源代码所编译而成。由于出自同样的源代码，因此有些要求高度稳定性的服务器以CentOS替代商业版的Red Hat Enterprise Linux使用。两者的不同，在于CentOS完全开源。

**在CentOS下安装mysql**
1. 首先检查系统是否装有mysql:

`rpm -qa | grep mysql`

若返回空值，说明没有安装；若已经装有mysql，通过以下命令删除可用：

`yum remove mysql`


2.
## 启动服务与连接

## 数据库管理
### 权限设置
### 用户设置
### 日志
### 备份

## 库操作
### 查看、选择
### 新建数据库
### 删除数据库

## 表操作
### 查看表结构
### 数据结构
### 新建表
### 删除表
### 修改表结构

## 数据操作
### 插入数据
### 修改数据
### 删除数据

## 视图

## SQL语法
