---
title: MySQL基础篇学习笔记索引
tags:
  - 标签/MySQL
  - 标签/学习笔记
source: 尚硅谷·宋红康《MySQL数据库入门到大牛》P1~P95
created: 2025-01-01
---

# MySQL 基础篇学习笔记

> 来源：尚硅谷·宋红康《MySQL数据库入门到大牛》| B站 [BV1iq4y1u7vj](https://www.bilibili.com/video/BV1iq4y1u7vj/)
> 基础篇集数：P1 ~ P95（共 95 集，约 45~48 小时）

## 学习路线概览

```
第1阶段 P01~P24   数据库概念 + SQL基础查询（SELECT/WHERE/排序/分页）
第2阶段 P25~P31   多表查询（7种JOIN）🔥 面试核心
第3阶段 P32~P48   函数（聚合函数）+ 子查询 🔥
第4阶段 P49~P65   DDL+DML（建表/增删改）+ 数据类型
第5阶段 P66~P82   6大约束 + 视图 + 存储过程
第6阶段 P83~P93   变量 + 流程控制 + 游标 + 触发器
第7阶段 P94~P95   窗口函数 + CTE 🔥 MySQL 8 重要新特性
```

---

## 笔记文件目录

| 文件 | 集数 | 核心内容 |
|---|---|---|
| [[01-数据库概述与SQL基础查询]] | P01~P24 | DB 基础概念、SQL 分类、SELECT 执行顺序、WHERE/排序/分页 |
| [[02-多表查询与JOIN]] | P25~P31 | **7 种 JOIN**、自连接、非等值连接、SQL92 vs SQL99 |
| [[03-函数与子查询]] | P32~P48 | 数值/字符串/日期函数、**5 大聚合函数**、GROUP BY、子查询 |
| [[04-表的管理与增删改操作]] | P49~P65 | DDL 建表改表、DML 增删改、**数据类型选择** |
| [[05-约束视图与存储过程]] | P66~P82 | **6 大约束**、视图、存储过程与函数 |
| [[06-变量流程控制与触发器]] | P83~P93 | 系统/用户/局部变量、IF/CASE/循环、触发器 |
| [[07-MySQL8新特性-窗口函数与CTE]] | P94~P95 | **窗口函数**（ROW_NUMBER/RANK/LAG）、**CTE**（WITH 递归） |

---

## 面试高频知识点速查

### ⭐ 必会

| 知识点 | 所在文件 | 一句话记忆 |
|---|---|---|
| SELECT 执行顺序 | [[01-数据库概述与SQL基础查询]] | FROM→WHERE→GROUP→HAVING→SELECT→ORDER→LIMIT |
| 7 种 JOIN | [[02-多表查询与JOIN]] | 内连接+左/右外+左/右独有+全外+全外去交集 |
| 5 大聚合函数 | [[03-函数与子查询]] | COUNT/SUM/AVG/MAX/MIN |
| GROUP BY + HAVING | [[03-函数与子查询]] | WHERE 过滤行，HAVING 过滤组 |
| DROP vs DELETE vs TRUNCATE | [[04-表的管理与增删改操作]] | DROP 删结构，TRUNCATE 清数据，DELETE 可条件删 |
| 数据类型选择 | [[04-表的管理与增删改操作]] | 金额用 DECIMAL，字符串用 VARCHAR |
| 6 大约束 | [[05-约束视图与存储过程]] | NOT NULL/UNIQUE/PK/FK/CHECK/DEFAULT |
| 窗口函数 | [[07-MySQL8新特性-窗口函数与CTE]] | ROW_NUMBER/RANK/DENSE_RANK |

### ⭐ 高频

| 知识点 | 所在文件 |
|---|---|
| NULL 的本质 | [[01-数据库概述与SQL基础查询]] |
| WHERE vs HAVING | [[03-函数与子查询]] |
| IN vs EXISTS | [[03-函数与子查询]] |
| VARCHAR vs CHAR | [[04-表的管理与增删改操作]] |
| DATETIME vs TIMESTAMP | [[04-表的管理与增删改操作]] |
| 视图的作用 | [[05-约束视图与存储过程]] |
| 存储过程（互联网禁用） | [[05-约束视图与存储过程]] |
| CTE 与递归查询 | [[07-MySQL8新特性-窗口函数与CTE]] |

---

## 相关笔记

- [[../数据库基础知识总结]] —— 数据库通用理论（范式、ER 图、主外键）
- [[../MySQL高级篇/README]] —— 高级篇索引（索引、事务、锁、MVCC）
