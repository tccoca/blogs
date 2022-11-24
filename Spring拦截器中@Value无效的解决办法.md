---
title: Spring拦截器中@Value无效的解决办法
date: 2016-05-11 23:08:49
tags:
- Spring
- Spring Boot
categories:
- 开发笔记
---
最近在使用SpringBoot开发项目时，用到了SpringMVC拦截器的功能。鉴于SpringBoot指南中建议使用Java Config的配置方式，拦截器的配置也不例外，从原先的xml配置方式转为了Java Config。

首先贴出拦截器的一种配置方式：

``` Java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
  @Override
    public void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);

        registry.addInterceptor(new LogInterceptor()).addPathPatterns("/**");
    }

}
```

上面的拦截器配置在网上搜索一下，也是随处可见的。拦截器可以正常运作，但是拦截器中@Value注解的属性值为null，没有读取到期望的properties文件中的值。

<!-- more -->

再看拦截器的另一种配置方式：

``` Java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
  @Bean
  public LogInterceptor logInterceptor() {
    return new LogInterceptor();
  }

  @Override
    public void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);

        registry.addInterceptor(logInterceptor()).addPathPatterns("/**");
    }

}
```

如果按照上面的配置方式，@Value注解可以成功注入properties文件中的属性值。

思考：第一种方式中，拦截器是手动new出来的，拦截器中的依赖注入并未得到处理；第二种方式，同样是new出来的拦截器，但通过@Bean的声明，表明拦截器是Spring管理的bean，依赖注入工作自然Spring会做处理。
