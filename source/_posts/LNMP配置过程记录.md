---
title: LNMP配置过程记录
date: 2019-07-09 21:25:45
tags: [linux, nginx]
---

我最近从把服务器从CentOS迁移到了Ubuntu([我为什么选择放弃CentOS](https://feng.si/posts/2019/07/centos-the-last-linux-distro-you-should-ever-consider/)), 本文记录一下其中踩的坑.

## 使用OneinStack配置LNMP

我是使用的一键包配置的LNMP, 自己手动配置比较繁琐, 也容易有失当之处. 我使用的是OneinStack, Github有[开源项目](https://github.com/oneinstack/lnmp), 官网是`https://oneinstack.com/`.

我使用如下的配置自动安装:

```bash
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ./oneinstack/install.sh --nginx_option 1 --php_option 4 --phpcache_option 1 --phpmyadmin  --db_option 4 --dbinstallmethod 1 --dbrootpwd <passwd>
```

基于性能考虑, 我使用了MySQL 5.5 和 PHP 5.6, 分别可以通过`service mysqld`和`service php-fpm`管理. 此外, Nginx通过`service nginx`管理.
<!-- more -->

## 补充安装fileinfo模块

因为安装php fileinfo模块太过于吃内存, Oneinstack一键包在安装fileinfo过程中是不包含fileinfo的, 所以需要自己再手动安装:

```bash
./install.sh --php_extensions fileinfo
```

## 手动链接PHP

估计是为了多版本PHP共存的考量, 一键包并没有为PHP设置环境变量, 于是需要手动设置一下:

```bash
ln -s /usr/local/php/bin/php /usr/local/bin/php
ln -s /usr/local/php/bin/php /usr/bin/php
```

## 设置网站根目录权限

设置根目录权限以方便引用的执行:

```bash
chown -R www.www /data/wwwroot/
find /data/wwwroot/ -type d -exec chmod 755 {} \;
find /data/wwwroot/ -type f -exec chmod 644 {} \;
```

对于某些应用, 可以还需要手动`chmod 777`以解决权限问题.

## 解决PHP disabled function

PHP默认是禁用了一部分函数的(出于安全的考量), 例如我遇到的就是`passthru()`函数被禁用了, 需要手动配置.

通过`phpinfo()`此函数创建的网页找到PHP配置文件`php.ini`的位置, 并修改此文件, 查找`disable_functions`项, 手动去掉自己需要的函数.

## 个人感想

我自己新部署的服务器是Ubuntu 18.04 LTS版本, 和CentOS相比更像是一个**智能完备**的OS. 例如, 同时自带Python2和Python3(CentOS7只包含前者), apt可以直接安装PHP7.2(CentOS官方只提供PHP5.4), 自带vim/lrzsz/git等常用命令, Ubuntu的Linux内核领先好多个版本(直接影响是不需要升级内核就能开启BBR加速). 不过CentOS最受人诟病的就是陈旧无比的官方源, 换了社区源又容易出现一些其他问题, 没有Ubuntu这样有商业公司维护的令人安心.

我原先是在CentOS上手动搭建的LAMP, 现在替换到了Ubuntu上一键包的LNMP, 最直观感受是Nginx相比Apache在速度上确实是有直观的提升的. 其次, 一键包进行部署虽然为了抹平Linux版本之间的差异采用的是源码编译安装的方式, 需要的安装时长在20-50分钟不等, 但是确实很方便, 安装的过程不需要任何操作, 且自带的探针/phpMyAdmin/Opcache缓存确实非常方便. 还能通过`vhost.sh`脚本快速地添加虚拟主机并自动配置TLS证书, 确实能够做到*开箱即用*, 体验上算是很不错的.