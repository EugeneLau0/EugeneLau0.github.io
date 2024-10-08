---
title: 'Postgres VS MySql内存模型对比，其实它们不一样'
categories:
  - Blog
tags:
  - Database
  - Postgres
  - MySql
---

PostgreSQL和MySQL是两种广泛使用的开源关系型数据库管理系统，它们在内存模型方面有一些关键的差异.

<!--more-->

pg cache 和mysql cache有神马关系，其实它们两者并无直接关系。

本文目的是想澄清我之前对这两个关系的理解误差（OS：我很早之前认为它们是一样的）。

## PG vs mysql memory model

![内存模型对比](../../../assets/images/attachments/20240824_pg_vs_mysql_memory_model.png)

## postgresql os cache

pg是一种强依赖os cache的数据库，以此提高查询效率。那么在优化慢SQL的时候，从开发者的视角就会遇到问题，有了os cache，就不能每次重现慢查询，以至于不知道如何去解决问题。其实从DBA的视角来看，除了RT（response time）之外，还有一项非常重要的指标，那就是cost（执行代价），分析慢SQL可以从这里下手。

![](../../../assets/images/svg/20240824_slow-sql-analysis.svg)

