---
title: SpringBoot多数据源及MyBatis配置详解
date: 2016-07-15 16:38:40
tags:
- Spring Boot
- Spring
- MyBatis
categories:
- 开发笔记
---

### 前言

最近迫于项目需要，笔者踏上了springboot多数据源的配置之旅。之前笔者配置过spring的动态多数据源切换，当时使用的是JDBC Template。

目前项目中持久化框架使用是mybatis，经过分析后不难发现，多数据源配置需要解决两个问题，一个是由原先的spring经典方式切换到了springboot方式下，多数据源如何配置？有无太大变化？另一个是怎样将多数据源与mybatis的配置关联起来？

不妨先来看下,单数据源下mybatis如何配置的？

### 单数据源示例

首先要声明一点，项目只是依赖单个数据源时，如果你不介意springboot帮你做事的话，那么恭喜你，你省事儿了！你只需要在项目的属性文件中添加数据源的相关属性配置，springboot会“免费”提供给你一个数据源使用，默认采用的是[tomcat jdbc connection pool](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html)。

当然你可以拒绝springboot的好意，如果你依赖第三方的连接池技术，你可以配置自己的数据源，那么springboot检测到你自己定义了DataSource后，就不会自动配置数据源了。

笔者不能拒绝springboot的好意，所以仅在项目的application.properties中添加了如下属性：

``` Java
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.max-idle=10
spring.datasource.max-wait=10000
spring.datasource.min-idle=5
spring.datasource.initial-size=5
spring.datasource.validation-query=SELECT 1
spring.datasource.test-on-borrow=false
spring.datasource.test-while-idle=true
spring.datasource.time-between-eviction-runs-millis=18800
```

然后笔者创建了一个专门用于配置mybatis的类，如下:

``` Java
@Configuration
public class MybatisSpringConfig {
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setTypeAliasesPackage("demo.model");

        return factoryBean.getObject();
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("demo.repository");

        return mapperScannerConfigurer;
    }
 }    
```

