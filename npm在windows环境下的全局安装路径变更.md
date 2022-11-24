---
title: npm在windows环境下的全局安装路径变更
date: 2018-01-18 20:29:40
tags:
- npm
- nodejs
- js
categories:
- 开发笔记
---
### 前言

笔者用的是windows操作系统，初次安装完node js的环境后，使用npm对几个前端项目依赖的包进行了安装，由于笔者C盘的空间本来不是很大，过了一段时间发现C盘空间在逐渐变小，想到npm全局安装的包都下载到了C盘中，所以笔者想是否可以变更npm全局安装的位置，答案肯定是可以的。

### npm配置文件的位置

node js安装完成后，默认会在环境变量Path加上node js的安装目录。所以在命令行下可以直接使用npm命令。为了在windows环境下应用Linux命令，笔者用的是Git Bash。

启动Git Bash，查看npm配置的信息：

```
npm config list
```

命令执行后的输出信息中，可以看到“; userconfig C:\User\\<计算机名>\\.npmrc”这行，这个位置的文件就是用户配置npm的地方。

### 变更npm全局安装路径

找到.npmrc文件，编辑其中的内容，配置你所期望的npm全局安装位置，如下所示。

```
prefix=D:\npm_repository\npm_modules
cache=D:\npm_repository\npm_cache
```

当使用“npm install -g <包名>”时，则不会再下载到C盘了。

### 使用淘宝镜像

因为npm官方仓库在国外，有时候下载速度会非常慢，不过有淘宝镜像可以使用，下载包的速度很快。而且淘宝镜像是定时更新同步npm的官方仓库的。

习惯性地使用cnpm命令行工具代替默认的npm。使用如下命令按照cnpm工具：

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

cnpm在执行安装包的命令时，会先从淘宝镜像去下载包。但是当你执行上述命令时，会遇到cnpm命令不识别的问题。因为我们自定义了npm的全局安装位置，所以安装后的cnpm命令执行文件在自定义的prefix下。解决办法很简单，在环境变量Path，添加prefix配置的目录路径即可。笔者配置的是“D:\npm_repository\npm_modules”。

环境变量配置完毕后，就可以在命令行中任意目录下随心所欲地使用cnpm命令了。当然，之后凡是全局安装的工具命令，均可以在命令行中任意目录执行。

