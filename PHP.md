# PHP

[TOC]

## CentOS编译安装

### 相关依赖包安装

#### 扩展支持(mcrypt、mhash扩展和libevent)

如果想让编译的php支持mcrypt、mhash扩展和libevent，需要安装以下包

```php
libmcrypt libmcrypt-devel mhash mhash-devel
```

说明：

Centos源不能安装libmcrypt-devel，由于版权的原因没有自带mcrypt的包
可以使用第三方源，这样还可以使用yum来安装
安装第三方yum源

```cmake
wget http://www.atomicorp.com/installers/atomic
sh ./atomic
```

使用yum命令安装

```cmake
yum install php-mcrypt libmcrypt libmcrypt-devel mhash mhash-devel
```

#### libevent相关包

可以根据需要安装libevent，系统一般会自带libevent，但版本有些低。因此可以升级安装如下两个rpm包。

```
yum install libevent libevent-devel 
```

说明：
**libevent**是一个异步事件通知库文件，其API提供了在某文件描述上发生某事件时或其超时时执行回调函数的机制
它主要用来替换事件驱动的网络服务器上的event loop机制。
目前来说， libevent支持/dev/poll、kqueue、select、poll、epoll及Solaris的event ports。

#### 支持xml的相关包

支持xml的rpm包
bzip2 是一个基于Burrows-Wheeler 变换的无损压缩软件能够高效的完成文件数据的压缩
libcurl主要功能就是用不同的协议连接和沟通不同的服务器，也就是相当封装了的sockPHP 
libcurl允许你用不同的协议连接和沟通不同的服务器

```
yum install libxml2 libxml2-devel bzip2-devel libcurl-devel
```

#### 图形相关的rpm包

通常对应的错误提示：JIS-mapped Japanese font support in GD

```
yum install libjpeg-devel libpng-devel freetype-devel
```

```
yum install -y gcc gcc-c++  make zlib zlib-devel pcre pcre-devel  libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers libxslt libxslt-devel
```

### 编译安装PHP

首先下载源码包至本地目录

```
wget https://www.php.net/distributions/php-7.4.3.tar.gz
```

编译配置项

```cmake
./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-opcache \
--enable-fpm \
--with-mysql=/usr/local/mysql \
--with-mysqli=/usr/local/mysql/bin/mysql_config \
--with-pdo-mysql=/usr/local/mysql \
--with-gettext \
--enable-mbstring \
--with-iconv \
--with-mcrypt \
--with-mhash \
--with-openssl \
--enable-bcmath \
--enable-soap \
--with-libxml-dir \
--enable-pcntl \
--enable-shmop \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-sockets \
--with-curl \
--with-zlib \
--enable-zip \
--with-bz2 \
--with-gd \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir
```



有关编译配置项的详细描述：https://segmentfault.com/a/1190000002717262

### 编译并安装

```
make && make install
```

### PHP配置

**php.ini ** 是php运行核心配置文件
**php-fpm.conf**  是php-fpm进程服务的配置文件

```
cp php.ini-production /usr/local/php/etc/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
```

### FPM测试PHP配置

```cmake
/usr/local/php/sbin/php-fpm -t
[23-May-2016 20:03:52] NOTICE:
configuration file /usr/local/php/etc/php-fpm.conf test is successful

chkconfig --add php-fpm # 设置自启动
chkconfig php-fpm on 	# 
service php-fpm start # 启动PHP
```

### PHP进程查看

```cmake
ps -ef|grep php
```

### PHP端口查看

```
netstat -nltp|grep 9000
```



## 安装扩展

### 通过pecl方式安装扩展

#### 安装pecl

```linux
cd /usr/local/php/bin/
wget http://pear.php.net/go-pear.phar -O go-pear.php
php go-pear.php
##回车默认安装
```

#### 安装php扩展

```cmake
pecl search key-word    # 用于查找扩展
pecl install key-word   # 用于安装扩展
```

#### 查询相关扩展