没错，mybatis在spring中就是可以通过如此的简练配置进而正常工作起来。你无需刻意地去创建mybatis的配置文件，无需刻意地去注册mapper接口及指定对应xml文件的位置，这完全得益于[mybatis-spring](http://www.mybatis.org/spring/)，它就像一个“粘合剂”，可以很方便地将mybatis和spring“粘合”在一起。

<!-- more -->

#### MyBatis-Spring的配置步骤

不妨先来说下mybatis-spring配置的一般步骤：

1 配置数据源DataSource的Bean。
2 使用DataSource配置事务管理器。
3 使用DataSource配置SqlSessionFactory的Bean。
4 配置MapperScannerConfigurer的Bean。

这里要求配置事务管理器和SqlSessionFactory的数据源必须是同一个，否则事务管理不起作用。配置MapperScannerConfigurer的目的是自动扫描mapper接口所在的包，自动帮你将mapper接口注册为Bean（代理生成接口的实现类），你就可以直接拿来依赖注入了，建议将mapper接口及其对应的xml文件放在同一个包下，这样的话你无需在SqlSessionFactory里指定xml文件的位置了。

OK，到此对比我上面贴出的配置类内容，你可能会发现笔者怎么少了几步？感谢springboot，因为它自动配置了一个DataSource，同时它还自动配置了一个事务管理器。所以笔者只配置了SqlSessionFactory和MapperScannerConfigurer。

当然如果看到这里你仍然“执意”要配置自己的数据源，参照下面的多数据源配置说明，抽出来多个中的一个就可以实现自定义单数据源的配置了。

### 多数据源示例

经过上面单数据源的示例，可以说当我们切换到springboot的方式下写代码时，springboot为我们带来了很大的便利，还不影响我们自定义，所以笔者认为，没用使用springboot之前，无论你使用spring怎样的配置，使用springboot之后，不会有阻碍，甚至会比原来更快！

简单说下需要多数据源的场景，笔者参照了一下其他的文章，绝大部分的需要来自于数据库主从方式或读写分离。那么就按照master和slave两个数据源，直接贴出数据源的配置类。

- application.properties

``` Java
datasource.master.url=jdbc:mysql://localhost:3306/master
datasource.master.username=root
datasource.master.password=root
datasource.master.driver-class-name=com.mysql.jdbc.Driver
datasource.master.max-idle=10
datasource.master.max-wait=10000
datasource.master.min-idle=5
datasource.master.initial-size=5
datasource.master.validation-query=SELECT 1
datasource.master.test-on-borrow=false
datasource.master.test-while-idle=true
datasource.master.time-between-eviction-runs-millis=18800

datasource.slave.url=jdbc:mysql://localhost:3306/slave
datasource.slave.username=root
datasource.slave.password=root
datasource.slave.driver-class-name=com.mysql.jdbc.Driver
datasource.slave.max-idle=10
datasource.slave.max-wait=10000
datasource.slave.min-idle=5
datasource.slave.initial-size=5
datasource.slave.validation-query=SELECT 1
datasource.slave.test-on-borrow=false
datasource.slave.test-while-idle=true
datasource.slave.time-between-eviction-runs-millis=18800
```

- master数据源

``` Java
@Configuration
public class MasterConfig {
    @Primary
    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "datasource.master")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "masterTransactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("masterDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
 }
```

- slave数据源

``` Java
@Configuration
public class SlaveConfig {
    @Bean(name = "slaveDataSource")
    @ConfigurationProperties(prefix = "datasource.slave")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "slaveTransactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("slaveDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
 }
```

不难看出两个数据源的配置步骤吧：

1 在属性文件中配置两个数据源需要用到的属性值，注意起个好点的前缀名称。
2 构建两个数据源的配置类，当然这不是必须的，愿意堆在一个配置类中未尝不可。
3 配置类中，配置DataSource的Bean，记得起个能够标识的name！
4 配置两个数据源各自对应的事务管理器，别嫌麻烦，否则会给自己埋坑里，记得起个能够标识的name！

配置DataSource时，利用@ConfigurationProperties(prefix = "xxx.xxx")可以依靠指定的前缀，在诸多的属性值中“挑选”出数据源依赖的属性，进而完成数据源的构建。

当自己定义了DataSource后，springboot就会取消自动配置的动作了。为了各司其职，为每个数据源配置各自的事务管理器，springboot自然也会取消自动配置事务管理器的动作。由于是多个数据源和多个事务管理器，都是一个类型的，你要是不起个区别的名字，任谁都分辨不出来吧？

*@Primary* 有什么作用呢？简单地说，当有两个同一类型的Bean，依赖注入时你没有指定name，正常情况下会报错，有两个你要的Bean，识别不了。但是 *@Primary* 相当于指定这个Bean为默认的，如果你没有指定name，就采用 *@Primary* 标识的Bean。

OK，两个数据源的配置配好了，还需要配置各自的Mybatis来进行持久化的操作。

#### MyBatis-Spring相关配置

- mybatis for master

``` Java
@Configuration
@MapperScan(basePackages = {"demo.repository.master"}, sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterConfig {
    @Primary
    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "datasource.master")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "masterTransactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("masterDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean(name = "masterSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setTypeAliasesPackage("demo.model");

        return factoryBean.getObject();
    }
 }
```

- mybatis for slave

``` Java
@Configuration
@MapperScan(basePackages = {"demo.repository.slave"}, sqlSessionFactoryRef = "slaveSqlSessionFactory")
public class SlaveConfig {
    @Bean(name = "slaveDataSource")
    @ConfigurationProperties(prefix = "datasource.slave")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "slaveTransactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("slaveDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "slaveSqlSessionFactory")
    public SqlSessionFactory basicSqlSessionFactory(@Qualifier("slaveDataSource") DataSource basicDataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(basicDataSource);
        factoryBean.setTypeAliasesPackage("demo.model");

        return factoryBean.getObject();
    }
 }
```

这里需要强调几个地方：

1 细心人会发现上面的配置类中，少了MapperScannerConfigurer的Bean配置，改用了@MapperScan注解。其实两者的作用是一样的，但是@MapperScan比较新，稍后会做解释为什么它比较新。

2 由于两个数据源的原因，引出了两套SqlSessionFactory的配置，所以@MapperScan中需要指明依赖的是哪个SqlSessionFactory，“sqlSessionFactoryRef”对应就是SqlSessionFactory的name属性。

3 @MapperScan会将扫描的mapper接口代理生成实现类，并自动注册为Bean。由于两个数据源的配置类中都有@MapperScan注解，为了避免造成冲突和排错时的困扰，猛烈提醒，两个数据源的配置，mybatis对应的mapper接口及对应xml文件也构建两套，最好接口名上也做些区分。model类使用同一套倒是没什么影响。所以你会看到上面的配置中，@MapperScan中basePackages指向的是两个包路径。

好了，来解释下@MapperScan为何比较新，并且笔者推荐使用@MapperScan。

首先@MapperScan要求的mybatis-spring版本比较新，说明它是新推出的特性。

其次@MapperScan要比配置MapperScannerConfigurer的Bean要简练的多，代码量上就看得出来。

最后，@MapperScan中的basePackageClasses属性是MapperScannerConfigurer所没有的。并且笔者用到了这个basePackageClasses属性，所以这里强力推荐使用@MapperScan注解。

多聊一些，描述下笔者为何会用到@MapperScan中的basePackageClasses属性吧，况且与上述示例中的basePackages有何区别呢？

上面提到了多数据源的一般场景，笔者的不同。笔者的项目中划分了n个子系统，每个子系统有各自的数据库，现在需要每个子系统共享一个公共信息的数据库。

从代码上来说，

### 总结
