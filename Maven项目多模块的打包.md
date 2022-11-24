---
title: Maven项目多模块的打包
date: 2016-05-13 23:08:49
tags:
- Maven
- Spring Boot
categories:
- 开发笔记
---
### 前言

最近使用SpringBoot开发项目，同时使用的构建工具是Maven。首次应用Maven的多模块(Modules)方式进行的项目构建，原因在于项目分为多个子模块进行阶段性开发，而其中多个子模块中存在重复的代码段，为了避免重复性地copy代码，决定将重复的代码进行抽离，放在一个公共的module里，其他module对这个公共module进行依赖。

### 说明

- 项目：P
- Module A：后面简称A
- Module B：后面简称B
- Module Common：后面简称C

A依赖C，B也依赖C，A和B可以分别打包成可执行的jar文件。

### Modules的打包

目前已开发完毕A的功能，需要将A打包并进行部署。当使用Maven对A直接进行package时，提示依赖于C的jar包找不到。因为直接对A打包，C并没有被打包，依赖关系无法保持。

但是如果直接对P进行package，P的pom中定义了所有的modules,会对执行所有的modules执行打包，在加上A的pom中声明对C的依赖，可以成功生成可执行的A的jar包。

- P的pom.xml

``` xml
<modules>
    <module>C</module>
    <module>A</module>
    <module>B</module>
</modules>
```

- A的pom.xml

``` xml
<dependencies>
  <dependency>
      <groupId>xxxxx</groupId>
      <artifactId>C</artifactId>
      <version>1.0</version>
  </dependency>
</dependencies>
```

**PS：**这里注意C打包出来的应该是不可执行的jar包，所以不要在C的pom中定义spring-boot-maven-plugin插件，因为这个SpringBoot插件会在Maven的package后进行二次打包，目的为了生成可执行jar包，如果C中定义了这个插件，会报错提示没有找到main函数。

<!-- more -->

### 按需打包

上述的打包方式，不太优雅。笔者只需要对A进行打包，因为A依赖C，进而需要对C进行打包。但是如果对P进行打包的话，B会“无辜”地被牵连到，B也会被打包。而且如果一个项目中的modules很多的话，每个module打包前还需要执行编译、测试等生命周期，未必高效。

那么笔者现在需要的是只对A和C进行打包，如何做到呢？执行如下命令：

``` Java
mvn -pl A -am install
```

上述命令的意思是指定构建Module A，同时依据依赖树的路径，构建A的依赖（无论是直接的还是间接的）。注意这里的命令是install，而不是package。

具体命令选项的含义可查看[这里](http://blog.sonatype.com/2009/10/maven-tips-and-tricks-advanced-reactor-options/)。