```cmake
[root@localhost src]# pecl search swoole
Retrieving data...0%
Matched packages, channel pecl.php.net:
=======================================
Package Stable/(Latest) Local
swoole  1.8.12 (stable) 1.8.12 Event-driven asynchronous and concurrent networking engine with high performance for PHP.
[root@localhost src]# pecl search xdebug
Retrieving data...0%
Matched packages, channel pecl.php.net:
=======================================
Package Stable/(Latest) Local
xdebug  2.4.1 (stable)        Provides functions for function traces and profiling
```

#### 安装相关扩展

```
pecl install xdebug
## 安装完成之后，输出
Build process completed successfully
Installing '/usr/lib64/php/modules/xdebug.so'
install ok: channel://pecl.php.net/xdebug-2.4.1
configuration option "php_ini" is not set to php.ini location
You should add "zend_extension=/usr/lib64/php/modules/xdebug.so" to php.ini
## 根据提示，我们在php.ini的最后添加
zend_extension=/usr/lib64/php/modules/xdebug.so
pecl install swoole
```

### CentOS yum方式安装扩展

#### 扩展包更新

```
yum  install epel-release
```

#### 安装扩展

```
yum install libmcrypt libmcrypt-devel mcrypt mhash
```

**"configure: error: Cannot find OpenSSL's <evp.h>"**

```
sudo apt-get install libssl-dev
```



## 面试

## PHP部分

### 掌握PHP的哪些框架、模板引擎

框架：框架有很多，例如CI、Yii、Laravel等等，咱们学过的是thinkphp
模板引擎：也有很多，在课本中有，咱们学过的是smarty

### 请问在开发中应该注意哪些安全机制？

（1）使用验证码防止注册机灌水。
（2）使用预处理，绑定参数，参数过滤转义 防止sql注入
（3）使用token防止远程提交，使用token验证登录状态。

### 如何提高程序的运行效率？

（1）优化SQL语句，查询语句中尽量不使用select *，用哪个字段查哪个字段；少用子查询可用表连接代替；少用模糊查询。
（2）数据表中创建索引。
（3）对程序中经常用到的数据生成缓存（比如使用redis缓存数据，比如使用ob进行动态页面静态化等等）。
（4）对mysql做主从复制，读写分离。（提高mysq执行效率和查询速度）
（5）使用nginx做负载均衡。（将访问压力平均分配到多态服务器）

### 请问MVC分别指哪三层，有什么优点？

MVC三层分别指：业务模型、视图、控制器，由控制器层调用模型处理数据，然后将数据映射到视图层进行显示。
优点是：①可以实现代码的重用性，避免产生代码冗余；②M和V的实现代码分离，从而使同一个程序可以使用不同的表现形式

### 如何实现单点登录

利用 jwt 实现 session 共享，具体使用 jwt 参考 http://blog.leapoahead.com/2015/09/07/user-authentication-with-jwt/

## THINKPHP部分

### TP中的URL模式有哪几种？默认是哪种

ThinkPHP支持四种URL模式，可以通过设置URL_MODEL参数来定义，包括普通模式、PATHINFO、REWRITE和兼容模式。
默认模式为：PATHINFO模式，设置URL_MODEL 为1



## Laravel部分

### 服务提供者是什么？

服务提供者是所有 Laravel 应用程序引导启动的中心, Laravel 的核心服务器、注册服务容器绑定、事件监听、中间件、路由注册以及我们的应用程序都是由服务提供者引导启动的。

### IoC 容器是什么？

IoC（Inversion of Control）译为 「控制反转」，也被叫做「依赖注入」(DI)。什么是「控制反转」？对象 A 功能依赖于对象 B，但是控制权由对象 A 来控制，控制权被颠倒，所以叫做「控制反转」，而「依赖注入」是实现 IoC 的方法，就是由 IoC 容器在运行期间，动态地将某种依赖关系注入到对象之中。

### 什么是 Composer， 工作原理是什么？

