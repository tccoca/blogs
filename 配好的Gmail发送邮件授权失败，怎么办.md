---
title: 配好的Gmail发送邮件认证失败，怎么办
date: 2020-06-01 16:57:55
tags:
  - Spring Boot
  - Java
categories:
  - 开发笔记
---

## 前言

Gmail对企业邮箱升级了安全限制，企业邮箱不能再用 账号+密码 的简单方式在应用中发送邮件了，需要采用OAuth2.0的授权方式，因此我们改用了AWS的邮件服务来发送邮件。

鉴于系统中会发一些预警邮件，我们决定改用一个个人Gmail邮箱来专门发送预警邮件。目前个人Gmail邮箱是没有升级安全限制的。

<!--more-->

用到的关键依赖如下：

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>2.1.7.RELEASE</version>
</dependency>
```

## 邮件发送，认证失败

因为之前在应用中用企业邮箱发送邮件，用的是 账号+密码 的简单方式。所以想着简单更换邮箱账号和密码即可。当然采用这种方式，如何保护邮箱密码这是一个需要解决的问题，这里不做讨论。

邮箱配置示例如下：

```properties
alert.mail.host=smtp.gmail.com
alert.mail.username=alert@gmail.com
alert.mail.password=xxx
alert.mail.port=587
alert.mail.smtp.auth=true
alert.mail.smtp.starttls.enable=true
alert.mail.smtp.timeout=300000
```

邮件发送代码示例如下：

```java
JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();
javaMailSender.setHost(host);
javaMailSender.setUsername(from);
javaMailSender.setPassword(password);
javaMailSender.setPort(Integer.parseInt(port));
javaMailSender.setDefaultEncoding("UTF-8");
Properties properties = new Properties();
properties.setProperty("mail.smtp.auth", auth);
properties.setProperty("mail.smtp.starttls.enable", startTlsEnable);
properties.setProperty("mail.smtp.timeout", timeout);
javaMailSender.setJavaMailProperties(properties);

MimeMessage mimeMessage = javaMailSender.createMimeMessage();
MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, "UTF-8");
messageHelper.setFrom(from, "Rebot-alert");
messageHelper.setTo(toList);
messageHelper.setSubject(subject);
messageHelper.setText(body, true);

javaMailSender.send(mimeMessage);
```

测试了一把，提示认证失败！只是更换了邮箱账号和密码，没想到得到这么一个“惊喜”。。。

## 解决方案

OK，遇见问题，解决问题~

首先确认邮箱账号和密码没有问题。笔者在浏览器中重新登录该Gmail账号，确认密码是正确的。

然后网上搜了一些类似情况，很多人提到，在应用中使用Gmail账号+密码的方式发送邮件，需要在该Gmail账号的安全设置中，将“Less secure app access”这一项开启。笔者确认使用的Gmail账号已经开启了这项不建议开启的配置。可是认证依然失败~

接着搜，笔者发现了一个“神奇”的地址，https://accounts.google.com/DisplayUnlockCaptcha

浏览器中登录Gmail账号后，打开上面的地址，然后点“继续”，会提示“Allow access to your Google Account”。紧接着马上试一把，邮件发送成功了！

[这里](https://support.google.com/mail/thread/9828582?hl=en)有对这种认证失败的情况提供了解决步骤。

## 总结

其实Google为了安全考虑，是不建议直接在应用中配置账号+密码来发送邮件的，甚至对企业邮箱做了强制要求。

Google建议对账号开启两步验证，同时设置一个应用专用密码（可以理解为163邮箱的授权码），在应用中用账号+应用专用密码的方式来发送邮件。

那么当邮箱配置都没有问题时，依然认证失败，就可以试试上面的“神奇”地址。笔者在Google帮助中，并没有发现很明显的关于这个地址的解释及应用场景。

不过从字面意思来看，“DisplayUnlockCaptcha”，显示解锁的验证码（翻译有些生硬哈~），它有点类似这样的过程，当你需要将账号用于某个新设备或新应用时，要先填个验证码，确认你要这么做。

但其实当你在浏览器访问这个地址时，并没有填写验证码的内容，只是提示“Allow access to your Google Account”，然后你点击“继续”就好了。

