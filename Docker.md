# Docker

## 启动容器

```sh
docker run -tid -p 8080:80 -p 3309:3306 -v /Users/kings/www:/var/www/html --name web ubuntu /bin/bash
-v 磁盘的映射
--name 容器名称

docker run -tid -p 8080:80 -p 3309:3306 -v /Users/kings/www:/var/www/html --name web cqzhongxc/web:v2 /bin/bash boot.sh
-v 磁盘的映射
--name 容器名称
boot.sh 容器启动脚本，用于启动nginx mysql php-fpm
```

## 登陆容器

```
docker exec -ti web /bin/bash
```

## 更新镜像源和软件

```
apt update > 更新镜像源
apt upgrade > 更新软件
```

## 安装PHP NGINX MYSQL VIM

```
apt install -y nginx php-fpm mysql-client mysql-server vim
```

## 通过容器生成DCOKER镜像

```
docker commit -m="first commit" -a='kings' web kings/web:v1
```

## 设置镜像标签

```
docker tag 2a816c3a0da4 cqzhongxc/lampserver:v2
```

## 发布镜像到Docker仓库

```
docker push cqzhongxc/lampserver:v4
```

## Docker从入门到实践

```
https://yeasy.gitbooks.io/docker_practice/install/mirror.html#macos
```

## Mac 正在运行的container添加端口映射

```
https://www.dazhuanlan.com/2019/12/10/5dee8c6cb0067/
```

## MYSQL安全配置与远程访问权限设置

**root账号远程访问**

```shell
1. service mysql start

2. 通过安全初始设置 mysql_secure_installation 命令完成设置

3. 打开 vim /etc/mysql/mysql.conf.d/mysqld.cnf 配置文件
    注释掉 # bind-address = 127.0.0.1 允许外部IP访问

4. 使用mysql命令完成
set global validate_password_policy=LOW;
grant all privileges on *.* to 'root'@'%' identified by 'admin888';
flush privileges;

usermod -d /var/lib/mysql/ mysql
chown -R msyql:mysql /var/lib/msyql

CREATE USER 'ssh'@'localhost'  IDENTIFIED BY 'ssh'; #创建用户，本地登录
CREATE USER 'ssh'@'%'  IDENTIFIED BY 'ssh'; #创建用户，远程登录

```

## Docker php扩展安装

### php 通过 pecl 安装 swoole 扩展

#### 简介

> Pecl 全称 The PHP Extension Community Library，php 社区扩展库，由社区编写，维护。使用 pecl 方便之处在于我们不用到处找源码包下载编译，配置，不用手动 phpize,configure,make,make install, 自动识别模块安装路径，我们只需要编辑 php.ini 配置文件开启扩展，当然我们也需要自己配置一些参数的时候可以先下载源码再构建

#### 安装 pecl

**Ubuntu/Debian/Deepin**

```
apt install php-dev php-pear autoconf automake libtool -y
```

**Centos**

```
yum install php-dev php-pear autoconf automake libtool -y
```

#### pecl 常用命令

**build**

> 从 C 的源码中构建扩展
> **install**
> 安装一个包，步骤包含 (configure,make,make install)
> **download**
> 下载源码包
> **list-all**
> 列出全部包
> **run-tests**
> 运行测试 (make test)

#### 安装 Swoole 扩展

```
pecl install swoole
```

#### 配置 PHP.ini

```
php -i |grep php.ini（查看php.ini位置）
extension=swoole.so（写入php.ini文件中）
```

#### 确认是否安装成功

```
php -m | grep swoole
```

#### 查看php扩展模块extension_dir的目录位置

```
php -i | grep -i extension_dir
```

### Nginx PHP配置

```shell
// 第一步
vim /etc/php/7.2/fpm/pool.d/www.conf  >  listen = /run/php/php7.2-fpm.sock
// 第二步
vim /etc/nginx/sites-enabled/default
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
#
    fastcgi_pass unix:/run/php/php7.2-fpm.sock;
# # With php-cgi (or other tcp sockets):
#   fastcgi_pass 127.0.0.1:9000;
}

ls /etc/init.d/
service php7.2-fpm start
service nginx start
```



## PHP+Nginx

### 安装Nginx

```
docker pull nginx
```

### 安装PHP

```
docker pull php:7.3.5-fpm
```

### 启动PHP-FPM

```shell
docker run -tid \
-v /Users/kings/www/html:/var/www/html \
--name myphpfpm \
php:7.3.5-fpm
```

 **/data/ftp** 是外部路径 **www** 是docker里的映射路径

### 启动Nginx

```dockerfile
docker run -tid --name php_nginx \
-p 8089:80 \
-v /Users/kings/www/html:/user/share/nginx/html:ro \
-v /data/nginx/conf/conf.d:/etc/nginx/conf.d:ro \
--link myphpfpm:php \
nginx
```

