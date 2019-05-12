---
layout:     post
title:      搭建owncloud私有云盘
subtitle:   阿里云ESC,ubuntu16.04下快速搭建私有云盘
date:       2019-05-12
author:     Jorian
header-img: img/post-bg-computer.jpg
catalog: true
tags:
    - 私有云盘
    - Apache
---

>在云服务器下搭建无限速私有云盘
>
>更好的隐私，更便捷的服务
>
>基于owncloud实现全平台支持服务

## 前言

ownCloud是一个开源的web套件，它通过网络提供云存储。数据可以通过浏览器上传/下载，也可以通过软件客户端免费下载。ownCloud基于PHP，可以在所有满足需求的平台上运行。它提供了商业套件中几乎所有的功能; ownCloud在AGPLv3许可下发布，所以我们通过它来搭建自己的云存储服务器，而不需要任何额外的成
本。ownCloud基于PHP和数据库的组合。数据库可以是SQLite、MySQL、Oracle或PostgreSQL。


![OwnCloud示例图片](https://i.loli.net/2019/05/12/5cd7f165d884d.png)




## 搭建环境

- 系统：ubuntu16.04
- 数据库：MariaDB
- PHP: PHP 7.0
- web服务：Apache2
- ownCloud: 10.0.10



## 安装环境依赖

##### 安装PHP扩展

```
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get -y install libapache2-mod-php php-gd php-json php-mysql php-curl php-intl php-mcrypt php-imagick php-zip php-xml php-mbstring
```

#### 安装wget,apache2,mariadb-server

```
sudo apt-get -y install wget apache2 mariadb-server 
```

#### 配置安装源

```
sudo wget -nv https://download.owncloud.org/download/repositories/stable/Ubuntu_16.04/Release.key -O Release.key
sudo apt-key add - < Release.key 
sudo sh -c "echo 'deb http://download.owncloud.org/download/repositories/stable/Ubuntu_16.04/ /' > /etc/apt/sources.list.d/owncloud.list"
```



## 安装ownCloud

```
sudo apt-get update
sudo apt-get install owncloud-files
```



## 创建数据库

```
sudo mysql -u root -p
create database ownclouddb;
grant all on ownclouddb.* to 'ownclouduser'@'localhost' identified by 'password';
FLUSH PRIVILEGES;
exit
```



## 配置apache2

#### 为ownCloud创建站点虚拟环境

```
sudo vi /etc/apache2/sites-available/owncloud.conf
```

#### 编辑owncloud.conf

```
Alias /owncloud "/var/www/owncloud/"
<Directory /var/www/owncloud/>
Options +FollowSymlinks
AllowOverride All

<IfModule mod_dev.c>
Dav off 
</IfModule>

SetEnv HOME /var/www/owncloud
SetEnv HTTP_HOME /var/www/owncloud
</Directory>
```
若端口被80端口被占用或者限制开放，可以`owncloud.conf`中进行如下配置（不同端口对应不同站点）  
**注意：云服务器安全组配置中需开放相应端口**
```
<VirtualHost *:8090>
		 #8090为配置端口号
         # The ServerName directive sets the request scheme, hostname and port that
         # the server uses to identify itself. This is used when creating
         # redirection URLs. In the context of virtual hosts, the ServerName
         # specifies what hostname must appear in the request's Host: header to
         # match this virtual host. For the default virtual host (this file) this
         # value is not decisive as it is used as a last resort host regardless.
         # However, you must set it for any further virtual host explicitly.
         #ServerName www.example.com
 
         ServerAdmin webmaster@localhost
         DocumentRoot /var/www/html
 
         # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
         # error, crit, alert, emerg.
         # It is also possible to configure the loglevel for particular
         # modules, e.g.
         #LogLevel info ssl:warn
 
         ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined
 
         # For most configuration files from conf-available/, which are
         # enabled or disabled at a global level, it is possible to
         # include a line for only one particular virtual host. For example the
         # following line enables the CGI configuration for this host only
         # after it has been globally disabled with "a2disconf".
         #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

<Directory /var/www/owncloud/>
	Options +FollowSymlinks
	AllowOverride All

<IfModule mod_dev.c>
	Dav off
</IfModule>
 
SetEnv HOME /var/www/owncloud
SetEnv HTTP_HOME /var/www/owncloud
</Directory>
```

`/etc/apache2/ports.conf`添加监听端口号

```
Listen 8090 #新添加的端口号
```

#### 激活站点配置及相关模块

```
sudo a2ensite owncloud
sudo a2enmod rewrite
sudo a2enmod headers

sudo systemctl restart apache2
```


## 配置ownCloud

1. 通过以上操作，ownCloud已经安装完成，通过浏览器打开服务器IP地址对ownCloud进行配置。
2. 创建管理员账号。
3. 配置数据目录。 默认路径为/var/www/owncloud/data。
4. 配置数据库。填入前面我们创建的数据库名称及用户名密码。
5. 以上信息填写完毕以后，点击"安装完成"。

![conf-ownCloud.jpg](https://i.loli.net/2019/05/12/5cd7fa302be35.jpg)


## 登录ownCloud

![login-ownCloud.jpg](https://i.loli.net/2019/05/12/5cd7faa7c16b3.jpg)



**忘记管理员密码？？？😭**  
嘿嘿，不用着急



## 重置密码

查看用户：
```
sudo -u www-data php occ user:list
```
重置用户jorian的密码
```
sudo -u www-data php occ user:resetpassword jorian
```



> 参考链接：
- [ownCloud/Nextcloud使用OCC命令重置密码](https://www.orgleaf.com/2147.html)
- [基于Ubuntu搭建ownCloud](https://www.kclouder.cn/ubuntu-owncloud/)
- [Apache配置文件httpd.conf详解](https://www.jianshu.com/p/c36dd3946e74)
- [ownCloud/Nextcloud OCC命令行工具详解](https://blog.csdn.net/qq_33468857/article/details/85869001)

