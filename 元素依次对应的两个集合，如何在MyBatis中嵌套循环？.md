---
title: 元素依次对应的两个集合，如何在MyBatis中嵌套循环？
date: 2020-05-19 14:46:34
tags:
  - MyBatis
categories:
  - 开发笔记
---

### 前言
在MyBatis的xml中需要定义一个方法，入参是两个List，这两个List中的元素要求是按顺序一一对应的。其中一个List的元素读取依赖于另一个List循环时的索引值。

### 假设场景

有一student表，有3个年级（grade），分别是1，2，3，每个年级对应不同的最小标准年龄，分别是 1 -> 10，2 -> 15， 3 -> 20。

现在需要查询3个年级中年龄小于各自最小标准年龄的学生。

### MyBatis中的实现
首先在UserMapper类中定义方法，如下：

```java
List<Student> listLessThanMinAge(@Param("grades") List<Integer> grades, @Param("minAges") List<Integer> minAges);
```

<!-- more -->

然后在UserMapper.xml中定义查询语句如下：

```xml
<select id="listLessThanMinAge" resultMap="BaseResultMap">
        <foreach collection="grades" item="grade" separator=" UNION ALL " index="idx">
            SELECT * 
            FROM `student`
            WHERE `grade` = #{grade}
            AND `age` &lt; #{minAges[${idx}]}
        </foreach>
  </select>
```
最后，调用定义的查询方法，如下：
```java
List<Integer> grades = Arrays.asList(1, 2, 3);
List<Integer> minAges = Arrays.asList(10, 15, 20);
List<Student> students = userMapper.listLessThanMinAge(grades, minAges);
```