- **-p 8089:80**: 端口映射，把 **nginx** 中的 80 映射到本地的 8089 端口。
- **/data/ftp**: 是本地 html 文件的存储目录，/usr/share/nginx/html 是容器内 html 文件的存储目录。
- **/data/conf/conf.d**: 是本地 nginx 配置文件的存储目录，/etc/nginx/conf.d 是容器内nginx 配置文件的存储目录。
- **--link myphpfpm:php**: 把 myphpfpm 的网络并入 **nginx**，并通过修改 **nginx** 的 /etc/hosts，把域名 **php** 映射成 127.0.0.1，让 nginx 通过 php:9000 访问 php-fpm。
- **ro** 表示只读

### Nginx配置文件

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm index.php;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /www/$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

配置文件说明：

- **php:9000**: 表示 php-fpm 服务的 URL，下面我们会具体说明。
- **/www/**: 是 myphpfpm 中 php 文件的存储路径，映射到本地的 ~/nginx/www 目录。



## MySQL

使用Docker搭建MySQL服务

### 拉取镜像

```dockerfile
docker pull mysql:5.7.29 # 指定版本
docker pull mysql	# 拉取最新版本
```

### 查看镜像是否拉取成功

```dockerfile
docker images
```

### 运行容器（未建立目录映射）

```dockerfile
docker run -tid -p 3306:3306 \
--name mysql \
mysql:5.7.29 \
-e MYSQL_ROOT_PASSWORD=123456
```

### 运行容器（建立目录映射）

```dockerfile
docker run -tid -p 3306:3306 \
-v /Users/kings/data/mysql/conf:/etc/mysql \
-v /Users/kings/data/mysql/logs:/var/log/mysql \
-v /Users/kings/data/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql5.7 \
mysql:5.7.29
```

### 检查容器是否正确运行

```dockerfile
docker container ls
```

- 可以看到容器ID，容器的源镜像，启动命令，创建时间，状态，端口映射信息，容器名字

### 连接MySQL

#### 进入dock er连接mysql

```cmake
docker exec -tid mysql /bin/bash
mysql -uroot -p123456
```

#### 使用 Navicat 远程连接mysql

**使用远程连接软件时要注意一个问题**

**在创建容器的时候已经将容器的3306端口和主机的3306端口映射到一起，所以应该访问：**

```mysql
host: 127.0.0.1
port: 3306
user: root
password: 123456
```

**如果你的容器运行正常，但是无法访问到MySQL，一般有以下几个可能的原因：**

- 防火墙阻拦

  ```cmake
  # 开放端口：
  $ systemctl status firewalld
  $ firewall-cmd  --zone=public --add-port=3306/tcp -permanent
  $ firewall-cmd  --reload
  # 关闭防火墙：
  $ sudo systemctl stop firewalld
  ```

  

- 需要进入docker本地客户端设置远程访问账号

  ```mysql
  $ docker exec -it mysql bash
  $ mysql -uroot -p123456
  mysql> grant all privileges on *.* to root@'%' identified by "password";
  ```

  原理：

  ```mysql
  # mysql使用mysql数据库中的user表来管理权限，修改user表就可以修改权限（只有root账号可以修改）
  
  mysql> use mysql;
  Database changed
  
  mysql> select host,user,password from user;
  +--------------+------+-------------------------------------------+
  | host                    | user      | password                                                                 |
  +--------------+------+-------------------------------------------+
  | localhost              | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
  | 192.168.1.1            | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
  +--------------+------+-------------------------------------------+
  2 rows in set (0.00 sec)
  
  mysql> grant all privileges  on *.* to root@'%' identified by "password";
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> flush privileges;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> select host,user,password from user;
  +--------------+------+-------------------------------------------+
  | host                    | user      | password                                                                 |
  +--------------+------+-------------------------------------------+
  | localhost              | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
  | 192.168.1.1            | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
  | %                       | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
  +--------------+------+-------------------------------------------+
  3 rows in set (0.00 sec)
  ```

  

## Docker PHP扩展配置

https://www.jianshu.com/p/20fcca06e27e

```dockerfile
# PHP 容器配置

# 从官方基础版本构建
FROM php:7.2-fpm
# 官方版本默认安装扩展: 
# Core, ctype, curl
# date, dom
# fileinfo, filter, ftp
# hash
# iconv
# json
# libxml
# mbstring, mysqlnd
# openssl
# pcre, PDO, pdo_sqlite, Phar, posix
# readline, Reflection, session, SimpleXML, sodium, SPL, sqlite3, standard
# tokenizer
# xml, xmlreader, xmlwriter
# zlib

# 1.0.2 增加 bcmath, calendar, exif, gettext, sockets, dba, 
# mysqli, pcntl, pdo_mysql, shmop, sysvmsg, sysvsem, sysvshm 扩展
RUN docker-php-ext-install -j$(nproc) bcmath calendar exif gettext \
sockets dba mysqli pcntl pdo_mysql shmop sysvmsg sysvsem sysvshm

# 1.0.3 增加 bz2 扩展, 读写 bzip2（.bz2）压缩文件
RUN apt-get update && \
apt-get install -y --no-install-recommends libbz2-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) bz2

# 1.0.4 增加 enchant 扩展, 拼写检查库
RUN apt-get update && \
apt-get install -y --no-install-recommends libenchant-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) enchant

# 1.0.5 增加 GD 扩展. 图像处理
RUN apt-get update && \
apt-get install -y --no-install-recommends libfreetype6-dev libjpeg62-turbo-dev libpng-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
docker-php-ext-install -j$(nproc) gd

# 1.0.6 增加 gmp 扩展, GMP
RUN apt-get update && \
apt-get install -y --no-install-recommends libgmp-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) gmp

# 1.0.7 增加 soap wddx xmlrpc tidy xsl 扩展
RUN apt-get update && \
apt-get install -y --no-install-recommends libxml2-dev libtidy-dev libxslt1-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) soap wddx xmlrpc tidy xsl

# 1.0.8 增加 zip 扩展
RUN apt-get update && \
apt-get install -y --no-install-recommends libzip-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) zip

# 1.0.9 增加 snmp 扩展
RUN apt-get update && \
apt-get install -y --no-install-recommends libsnmp-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) snmp

# 1.0.10 增加 pgsql, pdo_pgsql 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends libpq-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) pgsql pdo_pgsql

# 1.0.11 增加 pspell 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends libpspell-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) pspell

# 1.0.12 增加 recode 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends librecode-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) recode

# 1.0.13 增加 PDO_Firebird 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends firebird-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) pdo_firebird

# 1.0.14 增加 pdo_dblib 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends freetds-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure pdo_dblib --with-libdir=lib/x86_64-linux-gnu && \
docker-php-ext-install -j$(nproc) pdo_dblib

# 1.0.15 增加 ldap 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends libldap2-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu && \
docker-php-ext-install -j$(nproc) ldap

# 1.0.16 增加 imap 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends libc-client-dev libkrb5-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
docker-php-ext-install -j$(nproc) imap

# 1.0.17 增加 interbase 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends firebird-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) interbase

# 1.0.18 增加 intl 扩展 
RUN apt-get update && \
apt-get install -y --no-install-recommends libicu-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) intl

# 1.0.19 增加 mcrypt 扩展 
RUN apt-get update && \ 
apt-get install -y --no-install-recommends libmcrypt-dev && \
rm -r /var/lib/apt/lists/* && \
pecl install mcrypt-1.0.1 && \
docker-php-ext-enable mcrypt

# 1.0.20 imagick 扩展
RUN export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" && \
apt-get update && \
apt-get install -y --no-install-recommends libmagickwand-dev && \
rm -rf /var/lib/apt/lists/* && \
pecl install imagick-3.4.3 && \
docker-php-ext-enable imagick

# 1.0.21 增加 Memcached 扩展 
RUN apt-get update && \ 
apt-get install -y --no-install-recommends zlib1g-dev libmemcached-dev && \
rm -r /var/lib/apt/lists/* && \
pecl install memcached && \
docker-php-ext-enable memcached

# 1.0.22 redis 扩展
RUN pecl install redis-4.0.1 && docker-php-ext-enable redis

# 1.0.23 增加 opcache 扩展 
RUN docker-php-ext-configure opcache --enable-opcache && docker-php-ext-install opcache

# 1.0.24 增加 odbc, pdo_odbc 扩展 
RUN set -ex; \
docker-php-source extract; \
{ \
     echo '# https://github.com/docker-library/php/issues/103#issuecomment-271413933'; \
     echo 'AC_DEFUN([PHP_ALWAYS_SHARED],[])dnl'; \
     echo; \
     cat /usr/src/php/ext/odbc/config.m4; \
} > temp.m4; \
mv temp.m4 /usr/src/php/ext/odbc/config.m4; \
apt-get update; \
apt-get install -y --no-install-recommends unixodbc-dev; \
rm -rf /var/lib/apt/lists/*; \
docker-php-ext-configure odbc --with-unixODBC=shared,/usr; \
docker-php-ext-configure pdo_odbc --with-pdo-odbc=unixODBC,/usr; \
docker-php-ext-install odbc pdo_odbc; \
docker-php-source delete

# 镜像信息
LABEL Author="Leo"
LABEL Version="1.0.25-fpm"
LABEL Description="PHP FPM 7.2 镜像. All extensions."
```

