---
title: LNMP配置过程记录
date: 2019-07-09 21:25:45
tags: [linux, nginx]
---

我最近从把服务器从 CentOS 迁移到了 Ubuntu（[我为什么选择放弃CentOS](https://feng.si/posts/2019/07/centos-the-last-linux-distro-you-should-ever-consider/)），本文记录一下其中踩的坑。

## 使用OneinStack配置LNMP

我是使用的一键包配置的LNMP，自己手动配置比较繁琐，也容易有失当之处。我用的一键包是OneinStack，Github有[开源项目](https://github.com/oneinstack/lnmp)，官网是 `https://oneinstack.com/`。

我使用如下的配置自动安装:

```bash
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ./oneinstack/install.sh --nginx_option 1 --php_option 4 --phpcache_option 1 --phpmyadmin  --db_option 4 --dbinstallmethod 1 --dbrootpwd <passwd>
```

基于性能考虑，我使用了MySQL 5.5 和 PHP 5.6，分别可以通过 `service mysqld` 和 `service php-fpm` 管理。此外，Nginx通过 `service nginx` 管理。
<!-- more -->

## 补充安装fileinfo模块

因为安装php fileinfo模块太过于吃内存，Oneinstack一键包在安装过程中是不包含fileinfo的，需要自己手动安装:

```bash
./install.sh --php_extensions fileinfo
```

## 手动链接PHP

估计是为了多版本PHP共存的考量，一键包并没有为PHP设置环境变量，于是需要手动设置一下:

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

对于某些应用，可能还需要手动 `chmod 777` 以解决权限问题。

## 解决PHP disabled function

PHP出于安全的考量，默认是禁用了一部分函数的，例如我遇到的就是 `passthru()` 函数被禁用了，需要手动配置。

通过 `phpinfo()` 此函数创建的网页找到PHP配置文件 `php.ini` 的位置，并修改此文件，查找`disable_functions` 项，手动去掉自己需要的函数。

## 个人感想

我自己新部署的服务器是 Ubuntu 18.04 LTS版本，和 CentOS 相比更**完备**、更**省心**。例如，同时带有 Python2 和 Python3（CentOS 7 只包含前者）；apt可以直接安装PHP 7.2（CentOS官方包管理器只提供PHP 5.4）；自带vim、git、lrzsz等常用命令；Linux内核领先好多个版本（直接影响是不需要升级内核就能开启BBR加速）。CentOS最受人诟病的就是陈旧无比的官方源，换了社区源又容易出现一些其他问题。倒不如直接使用 Ubuntu，背后有商业公司提供免费支持，出 bug 的概率比较低。

我原先是在 CentOS 上手动搭建的 LAMP，现在替换到了 Ubuntu 上一键包的 LNMP，最直观的感受是 Nginx 相比 Apache 在速度上确实是有直观的提升的。其次，虽然一键包部署为了抹平 Linux 版本之间的差异，采用的是源码编译安装的方式，需要的安装时长在 20-50 分钟不等，但是确实很方便。安装的过程中不需要任何操作，且自带的探针、phpMyAdmin、Opcache缓存确实非常好用。此外还能通过 `vhost.sh` 脚本快速地添加虚拟主机并自动配置TLS证书，能够做到*开箱即用*，体验上是很棒的。