Composer 是 PHP 的一个依赖管理工具。工作原理就是将已开发好的扩展包从 packagist.org composer 仓库下载到我们的应用程序中，并声明依赖关系和版本控制。



## 缓存

### Redis、Memecached 这两者有什么区别？

- Redis 支持更加丰富的数据存储类型，String、Hash、List、Set 和 Sorted Set。Memcached 仅支持简单的 key-value 结构。
- Memcached key-value存储比 Redis 采用 hash 结构来做 key-value 存储的内存利用率更高。
- Redis 提供了事务的功能，可以保证一系列命令的原子性
- Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中
- Redis 只使用单核，而 Memcached 可以使用多核，所以平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。

### Redis 如何实现持久化？

- RDB 持久化，将 redis 在内存中的的状态保存到硬盘中，相当于备份数据库状态。
- AOF 持久化（Append-Only-File），AOF 持久化是通过保存 Redis 服务器锁执行的写状态来记录数据库的。相当于备份数据库接收到的命令，所有被写入 AOF 的命令都是以 redis 的协议格式来保存的。



## 数据库

### 什么是索引，作用是什么？常见索引类型有那些？Mysql 建立索引的原则？

索引是一种特殊的文件,它们包含着对数据表里所有记录的引用指针，相当于书本的目录。其作用就是加快数据的检索效率。常见索引类型有主键、唯一索引、复合索引、全文索引。

### 高并发如何处理？

- 使用缓存
- 优化数据库，提升数据库使用效率
- 负载均衡

### 什么是事务？及其特性？

事务：是一系列的数据库操作，是数据库应用的基本逻辑单位。
特性：
（1）原子性：即不可分割性，事务要么全部被执行，要么就全部不被执行。
（2）一致性或可串性。事务的执行使得数据库从一种正确状态转换成另一种正确状态
（3）隔离性。在事务正确提交之前，不允许把该事务对数据的任何改变提供给任何其他事务，
（4） 持久性。事务正确提交后，其结果将永久保存在数据库中，即使在事务提交后有了其他故障，事务的处理结果也会得到保存。
简单理解：在事务里的操作，要么全部成功，要么全部失败。

### Mysql中有哪几种锁？

数据库是一个多用户使用的共享资源。当多个用户并发地存取数据时，在数据库中就会产生多个事务同时存取同一数据的情况。若对并发操作不加控制就可能会读取和存储不正确的数据，破坏数据库的一致性。
加锁是实现数据库并发控制的一个非常重要的技术。当事务在对某个数据对象进行操作前，先向系统发出请求，对其加锁。加锁后事务就对该数据对象有了一定的控制，在该事务释放锁之前，其他的事务不能对此数据对象进行更新操作。
基本锁类型：锁包括行级锁和表级锁

### 说说对SQL语句优化有哪些方法

1）Where子句中：where表之间的连接必须写在其他Where条件之前，那些可以过滤掉最大数量记录的条件必须写在Where子句的末尾.HAVING最后。
（2）用EXISTS替代IN、用NOT EXISTS替代NOT IN。
（3） 避免在索引列上使用计算
（4）避免在索引列上使用IS NULL和IS NOT NULL
（5）对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
（6）应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描
（7）应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描

### MySQL数据库，一天五万条以上的增量，预计运行三年，怎么优化？

（1）设计良好的数据库结构，允许部分数据冗余，尽量避免join查询，提高效率。
（2） 选择合适的表字段数据类型和存储引擎，适当的添加索引。
（3） 做mysql主从复制读写分离。
（4）对数据表进行分表，减少单表中的数据量提高查询速度。
（5）添加缓存机制，比如redis，memcached等。
（6）对不经常改动的页面，生成静态页面（比如做ob缓存）。
（7）书写高效率的SQL。比如 SELECT * FROM TABEL 改为 SELECT field_1, field_2, field_3 FROM TABLE.



## Web 篇

### SEO 有哪些需要注意的

合理的 title、description、keywords

