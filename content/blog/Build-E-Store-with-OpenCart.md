---
title: 使用Opencart来搭建私家电商网站
date: 2014-07-20T09:20:02.322Z
---

本文主要介绍在Mac环境下如何构建LAMP环境，然后基于Opencart来构建私家电商网站.

### 1. LAMP on Mac

Mac 作为Unix系列最耀眼的孩子之一，拥有Unix的优良血统，这点真的很重要，也就是因为这一点，Mac和Linux一样成为程序员的最佳工作平台。
好吧，闲话依然少叙，最近准备接一个小项目，是一个电商网站，我们准备基于[Opencart](http://www.opencart.com/),由于Opencart本身是php
的框架，所以\*AMP(Linux/Unix, Apache, MySql, PHP)是必须的基础架构了，所以让们来看看Mac下如何安装AMP吧。

#### Apache
Mac 自带有Apache,所以做一些简单的设置，就可以满足我们的需求了.

```ruby
$  apachectl -v
Server version: Apache/2.2.26 (Unix)
Server built:   Dec 10 2013 22:09:38
```

Apache 的默认主目录是在home，我们可以设定在适合我们方便开发的地方。

```ruby
$ cd ~/
$ mkdir Sites
```

然后修改/etc/apache2/httpd.conf让主目录为~/Sites

```ruby
DocumentRoot “/Users/huangmh/Sites/”
<Directory “/Users/huangmh/Sites”>
     Options Indexes MultiViews
     AllowOverride All
     Order allow,deny
     Allow from all
</Directory>
```

然后重启我们的Apache就好了。

```ruby
$ sudo apachectl restart
```

#### PHP

Mac 本身也自带了php了，很多人为了良好的控制版本，都采用[brew](http://brew.sh/) 来重新安装，brew
这是包管理工具中的皎皎者，太好用了，所以Ruby的社区是最才华横溢的，对，没错，我就是Ruby的忠诚粉丝。
但是我们使用Mac 本身自带的php就好了。但是值得注意的是，如果安装php的一些拓展的话，最好才看一下php的
版本。

```ruby
$ php -v
 PHP 5.4.24 (cli) (built: Jan 19 2014 21:32:15) 
 Copyright (c) 1997-2013 The PHP Group
 Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies
```

当发现自己安装的有些模块没有load的好的话，那么可以在查看/etc/php.ini文件是否以及load相应的.so文件.

#### MySql

MySqle, Mac 是没有预装的，我们可以通过万能的brew来安装.

```ruby
$ brew install mysql
```

之后设定一下root用户密码。

```ruby
$ mysql -u root -p
```

至此，我们就完成了AMP的安装，可以进行Opencart安装了。

### 2. 安装Opencart

Opencart 的安装和Wordpres一样，都非常的简单，在基础环境（也就是Apache, PHP, Mysql) 已经具备的情况下, 安装一般都不会有什么问题。
其实只要确定你已经设置好了MySql和PHP的相关拓展，那么安装就是剩下点击鼠标的事情了。好吧，但是还是让我们慢慢来试试吧。

### MySql

Opencart 的相关数据库设置，首先我们确定一下用户名和密码，这会成为以后Opencart的专用数据库，也是以后管理员经常要进行交互的。
让我们来创建一下opencart的数据库和用户吧

```ruby
$mysql -u root -p # 输入你的root密码之后进入mysql的shell

mysql> CREATE DATABASE opencart; # 创建数据库
mysql> CREATE USER opencartuser@localhost; # 创建用户
mysql> SET PASSWORD FOR opencartuser@localhost= PASSWORD(“输入你的密码”); # 创建用户的密码
mysql> GRANT ALL PRIVILEGES ON opencart.* TO opencartuser@localhost IDENTIFIED BY ‘你的密码’; # 授权给用户
mysql> FLUSH PRIVILEGES; # 是权限生效
```
这样就拥有了一个Opencart的数据库。

### Opencart的安装

从［github](http://www.github.com/)上直接clone下来Opencart就好了

```ruby
$ git clone https://github.com/opencart/opencart.git
```

然后将upload的目录移到我们的Apache主目录下方即可

```ruby
$ mv opencart/upload ~/Sites/opencart
$ cd ~/Sites/opencart/
$ cp config-dist.php config.php
$ cp admin/config-dist.php admin/config-dist.php
```

然后访问 http://localhost/opencart, 此时，你立即可以看到你的安装环境是否已经具备，如果没有具备，作为相关
的设置之后即可完成安装了。

### 后续
将研究其源代码，进行相关的定制。

Opencart作为php中比较成熟的MVC框架，阅读起来应该还可以。

