---
title: CentOS部署应用常用命令汇总
date: 2016-06-08 17:05:40
tags: [CentOS,Linux]
categories: [系统运维]
---

### 前言

最近项目在不断地更新迭代，项目部署也由笔者来做了。由于笔者不是专业的Linux服务器运维人员，所以好多用到的命令都是现查现用。这里汇总一下笔者目前自己用的命令，方便今后查阅。

当然随着部署工作的不断进行，笔者也会陆续更新学习到的内容。

### 常用命令

#### 删除目录或文件

``` Shell
rm -rf <目录或文件>
```

选项f表示强制删除，注意使用。

#### 解压缩zip压缩包

``` Shell
unzip <zip文件位置>
```

默认解压到zip所在位置。笔者上传部署项目至服务器上时，习惯打包为zip。

#### 重命名目录或文件名

``` Shell
cd <目录或文件位置>
mv <目录或文件> ./<新名称>
```

重命名使用的是mv命令，mv代表移动目录或文件。

#### 查看指定名称的程序进程

``` Shell
ps aux | grep <模糊名称>
```

#### 查看指定端口号的占用情况

``` Shell
netstat -tlnp | grep <端口号>
```

<!-- more -->

#### 可运行jar包的执行

``` Shell
nohup java -jar <jar包位置> >/dev/null &
```

>/dev/null表示运行jar时控制台输出重定向到null中，就是控制台不显示信息。

&表示程序作为后台进程运行，这样即使关闭控制终端，程序依然在运行不会终止。

运行jar时可能需要配置内存，运行如下命令：

``` Shell
nohup java -Xmx1024m -Xms1024m -jar <jar包位置> >/dev/null &
```
#### 查看程序内存占用的排行信息

``` Shell
ps -eo rss,pmem,pcpu,vsize,args |  sort -k 1 -r -n | less
```

- rss: resident set size,表示进程占用RAM(内存)的大小，单位是KB
- pmem: %M, 占用内存的百分比
- pcpu: %C，占用cpu的百分比
- vsize: 表示进程占用的虚拟内存的大小，KB
- args: 进程名（command）

sort命令对ps结果进行排序

-k 1: 按第一个参数 rss进行排序

-r: 逆序

-n: numeric，按数字来排序
