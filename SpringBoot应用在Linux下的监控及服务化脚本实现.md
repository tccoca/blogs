---
title: SpringBoot应用在CentOS下的监控及服务化脚本实现
date: 2017-1-17 09:29:40
tags:
- CentOS
- Linux
- Spring Boot
categories:
- 系统运维
---
### 前言
近期由于服务器内存配置过低，其中仍有不断更新上线的应用功能，新服务器未落定的情况下，本着“艰苦朴素”的优良传统，暂时关闭了一些使用率不高的服务。

但是由于内存吃紧，为了保障高使用率的服务正常运行，需要想办法监控服务运行状态，一旦发现服务停止，需要自动重启。

笔者首先想到了Shell脚本，真的是好久没写过脚本了~~~

基本解决思路：通过常用的ps命令，查看对应的服务是否运行，如果未运行，执行运行的命令，记录日志。将所有的命令写入一个脚本，然后交由CentOS的定时任务，每隔一段时间执行一次检测。

那么就需要去搜索相关的命令以达到预期目的了。不过笔者在网上搜索相关的脚本命令过程中，有了一些意外领悟，那就是如何将SpringBoot应用的jar包在Linux下服务化。

### “自我实现”的脚本
首先感谢[这篇博文](http://blog.csdn.net/catoop/article/details/50588851)的作者，笔者看过后从中受益匪浅，稍作调整，实现了自己业务需要的脚本内容。

这里笔者只强调脚本实现过程中的关键点，具体实现的结果，可点击[这里](http://git.oschina.net/devchenx/ShellScriptAboutJar)查看。

#### SpringBoot应用监控

- 获取应用进程id的命令

``` Shell
ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'
```

ps命令，查看进程信息的命令，需要从ps执行结果中筛选出目标应用的那行信息，且截取出进程id。

grep $APP_NAME：筛选出包含目标应用名称的信息行。变量$APP_NAME表示要检测的目标应用名称，尽量精确，不然会影响筛选结果。

grep -v grep|grep -v kill：“剔除”grep命令和kill命令的信息行，避免造成干扰。亲身执行过ps...grep...命令的人会清楚，进程信息中除了目标进程，还会显示grep命令的进程。

awk '{print $2}'：截取出进程id的信息。

<!-- more -->

- 判断进程id是否存在

``` Shell
tpid=`ps -ef|grep $app_name|grep -v grep|grep -v kill|awk '{print $2}'`
if [ ! ${tpid} ]
then
  curdatetime=$(date '+%Y-%m-%d %T')
  echo '====== Service Check Time: '${curdatetime} >> $CK_LOG
  echo ${app_name}' is not running.' >> $CK_LOG
  echo 'try to restart '${app_name}' ...' >> $CK_LOG
  # 启动指定的应用
  nohup java $opt_xmx $opt_xms -jar $app_location > /dev/null 2>&1 &
fi
```

- 将shell脚本设置为定时任务

看过完整的执行脚本内容，只要有编辑语言基础，你会很快熟悉shell语法中的控制语句使用方法。OK，整合上述关键的命令内容，写入.sh文件中，保存生成shell脚本文件。记住给脚本文件设置执行权限。

``` Shell
chmod 500 ./test.sh
# 执行脚本试试看
./test.sh
```

手动执行脚本无误后，设置由操作系统定时周期性地执行脚本。执行下面的命令打开定时任务文件。

``` Shell
crontab -e
```

打开定时任务文件后，写入内容保存，表示每隔30分钟，执行/www目录下的test.sh脚本。

``` Shell
*/30 * * * * * /www/test.sh
```

注意确保操作系统中的crond服务是打开的，不然即使你填写了定时任务文件，依然无效，笔者吃过亏~：

``` Shell
service crond status
```

#### SpringBoot应用服务化脚本配置

经历了上述检测脚本的实现过程，笔者学习到了如何判断目标应用运行中的一种方法。举一反三一下，配合一些脚本执行时输入的参数，可以实现应用服务化脚本。

服务化脚本，就是将自己写出的执行脚本，配置成当前操作系统的服务。请类比Windows中的服务。

举个例子，在CentOS中安装Nginx后，你可以通过下面的命令对Nginx进行操作：

``` Shell
# 查看Nginx运行状态
service nginx status
# 重载配置文件内容
service nginx reload
# 启动nginx
service nginx start
# 停止nginx
service nginx stop
```

如果将自己的应用，起个服务名称，替换上面命令中的nginx，那么日常维护运行应用时，会简化许多手动工作量。

其实上面命令中的status|reload|start|stop均是脚本执行时接收的参数，在脚本里根据接收到的参数，执行不同的命令。

具体实现的脚本内容可查看[这里](http://git.oschina.net/devchenx/ShellScriptAboutJar/blob/master/myservice.sh?dir=0&filepath=myservice.sh&oid=a017247d96abcdbd6844b5b13189cd3ea0092495&sha=c58cef921df38b7e61b7c5c9740fdb16893b2843)。假设执行脚本已实现，就叫service.sh吧。

创建软连接到/etc/init.d/目录下，给服务起个容易记住的名字：

``` Shell
ln -s /www/service.sh /etc/init.d/myservice
```

结合脚本中的参数意义，即可执行类似如下的命令来验证结果了。

``` Shell
service myservice <参数>
# 或者
/etc/init.d/myservice <参数>
```

不难看出，命令service可以理解为“/etc/init.d/”的替代。

### 服务脚本开机启动配置

完成执行脚本服务化的配置后，笔者又想将其配置为开机自动启动，这样可以避免意外宕机重启后还需手动启动服务的过程。

CentOS下依靠chkconfig命令，Ubuntu下依靠update-rc.d命令。笔者这里着重讲解chkconfig命令。

首先，确保执行的服务化脚本开头一定要添加如下chkconfig需要的选项说明，告知chkconfig该服务的运行级别和开机顺序，具体可搜索chkconfig命令的说明，一看便知。

``` Shell
#!/bin/sh
# chkconfig: 2345 99 10
# description: myservice
```

然后将配置的服务加入开机启动的行列吧。

``` Shell
chkconfig myservice on
```

其实chkconfig ... on命令是更改服务所在操作系统中运行级别时用的，真正意义上添加开机启动服务的命令如下：

``` Shell
chkconfig --add myservice
```

但经过笔者亲试，如果是第一次添加开机启动服务，chkconfig ... on也会将服务注册为卡机启动。

查看开机启动的服务列表：

``` Shell
chkconfig --list
```

删除开机启动的服务：

``` Shell
chkconfig --del myservice
```

### 环境变量问题

在上述的执行脚本构建过程中，笔者发现一个问题，即手动执行脚本没问题，可是将脚本设置为定时任务执行时或配置为服务脚本时，会发现java命令找不到，导致应用无法启动~~

查询了相关资料，说定时任务执行脚本时是无法识别操作系统中的环境变量的，自然无法识别java命令。所以需要在脚本的开始处添加如下命令，引入操作系统中的环境变量。

``` Shell
source /etc/profile
```

但笔者不明白为何服务化脚本中也识别不了环境变量。笔者做了测试，在自建的虚拟机环境下，即使没有引用操作系统的环境变量，service命令依然可以正常运行，但是在线上服务器下，则需要引入环境变量，service命令才能运行。这点还请了解的朋友帮助解答哈~

### 小结

虽然本文标题中写着SpringBoot应用，看完你会发现，执行脚本适用于任何可执行的jar应用程序，只不过SpringBoot应用最终的产出物也是可执行的jar罢了。

整个过程中，笔者算是又温习了一遍shell语法，不难发现基础语法可以很快上手，并没有想象中的那么难。还有就是熟练掌握Linux命令，可以帮助我们在部署项目时事半功倍。
