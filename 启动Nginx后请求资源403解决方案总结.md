---
title: 启动Nginx后请求资源403解决方案总结
date: 2016-08-27 09:29:40
tags:
- Nginx
- CentOS
- Linux
categories:
- 系统运维
---
### 前言

最近在测试服务器上安装并启动Nginx来运行项目，遇到了之前未见过的403错误。

测试服务器是CentOS的系统，上网搜索了一些办法，大多数都是说web目录权限不够，或者是项目根目录下没有index索引文件。

恰恰笔者的问题并不是常见的两种情况，不过还是有人提到了第三种特殊情况。极少数的特殊情况，时间长了真的容易忘记，为了避免重复踩坑，有必要总结一下。

### 403解决方案

大体上Nginx出现403的原因有三种，每种都有对应的解决办法。常见的两种原因就是权限问题或index文件缺失。这里假设web目录为“/www”，项目目录为"/www/OA"。

<!-- more -->

- 如果是Nginx对web目录没有操作权限，解决办法如下：

  1. 修改web目录的读写权限。
  ``` Shell
  chmod -R 755 /www
  ```
  2. 将Nginx的启动用户改为web目录的所属用户，修改/etc/nginx下nginx.conf文件的user项。

-  如果是项目目录下缺少index索引文件，解决办法如下：

  在项目根目录下创建index.html或index.php文件，一般情况下就是这两个文件了，具体还得看Nginx项目配置文件中具体是如何指定的了。
  ```
  server {
    listen       80;
    server_name  localhost;
    index  index.php index.html;
    root  /www;
  }
  ```

- 第三种原因比较特殊，查看服务器上是否开启了SELinux。

  SELinux如果是enabled，会产出Nginx的403问题。查看服务器SELinux状态命令如下：

  ```Shell
  /usr/sbin/ sestatus -v
  ```

  确定是SELinux的原因后，可以选择关闭SELinux，有两种方式：

  1. 临时关闭，不需要重启服务器，但是如果服务器重启后会失效。
  ```Shell
  setenforce 0
  ```
  2. 修改配置文件 /etc/ selinux/config，将SELINUX=enforcing改为SELINUX=disabled，注意修改后需要重启系统。