语义化的 HTML 代码，符合 W3C 规范：语义化代码让搜索引擎容易理解网页

重要内容 HTML 代码放在最前：搜索引擎抓取 HTML 顺序是从上到下，有的搜索引擎对抓取长度有限制，保证重要内容一定会被抓取

重要内容不要用 js 输出：爬虫不会执行 js 获取内容

少用 iframe：搜索引擎不会抓取 iframe 中的内容

非装饰性图片必须加 alt

### CSS 选择器的分类

通配选择器、类选择器、ID选择器、属性选择器、伪类选择器、伪元素选择器

### display: none 与 visibility: hidden 的区别

display:none 会让元素完全从渲染树中消失，渲染的时候不占据任何空间；visibility: hidden 不会让元素从渲染树消失，渲染师元素继续占据空间，只是内容不可见

display: none 是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示；visibility: hidden 是继承属性，子孙节点消失由于继承了hidden，通过设置visibility: visible 可以让子孙节点显式

修改常规流中元素的 display 通常会造成文档重排。修改 visibility 属性只会造成本元素的重绘

读屏器不会读取 display: none 元素内容；会读取 visibility: hidden 元素内容

### flex 与 CSS 盒子模型有什么区别

布局的传统解决方案，基于盒状模型，依赖 display 属性 + position属性 + float属性。它对于那些特殊布局非常不方便，比如，垂直居中就不容易实现

Flex 布局，可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持，这意味着，现在就能很安全地使用这项功能

### 为什么把 JavaScript 文件放在 Html 底部

脚本会阻塞页面其他资源的下载，因此推荐将所有``标签尽可能放到``标签的底部，以尽量减少对整个页面下载的影响

### 如何解决跨域问题

- 通过 JSONP 跨域
- document.domain + iframe 跨域
- location.hash + iframe
- window.name + iframe 跨域
- postMessage 跨域
- 跨域资源共享（CORS）
- nginx 代理跨域
- nodejs 中间件代理跨域
- WebSocket 协议跨域

### 引起内存泄漏的操作有哪些

- 全局变量引起
- 闭包引起
- dom清空，事件未清除
- 子元素存在引用
- 被遗忘的计时器

### 对 JavaScript 模块化的理解

在ES6出现之前，js没有标准的模块化概念，这也就造成了js多人写作开发容易造成全局污染的情况，以前我们可能会采用立即执行 函数、对象等方式来尽量减少变量这种情况，后面社区为了解决这个问题陆续提出了AMD规范和CMD规范，这里不同于Node.js的 CommonJS的原因在于服务端所有的模块都是存在于硬盘中的，加载和读取几乎是不需要时间的，而浏览器端因为加载速度取决于网速， 因此需要采用异步加载，AMD规范中使用define来定义一个模块，使用require方法来加载一个模块，现在ES6也推出了标准的模块 加载方案，通过export和import来导出和导入模块。

### 如何判断网页中图片加载成功或者失败

使用onload事件运行加载成功，使用onerror事件判断失败

### 如何实现懒加载

懒加载就是根据用户的浏览需要记载内容，也就是在用户即将浏览完当前的内容时进行继续加载内容，这种技术常常用来加载图片的时候使用。我们判断用户是否即将浏览到底部之后进行在家内容 这时候可能会需要加载大量的内容，可以使用fragment来优化一下，因为大部分是使用滑动和滚轮来触发的，因此很有可能会不断触发，可以使用函数节流做一个优化，防止用户不断触发

### Vue.js 双向绑定原理

vue数据双向绑定是通过数据劫持结合发布者-订阅者模式的方式来实现的

### 如何进行网站性能优化

浏览器单域名并发数限制、静态资源缓存 304 （If-Modified-Since 以及 Etag 原理）、多个小图标合并使用 position 定位技术 减少请求、静态资源合为单次请求 并压缩、CDN、静态资源延迟加载技术、预加载技术、keep-alive、CSS 在头部，JS 在尾部的优化（原理）