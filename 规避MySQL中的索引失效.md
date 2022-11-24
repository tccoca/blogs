---
title: 规避MySQL中的索引失效
date: 2020-05-19 14:42:29
tags:
  - MySQL
categories:
  - 开发笔记
---

# 前言

之前有看过许多类似的文章内容，提到过一些sql语句的使用不当会导致MySQL的索引失效。还有一些MySQL“军规”或者规范写明了某些sql不能这么写，否则索引失效。

绝大部分的内容笔者是认可的，不过部分举例中笔者认为用词太绝对了，并没有说明其中的原由，很多人不知道为什么。所以笔者绝对再整理一遍MySQL中索引失效的常见场景，并分析其中的原由供大家参考。

当然请记住，**explain是一个好习惯！**

# MySQL索引失效的常见场景

在验证下面的场景时，请准备足够多的数据量，因为数据量少时，MySQL的优化器有时会判定全表扫描无伤大雅，就不会命中索引了。

## 1. where语句中包含or时，可能会导致索引失效

使用or并不是一定会使索引失效，你需要看or左右两边的查询列是否命中相同的索引。

假设USER表中的user_id列有索引，age列没有索引。

下面这条语句其实是命中索引的（据说是新版本的MySQL才可以，如果你使用的是老版本的MySQL，可以使用explain验证下）。

<!-- more -->

```
select * from `user` where user_id = 1 or user_id = 2;
```

但是这条语句是无法命中索引的。

```
select * from `user` where user_id = 1 or age = 20;
```

假设age列也有索引的话，依然是无法命中索引的。

```
select * from `user` where user_id = 1 or age = 20;
```

因此才有建议说，尽量避免使用or语句，可以根据情况尽量使用union all或者in来代替，这两个语句的执行效率也比or好些。

## 2. where语句中索引列使用了负向查询，可能会导致索引失效

负向查询包括：NOT、!=、<>、!<、!>、NOT IN、NOT LIKE等。

某“军规”中说，使用负向查询一定会索引失效，笔者查了些文章，有网友对这点进行了反驳并举证。

其实负向查询并不绝对会索引失效，这要看MySQL优化器的判断，全表扫描或者走索引哪个成本低了。

## 3. 索引字段可以为null，使用is null或is not null时，可能会导致索引失效

其实单个索引字段，使用is null或is not null时，是可以命中索引的，但网友在举证时说两个不同索引字段用or连接时，索引就失效了，笔者认为确实索引失效，但这个锅应该由or来背，属于第一种场景~~

假设USER表中的user_id列有索引且允许null，age列有索引且允许null。

```
select * from `user` where user_id is not null or age is not null;
```

不过某些“军规”和规范中都有强调，字段要设为not null并提供默认值，是有原因值得参考的。

> - null的列使索引/索引统计/值比较都更加复杂，对MySQL来说更难优化。
> - null 这种类型MySQL内部需要进行特殊处理，增加数据库处理记录的复杂性；同等条件下，表中有较多空字段的时候，数据库的处理性能会降低很多。
> - null值需要更多的存储空，无论是表还是索引中每行中的null的列都需要额外的空间来标识。
> - 对null 的处理时候，只能采用is null或is not null，而不能采用=、in、<、<>、!=、not in这些操作符号。如：where name!=’shenjian’，如果存在name为null值的记录，查询结果就不会包含name为null值的记录。

## 4. 在索引列上使用内置函数，一定会导致索引失效

比如下面语句中索引列login_time上使用了函数，会索引失效：

```
select * from `user` where DATE_ADD(login_time, INTERVAL 1 DAY) = 7;
```

优化建议，尽量在应用程序中进行计算和转换。


其实还有网友提到的两种索引失效场景，应该都归于索引列使用了函数。

### 4.1 隐式类型转换导致的索引失效

比如下面语句中索引列user_id为varchar类型，不会命中索引：

```
select * from `user` where user_id = 12;
```

这是因为MySQL做了隐式类型转换，调用函数将user_id做了转换。

```
select * from `user` where CAST(user_id AS signed int) = 12;
```

### 4.2 隐式字符编码转换导致的索引失效

当两个表之间做关联查询时，如果两个表中关联的字段字符编码不一致的话，MySQL可能会调用CONVERT函数，将不同的字符编码进行隐式转换从而达到统一。作用到关联的字段时，就会导致索引失效。

