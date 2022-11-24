---
title: Ubuntu下快速安装Nginx
date: 2016-06-30 16:14:40
tags:
- Nginx
- Ubuntu
- Linux
categories: 
- 开发笔记
---
### 前言

Linux操作系统有很多不同的发行版本，像Ubuntu、CentOS、RedHat等等，有些是收费的，有些是免费的。接触过Linux的人多少对这些信息有些了解。虽然版本不同，但大部分系统命令还是一致的，可能会有细微的差别。

今天笔者就经历一把不同版本的Linux操作系统中安装nginx。笔者之前写过一篇CentOS下安装nginx的文章，这次由于客户那边换了个新的服务器，Ubuntu的，没办法，再来一遍~对比两次的安装经历，诞生了这篇文章。

### Nginx安装

虽然发行版本不同，但说白了还都是Linux的系统，所以系统命令没有太大差别。安装nginx，我们可以选择纯手工方式和自动化方式。

纯手工方式就是自己下载tar.gz包，然后编译安装。这种方式在CentOS和Ubuntu下是无差别的。

自动化方式即利用系统中的软件库，通过快捷命令实现一键安装。CentOS常见的是yum，而Ubuntu中方便的的是apt。这里可以理解为windows操作系统中的360软件管家，里面有许多现成的软件供我们一键安装到位！

笔者是个懒人~自然还是选择自动化的方式。

原先在CentOS下靠的是yum，结果到Ubuntu下一看，没有！网上帖子和文章还是以apt居多，那自然选用apt走着。

但笔者突然先发现使用apt安装nginx的命令中并没有指定版本号，所以笔者好奇自动安装的nginx版本是啥呢？

<!-- more -->

``` Shell
# sudo apt-cache policy <packagename>
```

将<packagename>换成nginx一看，版本是1.4.6。笔者觉着有点低，怎么样才能利用apt安装最新的版本呢？根据查看nginx官网的安装说明和其他文章作为验证，总结出以下的快速安装方法，而且是最新版本呦。

1 从Nginx官网下载供apt程序认证使用的key，具体说明点[这里](http://nginx.org/en/linux_packages.html#stable)。

``` Shell
# cd /tmp
# wget http://nginx.org/keys/nginx_signing.key
```

2 将下载好的认证key添加到apt程序的key中。

``` Shell
# sudo apt-key add nginx_signing.key
```

3 向/etc/apt/sourses.list文件中追加如下内容：

> deb http://nginx.org/packages/mainline/ubuntu/ {codename} nginx

> deb-src http://nginx.org/packages/mainline/ubuntu/ {codename} nginx

上述内容中的codename需要替换成Nginx官方指定的值，这里需要先查看下Ubuntu的版本是啥？

``` Shell
# lsb_release -a
```

笔者的Ubuntu版本是14.04，对照[这里](http://nginx.org/en/linux_packages.html#distributions)，codename应该换成trusty。所以最终内容应该是：

> deb http://nginx.org/packages/mainline/ubuntu/ trusty nginx

> deb-src http://nginx.org/packages/mainline/ubuntu/ trusty nginx

记得保存退出文件哦~

4 执行apt安装的命令

``` Shell
# sudo apt-get update
# sudo apt-get install nginx
```

搞定！当然如果不介意nginx的版本，可以直接进行第4步，安装过程更快了一步。

### 测试运行

依靠apt自动安装过nginx后，文件的结构大致如下：

- 所有的配置文件在/etc/nginx目录下。
- 执行程序文件在/usr/sbin/nginx目录下。
- 日志放在/var/log/nginx目录下。

自动化安装方式默认已经在/etc/init.d下创建了nginx的启动脚本，所以可以很方便地使用如下的命令来操纵nginx服务。

``` Shell
# service nginx {start|stop|status|restart|reload|configtest}
```
