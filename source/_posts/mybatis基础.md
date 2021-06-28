---
title: mybatis基础
categories: 后端
tags: mybatis
translate_title: mybatis-basics
date: 2020-02-21 19:45:55
---

# mybatis基础

## ORM

对象关系映射（Object Relational Mapping，简称ORM）是通过使用描述对象和数据库之间映射的元数据，面向对象语言程序中的对象自动持久化到关系数据库中。本质上就是将数据从一种形式转换到另外一种形式。

> 对象指的是变相对象
>
> 关系是指关系型数据库
>
> Java到MySQL的映射，开发者可以以面向对象的思想来管理数据库

## MyBatis的优点

> - 对JDBC的封装，与JDBC相比减少了一定的代码量。
> - MyBatis是最简单的持久化框架，小巧简单易学。
> - MyBatis相对灵活，不会对应用程序或者数据库的现有设计强加任何影响。SQL语句写在xml文件中，从程序中彻底的分离，降低耦合度，便于统一管理，提高可重用性。
> - 提供xml标签，支持动态sql语句编写
> - 提供映射标签，支持对象与数据库的ORM字段关系映射

## MyBatis的缺点

> - 每一个sql都需要开发者编写，不像其他hibernate可以自动生成一些。sql语句的编写工作量大，尤其是字段多，关联表多时，对开发人员编写sql语句的功底有一定的要求。
> - sql语句依赖于数据库，导致数据库的移植性差，不能随意更改数据库。更换数据库就需要修改sql语句（mysql更换为Oracle）。

## MyBatis的开发方式

### 1、使用原生接口

### 2、Mapper代理实现自定义接口

