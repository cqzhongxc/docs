# MySQL

## 新建用户

**创建ssh用户，密码是ssh。localhost 就是本地连接，即127.0.0.1**
**%用于远程连接，即任意ip都可以链接**

```mysql
mysql -u root -p
CREATE USER 'ssh'@'localhost' IDENTIFIED BY 'ssh'; #本地登录
CREATE USER 'ssh'@'%' IDENTIFIED BY 'ssh'; #远程登录
quit   先退出，在测试
mysql -ussh -p #测试是否创建成功
```

## 为用户授权

**a. 授权格式：grant 权限 on 数据库.\* to 用户名@登录主机 identified by '密码'**
**b. 登录MYSQL，这里以ROOT身份登录：**

```
mysql -u root -p
```

**c. 为用户创建一个数据库(test )**

```
create database test DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
show databases;    # 查看数据库
```

**d. 授权ssh用户拥有test数据库的所有权限：**

```
grant all privileges on `test`.* to 'ssh'@'localhost' identified by 'ssh';
grant all privileges on `test`.* to 'ssh'@'%' identified by 'ssh';
flush privileges; #刷新系统权限表
```

::: alert-info
注意：MySQL中字符要求很严格，必须用英文小写字符！
:::

## 删除用户

```
mysql -u root -p 
Delete FROM mysql.user Where User=”test” and Host=”localhost”; 
flush privileges; 
drop database testDB;
```

## 删除账户及权限

```
drop user 用户名@’%’;
drop user 用户名@ localhost;
```

*附：有可能出现的问题：*
**使用以下命令行删除账户：**

```
delete from user where user='账户名';
```

**出现：**

```
ERROR 1046 (3D000): No database selected 错误：没有选中数据库。
```

**因为是直接使用 SQL 语句的方式来删除账户，所以必须先选择 mysql 自身的数据库：**

```
use mysql;
```

**查看数据库所用端口**

```
show variables  like 'port';
```

