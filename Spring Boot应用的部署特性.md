---
title: Spring Boot应用的部署特性
date: 2017-1-18 09:29:40
tags:
- CentOS
- Linux
- Spring Boot
categories:
- 系统运维
---
### 前言
按着笔者前一篇文章[《SpringBoot应用在CentOS下的监控及服务化脚本实现》](https://my.oschina.net/angerbaby/blog/826471)所写，笔者花了一天的时间实现了关于运行监控Spring Boot应用的服务化脚本，并在线上服务器上进行了测试运行。

或许冥冥之中自有天意，隔天笔者发现了一篇[英文文档](http://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html)，详尽地讲述了Spring Boot应用打包后的部署说明。

看完之后，顿时想哭的节奏，原来Spring Boot自带强大特性，可以直接部署为Linux服务~~~

简单点说，就是在使用Spring Boot Maven插件打包应用时，添加些配置，可以在产出的jar包中包含一个执行脚本，这个执行脚本支持jar运行为系统服务。具体说明请参看英文文档，说明很详细！

这里笔者自我安慰下，虽然自己先笨笨地手动实现了一遍服务化脚本，不过通过这个过程，再看Spring Boot这个部署特性时，理解会更透彻，遇见某些问题时会有更快的反应去解决。

### Spring Boot应用强大的部署特性

在介绍Spring Boot强大部署特性之前，请先看下面的一段说明。插件生成的默认脚本仅在CentOS和Ubuntu中测试过呦~并不保证100%的可用性。

> The default script supports most Linux distributions and is tested on CentOS and Ubuntu. Other platforms, such as OS X and FreeBSD, will require the use of a custom embeddedLaunchScript.

- 想要打包出的jar包含执行脚本，在Spring Boot Maven插件中添加如下配置：

``` XML
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

- 按照Spring Boot正常的打包流程，打包应用，生成jar文件。

- 上传jar文件到服务器，直接将jar文件配置为服务执行脚本。

<!-- more -->

``` Shell
# 确保jar文件有执行权限
chmod 500 serviceA.jar
# 创建jar文件到/etc/init.d/的软连接
ln -s /www/serviceA.jar /etc/init.d/myservice
```
**ps**：创建软连接时，一定要指定jar文件的绝对路径。建立软连接后，可以查看/etc/init.d/目录下多了myservice文件，这个文件就是包含在jar中的执行脚本内容。不妨预览一下脚本内容，其中内容还是挺全面的，包括chkconfig命令的选项声明，可以方便地使用chkconfig命令将服务注册为开机启动，无需做任何修改。

- 使用service命令运行应用

``` Shell
service myservice start
```

关于如何将服务设置为开机启动，请参照笔者的[上篇文章](https://my.oschina.net/angerbaby/blog/826471)中的服务开机自启动配置部分的说明。

Spring Boot不仅支持生成默认的执行脚本，还支持配置相关属性，控制生成的脚本中的内容。如果这个默认脚本对你不适用，还可以自行编写脚本，Spring Boot Maven插件支持集成自定义的执行脚本。

通过service命令运行Spring Boot应用，会在/var/log/目录下生成对应的日志文件，会生成对应的pid文件等等。打包出的jar文件不仅包含需要的执行脚本，还精心准备了运行时的相关配置方式，通过service命令运行Spring Boot应用时，可以自定义Java运行时的参数，可以自定义生成日志文件的位置等等。

具体的配置说明，请查看上面提到的英文文档。笔者采用是.conf文件+.jar文件的方式，对运行的服务进行了运行参数的配置。举个例子，conf文件中内容可以是：

``` Shell
JAVA_OPTS="-Xmx512m -Xms512m"
```

### Mybatis-Spring的配置干扰问题

如果你的Spring Boot应用中用到了Mybatis，甚至用到了Mybatis-Spring中间件，那么恭喜你，当使用service命令运行服务时，会报错。这个问题已经有人提出过了，可是并没有得到很好的解决。具体参看[这里](http://www.scienjus.com/mybatis-vfs-bug/)。

这里笔者具体说明下解决办法吧。

1. 去除应用中Maven依赖配置中的Mybatis和Mybatis-Spring。

2. 在Maven依赖配置中添加对mybatis-spring-boot-starter的依赖，版本一定要用最新的，笔者使用的是1.1.1。
``` XML
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>${mybatis.starter.version}</version>
</dependency>
```
3. 在SqlSessionFactory定义处重置VFS，如下所示。
``` Java
@Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource) {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setVfs(SpringBootVFS.class);
        factoryBean.setDataSource(dataSource);
        factoryBean.setTypeAliases(new Class[]{ActionLog.class});

        try {
            return factoryBean.getObject();
        } catch (Exception e) {
            // ......
        }
    }
```

确保按照上述内容调整过后，重新打包，service命令运行起来，OK啦！

### 小结

膜拜过Spring Boot的部署特性后，笔者对比自己之前实现的服务化脚本，总结一下发现的不同点吧。

- 判断应用运行状态的方式不同。

笔者之前实现的方式为，通过ps命令解析出应用对应的进程id，如果这个进程id存在，则表示应用运行正常。

通过查看Spring Boot生成的默认脚本的内容以及jar包的运行结果，会发现应用运行后会生成一个pid文件，里面存放在应用启动时的进程id，通过读取pid文件中的进程id号，判断应用是否正常运行。

其实如果你安装过Linux应用程序的话，会发现生成pid文件是一种主流的做法~

- Spring Boot默认脚本内容更全面，命令更适配，考虑情况更多些。

笔者自我实现的脚本，可能更倾向于笔者当前所有处于的服务器环境配置，脚本命令中缺乏各种情况的考虑，偏小众。

再看Spring Boot默认脚本中内容，你会发现更多的命令处理，虽然有些命令笔者也很陌生，不过大体意思还是看懂的。再次向Spring Boot的广泛参与实现者致敬！

最后，再次献上笔者自己实现的[脚本地址](http://git.oschina.net/devchenx/ShellScriptAboutJar)吧，仅供参考！
