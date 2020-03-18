# Composer

## 镜像用法

有两种方式启用本镜像服务：

- **系统全局配置：** 即将配置信息添加到 Composer 的全局配置文件 `config.json` 中。见[“方法一”](https://pkg.phpcomposer.com/#tip1)
- **单个项目配置：** 将配置信息添加到某个项目的 `composer.json` 文件中。见[“方法二”](https://pkg.phpcomposer.com/#tip2)

### **方法一：** 修改 composer 的全局配置文件**（推荐方式）**

打开命令行窗口（windows用户）或控制台（Linux、Mac 用户）并执行如下命令：

```bash
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```



### **方法二：** 修改当前项目的 `composer.json` 配置文件：

打开命令行窗口（windows用户）或控制台（Linux、Mac 用户），进入你的项目的根目录（也就是 `composer.json` 文件所在目录），执行如下命令：

```bash
composer config repo.packagist composer https://packagist.phpcomposer.com
```

上述命令将会在当前项目中的 `composer.json` 文件的末尾自动添加镜像的配置信息（你也可以自己手工添加）：

```json
"repositories": {
    "packagist": {
        "type": "composer",
        "url": "https://packagist.phpcomposer.com"
    }
}
```

以 laravel 项目的 `composer.json` 配置文件为例，执行上述命令后如下所示（注意最后几行）：

```json
{
    "name": "laravel/laravel",
    "description": "The Laravel Framework.",
    "keywords": ["framework", "laravel"],
    "license": "MIT",
    "type": "project",
    "require": {
        "php": ">=5.5.9",
        "laravel/framework": "5.2.*"
    },
    "config": {
        "preferred-install": "dist"
    },
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "https://packagist.phpcomposer.com"
        }
    }
}
```

OK，一切搞定！试一下 `composer install` 来体验飞一般的速度吧！



## 镜像原理：

一般情况下，安装包的数据（主要是 zip 文件）一般是从 `github.com` 上下载的，安装包的元数据是从 `packagist.org` 上下载的。

然而，由于众所周知的原因，国外的网站连接速度很慢，并且随时可能被“墙”甚至“不存在”。

“Packagist 中国全量镜像”所做的就是缓存所有安装包和元数据到国内的机房并通过国内的 CDN 进行加速，这样就不必再去向国外的网站发起请求，从而达到加速 `composer install` 以及 `composer update` 的过程，并且更加快速、稳定。因此，即使 `packagist.org`、`github.com` 发生故障（主要是连接速度太慢和被墙），你仍然可以下载、更新安装包。



## 解除镜象：

如果需要解除镜像并恢复到 packagist 官方源，请执行以下命令：

```bash
composer config -g --unset repos.packagist
```

执行之后，composer 会利用默认值（也就是官方源）重置源地址。

将来如果还需要使用镜像的话，只需要根据前面的“镜像用法”中介绍的方法再次设置镜像地址即可。



## 阿里云 Composer 全量镜像

https://developer.aliyun.com/composer?accounttraceid=8a144b2d82cf499b895cf43ac75e18a5buvh

### 全局配置（推荐）

所有项目都会使用该镜像地址：

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

取消配置：

```bash
composer config -g --unset repos.packagist
```



### 项目配置

仅修改当前工程配置，仅当前工程可使用该镜像地址：

```bash
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```

取消配置：

```bash
composer config --unset repos.packagist
```



### 调试

composer 命令增加 -vvv 可输出详细的信息，命令如下：

```bash
composer -vvv require alibabacloud/sdk
```



### 遇到问题？

1. 建议先将Composer版本升级到最新：

```bash
composer self-update
```

2. 执行诊断命令：

```bash
composer diagnose
```

3. 清除缓存：

```bash
composer clear
```

4. 若项目之前已通过其他源安装，则需要更新 composer.lock 文件，执行命令：

```bash
composer update --lock
```



## Composer国内加速，修改镜像源

### 为什么慢

由于默认情况下执行 composer 各种命令是去国外的 composer 官方镜像源获取需要安装的具体软件信息，所以在不使用代理、不翻墙的情况下，从国内访问国外服务器的速度相对比较慢

### 如何修改镜像源

可以使用阿里巴巴提供的 Composer 全量镜像 https://mirrors.aliyun.com/composer/

#### a). 配置只在当前项目生效

```shell
composer config repo.packagist composer https://mirrors.aliyun.com/composer/

# 取消当前项目配置
composer config --unset repos.packagist
```

#### b). 配置全局生效

```shell
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 取消全局配置
composer config -g --unset repos.packagist
```

#### c). 使用第三方软件快速修改、切换 composer 镜像源

[crm](https://github.com/slince/composer-registry-manager) composer registry manager

##### 安装 crm

```shell
composer global require slince/composer-registry-manager
```

##### 列出可用的所有镜像源，前面带 * 代表当前使用的镜像

```bash
composer repo:ls

-- --------------- ------------------------------------------------
     composer        https://packagist.org
     phpcomposer     https://packagist.phpcomposer.com
     aliyun          https://mirrors.aliyun.com/composer
     tencent         https://mirrors.cloud.tencent.com/composer
     huawei          https://mirrors.huaweicloud.com/repository/php
     laravel-china   https://packagist.laravel-china.org
     cnpkg           https://php.cnpkg.org
     sjtug           https://packagist.mirrors.sjtug.sjtu.edu.cn
-- --------------- ------------------------------------------------

```

##### 使用 aliyun 镜像源

```bash
composer repo:use aliyun

# 执行成功之后会输出类似以下信息
[OK] Use the repository [aliyun] success
```

##### 再次执行 composer repo:ls 命令，看到前面带 * 的就是当前使用的镜像

```shell
composer repo:ls

# 可以看到 aliyun 前面有一个 * 号，代表当前使用的是 aliyun 的源
--- --------------- ------------------------------------------------
      composer        https://packagist.org
      phpcomposer     https://packagist.phpcomposer.com
  *   aliyun          https://mirrors.aliyun.com/composer
      tencent         https://mirrors.cloud.tencent.com/composer
      huawei          https://mirrors.huaweicloud.com/repository/php
      laravel-china   https://packagist.laravel-china.org
      cnpkg           https://php.cnpkg.org
      sjtug           https://packagist.mirrors.sjtug.sjtu.edu.cn
--- --------------- ------------------------------------------------
```

https://github.com/slince/composer-registry-manager