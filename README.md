# MySQL
MySQL重要概念与常用语句

## 安装
### Ubuntu
>Ubuntu（友帮拓、优般图、乌班图）：是一个以桌面应用为主的开源GNU/Linux操作系统。Ubuntu 是基于Debian GNU/Linux，支持x86、amd64（即x64）和ppc架构，由全球化的专业开发团队（Canonical Ltd）打造的。

**在Ubuntu下安装MySQL**

首先执行下面三条命令：
```
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
执行完成后可以通过下面的命令测试是否安装成功：

`sudo netstat -tap | grep mysql`

通过如下命令进入MySQL服务：

`mysql -uroot -p密码`


### CentOS

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
