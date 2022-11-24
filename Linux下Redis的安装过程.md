---
title: Linux下Redis的安装过程
date: 2016-05-21 10:50:40
tags: 
- redis
- CentOS
- Linux
categories:
- 开发笔记
---
### 前言

在linux操作系统下安装redis的教程，网上已经烂大街了，笔者觉着自己在配置过程中有些操作不太理解，所以在此重新梳理一下安装过程，并对一些要点着重进行说明。

笔者的服务器为CentOS 6.5，redis下载的版本为当前最新的稳定版本。

### 下载redis

下载redis的方式主要有两种，一种是自己下载tar.gz包，然后上传到服务器上进行解压缩。另一种是通过wget命令直接下载redis的压缩包到服务器，然后解压缩。

``` Shell
 wget http://download.redis.io/releases/redis-3.0.5.tar.gz
```

看到此处时，根据自己的需要去选择redis的版本。

<!-- more -->

### 编译安装redis

下载好redis的tar.gz包后，在指定的位置进行解压缩：

``` Shell
 tar -xzvf redis-3.0.5.tar.gz
```

进入解压缩后的程序目录，使用make对redis进行编译：

``` Shell
 cd redis-3.0.5

 make
```

执行完make后，会在src目录中发现生成了几个可执行文件，如：

- redis-server：redis服务器启动程序
- redis-cli：Redis客户端操作工具。也可以用telnet根据其纯文本协议来操作
- redis-benchmark：Redis性能测试工具
- redis-check-aof：数据修复工具
- redis-check-dump：检查导出工具

手动将上述的执行文件复制到/usr/local/bin下，即可在任意位置执行这些redis命令了，不需要每次执行都要带上执行文件的路径位置了。

或者你还可以执行make install命令，默认会自动将生成的可执行文件复制到/usr/local/bin下，你也可以通过命令参数来指定安装的目录位置，具体查看redis程序目录下的ReadMe文件。


> 可能你会问，linux下安装程序，一般都是需要./configure,make和make install三个步骤，为何这里没有configure？那是因为configure是程序安装目录下的一个可执行脚本，一般用来生成 Makefile，为下一步的编译做准备。而redis的安装包中已经存在一个Makefile，所以自然省去了configure这步。

在make命令执行后，你可以选择不执行make install命令，自己手动的将程序目录下src目录中的对应可执行文件copy到/usr/bin下，这招是从oschina的红薯[那里](http://www.oschina.net/question/12_18065?fromerr=uNX17fsi)学来的。

当然你可能会有疑惑，make install是安装到了/usr/local/bin下，而自己copy为何在/usr/bin下。其实这两个不同的位置都可以起到同一个作用，在任何位置执行redis相关命令。

- /usr/bin下面的通常都是系统预装的可执行程序，会随着系统升级而改变
- /usr/local/bin目录是给用户放置自己的可执行程序的地方，推荐放在这里，不会被系统升级而覆盖同名文件
- 如果两个目录下有相同的可执行程序，谁优先执行受到PATH环境变量的影响

### 配置redis

将redis的配置文件复制到/etc/目录下：

``` Shell
cp redis.conf /etc/
```

为了让redis后台运行，修改redis.conf文件，修改daemonize配置项为yes：

``` Shell
vi /etc/redis.conf
```

### 运行redis

完成上述步骤后，启动redis：

``` Shell
redis-server /etc/redis.conf
```

检查redis是否启动成功：

``` Shell
ps -ef | grep redis
```

看到类似下面的一行，表示启动成功：

``` Shell
root     18443     1  0 13:05 ?        00:00:00 ./redis-server *:6379
```

至此redis已经安装运行完毕，下载的redis安装包和解压缩出来的目录可以删除了。

如果需要设置redis开机启动项，点击[这里](http://itbilu.com/linux/management/4kB2ninp.html)。

### redis重点配置项说明

- daemonize：是否以后台daemon方式运行
- pidfile：pid文件位置
- port：监听的端口号
- timeout：请求超时时间
- loglevel：log信息级别
- logfile：log文件位置
- databases：开启数据库的数量
- save \* \*：保存快照的频率，第一个*表示多长时间，第三个*表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存快照。可设置多个条件
- rdbcompression：是否使用压缩
- dbfilename：数据快照文件名（只是文件名）
- dir：数据快照的保存目录（仅目录）
- appendonly：是否开启appendonlylog，开启的话每次写操作会记一条log，这会提高数据抗风险能力，但影响效率
- appendfsync：appendonlylog如何同步到磁盘。三个选项，分别是每次写都强制调用fsync、每秒启用一次fsync、不调用fsync等待系统自己同步

### 最佳配置实践

进入到redis的解压目录下，开始进行如下的配置操作：

- 创建配置文件和数据文件的存储位置

``` Shell
sudo mkdir /etc/redis
sudo mkdir /var/redis
```

- 复制redis解压包中的启动脚本到操作系统的自启动目录下，建议重命名文件时加上端口号

``` Shell
sudo cp ./utils/redis_init_script /etc/init.d/redis_6379
```

- 编辑自启动的redis脚本

``` Shell
sudo vi /etc/init.d/redis_6379
```

将其中**REDISPORT**改为实际使用的端口号。仔细查看脚本中的代码，会发现其中pid文件路径和配置文件名称都依赖于端口号。

- 复制配置文件到etc下

``` Shell
sudo cp ./redis.conf /etc/redis/6379.conf
```

可以看到，实际使用的配置文件以端口号来命名，这样自启动脚本执行时就会使用这个配置文件。

- 创建redis的工作数据目录，建议以端口进行区分

``` Shell
sudo mkdir /var/redis/6379
```
