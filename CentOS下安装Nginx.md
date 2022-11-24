---
title: CentOS下安装Nginx
date: 2016-05-23 09:29:40
tags: [Nginx,CentOS,Linux]
categories: [开发笔记]
---
### 前言

项目需要，自己整个nginx玩玩，部署服务器的操作系统为CentOS 6.5。

### nginx安装

上网搜了一下关于nginx的安装教程，大致存在两种安装方法，都是通过yum install命令来完成。

方法一：自己选择下载安装nginx依赖的其他包，然后下载nginx的安装包，编译安装即可。这种方式的好处是扩展性强，往后需要安装其他模块都是可控的。不妥之处是步骤繁琐，依赖包多，需要牢记。

可参照如下教程：

- [nginx服务器安装及配置文件详解](http://seanlook.com/2015/05/17/nginx-install-and-config/)
- [nginx 安装配置](http://www.runoob.com/linux/nginx-install-setup.html)
- [CentOS nginx安装与配置](http://www.jianshu.com/p/d5114a2a2052)

方法二：由于CentOS下，yum源不提供nginx的安装，所以可以通过添加yum源进行快速安装。这种方式的好处是快，方便。不妥之处是不可控，都是别人准备好的东西进行的“一键安装”，今后想安装第三方模块也无从下手。而且笔者目前对这种方式也不是特别理解，所以可能出了差错，也爱莫能助。

<!-- more -->

可参照如下教程：

- [CentOS 6.5 nginx安装与配置](https://gist.github.com/ifels/c8cfdfe249e27ffa9ba1)

#### 小结

如果你已经浏览过上述提供的安装方式后，这里可以对nginx的安装过程进行一些总结。

其实概括地来讲，安装nginx的步骤为两步，先安装nginx依赖的库，然后安装nginx。安装方式无非两种，如果yum源中存在需要的库，可以方便地使用yum来安装；如果yum源中没有需要的库，那么只能自己下载对应的安装包，然后执行编译安装命令。当然你也可以完全不使用yum，自己下载所有需要的源码安装包，然后编译安装。

### nginx运行

假设nginx安装在/usr/local/nginx/sbin下：

``` Shell
# cd /usr/local/nginx
```

#### 启动nginx

``` Shell
# ./sbin/nginx        # 默认配置文件 conf/nginx.conf，-c 指定
```

#### 停止nginx

``` Shell
# ./sbin/nginx -s stop
```

或者

``` Shell
# pkill nginx
```

#### 重新加载配置文件

reload用于配置文件改变后的nginx刷新应用，没必要一定得停止nginx。reload不会改变启动时指定的配置文件位置。

``` Shell
# ./sbin/nginx -s reload
```

#### 系统服务

当然如果将nginx设置为linux系统服务，那么就可以方便使用下面的命令进行上述的操纵：

``` Shell
# service nginx {start|stop|status|restart|reload|configtest}
```

笔者使用方法二进行的安装，所以nginx会自动被安装为系统服务了。你也可以搜索如何设置nginx为linux服务。

### 其他相关命令

#### 查看系统中指定端口号是否被占用

``` Shell
# netstat -apn|grep <端口号>
```