比如下面这个语句，其中d.tradeid字符编码为utf8，而l.tradeid的字符编码为utf8mb4。因为utf8mb4是utf8的超集，所以MySQL在做转换时会用CONVERT将utf8转为utf8mb4。简单来看就是CONVERT作用到了d.tradeid上，因此索引失效。

```
select l.operator from tradelog l , trade_detail d where d.tradeid=l.tradeid and d.id=4;
```

这种情况一般有两种解决方案。

**方案1:** 将关联字段的字符编码统一。

**方案2:** 实在无法统一字符编码时，手动将CONVERT函数作用到关联时=的右侧，起到字符编码统一的目的，这里是强制将utf8mb4转为utf8，当然从超集向子集转换是有数据截断风险的。如下：

```
select d.* from tradelog l , trade_detail d where d.tradeid=CONVERT(l.tradeid USING utf8) and l.id=2; 
```

## 5. 对索引列进行运算，一定会导致索引失效

运算如+，-，*，/等，如下：

```
select * from `user` where age - 1 = 10;
```

优化的话，要把运算放在值上，或者在应用程序中直接算好，比如：

```
select * from `user` where age = 10 - 1;
```

## 6. like通配符可能会导致索引失效

like查询以%开头时，会导致索引失效。解决办法有两种：

- 将%移到后面，如：

```
select * from `user` where `name` like '李%';
```

- 利用覆盖索引来命中索引。

```
select name from `user` where `name` like '%李%';
```

## 7. 联合索引中，where中索引列违背最左匹配原则，一定会导致索引失效

当创建一个联合索引的时候，如(k1,k2,k3)，相当于创建了(k1)、(k1,k2)和(k1,k2,k3)三个索引，这就是最左匹配原则。

比如下面的语句就不会命中索引：

```
select * from t where k2=2;
select * from t where k3=3;
slect * from t where k2=2 and k3=3;
```

下面的语句只会命中索引(k1):

```
slect * from t where k1=1 and k3=3;
```

## 8. MySQL优化器的最终选择，不走索引

上面有提到，即使完全符合索引生效的场景，考虑到实际数据量等原因，最终是否使用索引还要看MySQL优化器的判断。当然你也可以在sql语句中写明强制走某个索引。

# 优化索引的一些建议

1. 禁止在更新十分频繁、区分度不高的属性上建立索引。
  - 更新会变更B+树，更新频繁的字段建立索引会大大降低数据库性能。
  - “性别”这种区分度不大的属性，建立索引是没有什么意义的，不能有效过滤数据，性能与全表扫描类似。
2. 建立组合索引，必须把区分度高的字段放在前面。

# 参考

[《为什么这些SQL语句逻辑相同，性能却差异巨大？》](https://time.geekbang.org/column/article/74059)

[《后端程序员必备：索引失效的十大杂症》](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650125497&idx=3&sn=605d9a64e06d39542aeeb95fb8810a80&chksm=f36ba998c41c208e1b32b93cda9593742da2979364d45f61eaac91d435c1a8235a4c07a94943&mpshare=1&scene=1&srcid=&sharer_sharetime=1576197780287&sharer_shareid=62fea8ec9817da4567d1a2aa6f47960a&key=926dc3f74d7bfd3c8d76c512888247ab1136758af01ed47267fc6e06c0a88e34c5c10143f09b7966be80a245cdc8ae0bbe06277c6be24831128bbf3e433f2df9f94a83737e8d82c8cef5f812ec6d61d8&ascene=1&uin=MTYwNTU1NjcwMQ%3D%3D&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=AbIuqKsj1a2HQDdaKjOLS3o%3D&pass_ticket=0pYb9fePtEY5XrZZcts%2Fq81IKEBRl9PMhDBd%2F808ZY41oLYgHlJD2%2FmD8Ej%2Ba6lm)

[《58到家数据库30条军规解读》](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959906&idx=1&sn=2cbdc66cfb5b53cf4327a1e0d18d9b4a&chksm=bd2d07be8a5a8ea86dc3c04eced3f411ee5ec207f73d317245e1fefea1628feb037ad71531bc&scene=21#wechat_redirect)

[《MySQL的or/in/union与索引优化 | 架构师之路》](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651960277&idx=1&sn=bc452fbe863fd915c08f95a680e4bdbe&chksm=bd2d06098a5a8f1fa0262290a65b6a6634f84d394b072f701ed44a6df9dd4b9df8c926537a59&scene=21#wechat_redirect)