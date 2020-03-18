# Redis

## Redis介绍

> Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
>
> 它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。
>
> Redis 与其他 key - value 缓存产品有以下三个特点：
>
> - Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
> - Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
> - Redis支持数据的备份，即master-slave模式的数据备份。
> - 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
> - 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
> - 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
> - 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。
>
> OS: ubuntu18.04
>
> Language： php7

## 安装Redis

```ubuntu
sudo apt-get install redis-server
```

## 查看Redis进程

```
ps -aux|grep redis
```

## 设置redis的访问密码

安装后配置文件路径： /etc/redis/redis.conf

修改 redis.conf, 去掉requirepass password 的#

```
requirepass 12345678
```

修改配置重启redis服务：

```
sudo service redis-server restart
```

访问时使用auth：

```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth('12345678');
```

## 安装php-redis扩展：

```
sudo pecl install redis
```

报错

```
checking use system liblzf... no
checking for igbinary includes... configure: error: Cannot find igbinary.h
ERROR: `/tmp/pear/temp/redis/configure --with-php-config=/usr/local/php/bin/php-config --enable-redis-igbinary=y --enable-redis-lzf=y' failed
```

此处是因为没有安装 igbinary 一个序列号与反序列化的php扩展

### 安装 igbinary lzf zstd 扩展

```
sudo pecl install igbinary 
sudo pecl install lzf 
sudo pecl install channel//pecl.php.net/zstd-0.8.0
```

### Ubuntu安装libzstd库

```makefile
apt update
apt install libzstd-dev
```

再执行

```
sudo pecl install redis
```

重启php-fpm： 查看php-info

![img](https://img2018.cnblogs.com/blog/540840/201903/540840-20190321103339202-681383649.png)

## redis设置key

```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->set('a','aaa');
```

## redis 读取key->value

```php
$a= $redis->get('a');
print_r($a);
```

## redis 设置集合

```php
$redis->sAdd('fd',1);
$redis->sAdd('fd',2);
$redis->sAdd('fd','333');
```

## redis读取集合

```php
foreach ($redis->sMembers('fd') as  $value) {
    print_r($value);
}
```

## redis 设置list列表

```php
$arr = array('h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd');
foreach ($arr as $v) {
    $redis->rpush("mylist", $v);
}
```

## redis 获取list列表

```php
$value = $redis->lpop('mylist');
print_r($value);
```

## redis 的实时消息订阅

发布消息文件

```php
$message= json_encode(['run=12','did=10001']);
$ret=$redis->publish('worker_channel',$message);
$ret=$redis->publish('worker_channel',"send_msg.hahaha");
```

订阅消息文件

```php
$result = $redis->subscribe(["worker_channel"], 'callback');
function callback($redis, $channel, $message) {
    if ($channel == 'w1') {
        echo '频道1';
    } else {
        echo $message;
    }
}
```

 但当订阅的文件执行60s后会报错：

```php
PHP Fatal error:  Uncaught RedisException: read error on connection in ***
Stack trace:
#0 ****: Redis->subscribe(Array, 'callback')
```

这是因为订阅默认60s超时就退出连接。解决办法：

设置-1 永不超时

```php
$redis->setOption(Redis::OPT_READ_TIMEOUT, -1);
```

## Redis与Memcached的区别

> 1、Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等；
>
> 2、Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储；
>
> 3、虚拟内存--Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘；
>
> 4、过期策略--memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10；
>
> 5、分布式--设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从；
>
> 6、存储数据安全--memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）；
>
> 7、灾难恢复--memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复；
>
> 8、Redis支持数据的备份，即master-slave模式的数据备份；