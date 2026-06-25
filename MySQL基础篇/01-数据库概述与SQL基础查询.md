---
title: 数据库概述与SQL基础查询
tags:
  - 标签/MySQL
  - 标签/SQL
  - 标签/基础
source: 尚硅谷·宋红康 MySQL基础篇 P01~P24
created: 2025-01-01
---

# 数据库概述与 SQL 基础查询

## 一、数据库基础概念

### 1.1 数据管理演进

文件系统（数据冗余、不一致） → 数据库系统（统一管理、事务保障）

| 维度 | 文件系统 | 数据库系统 |
|---|---|---|
| 数据冗余 | 高 | 低 |
| 一致性 | 差 | 好（事务保障） |
| 并发访问 | 困难 | 内置并发控制 |
| 安全性 | 权限难控 | 多层权限体系 |

### 1.2 常见 DBMS 对比

| DBMS | 定位 | 一句话 |
|---|---|---|
| **MySQL** | 开源免费、轻量高效 | 互联网标配，性价比之王 |
| **Oracle** | 功能最强、商业收费 | 银行级稳定，贵但值 |
| **SQL Server** | 微软生态 | Windows 全家桶一环 |
| **PostgreSQL** | 开源学术范、扩展强 | 学院派首选 |

### 1.3 SQL 分类

| 分类 | 全称 | 作用 | 典型命令 |
|---|---|---|---|
| **DDL** | Data Definition Language | 定义结构 | CREATE / ALTER / DROP |
| **DML** | Data Manipulation Language | 操作数据 | INSERT / UPDATE / DELETE |
| **DQL** | Data Query Language | 查询数据 | SELECT |
| **DCL** | Data Control Language | 权限控制 | GRANT / REVOKE |
| **TCL** | Transaction Control Language | 事务控制 | COMMIT / ROLLBACK |

### 1.4 四种表关系

| 关系 | 现实举例 | 实现方式 |
|---|---|---|
| **一对一** | 用户 ↔ 身份证 | 任意表加唯一外键 |
| **一对多** | 部门 ↔ 员工 | "多"方加外键 |
| **多对多** | 学生 ↔ 课程 | 中间表（两个 1:N） |
| **自引用** | 员工 ↔ 上级 | 表内加外键指向自身 |

### 1.5 MySQL 5.7 vs 8.0

| 特性 | 5.7 | 8.0 |
|---|---|---|
| 窗口函数 | ❌ | ✅ |
| CTE | ❌ | ✅ |
| 默认字符集 | latin1 | utf8mb4 |
| 事务性 DDL | ❌ | ✅ 原子 DDL |
| 角色管理 | ❌ | ✅ |

---

## 二、SQL 基础查询（P12~P24）

### 2.1 SELECT 执行顺序（面试必背）

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
 1       2        3          4        5         6         7
```

### 2.2 基本查询

```sql
-- 查询所有列
SELECT * FROM employees;

-- 查询指定列
SELECT employee_id, first_name, salary FROM employees;

-- 列别名
SELECT name AS 姓名, salary * 12 AS 年薪 FROM employees;

-- 去重
SELECT DISTINCT department_id FROM employees;
```

### 2.3 NULL 的本质

NULL 不是 0，不是空字符串，而是"**不知道/不存在**"。任何值与 NULL 运算结果都是 NULL。

```sql
WHERE commission_pct IS NULL;     -- ✅ 正确
WHERE commission_pct = NULL;      -- ❌ 错误！
```

### 2.4 比较运算符

| 运算符 | 含义 | 示例 |
|---|---|---|
| `=` | 等于 | `WHERE salary = 5000` |
| `<>` / `!=` | 不等于 | `WHERE dept_id <> 10` |
| `BETWEEN...AND` | 闭区间 | `WHERE salary BETWEEN 5000 AND 10000` |
| `IN` | 在集合中 | `WHERE dept_id IN (10,20,30)` |
| `LIKE` | 模糊匹配 | `WHERE name LIKE '张%'` |
| `IS NULL` | 判空 | `WHERE commission IS NULL` |

**LIKE 通配符**：`%` 匹配任意多个字符，`_` 匹配恰好一个字符。

### 2.5 逻辑运算符

| 运算符 | 含义 |
|---|---|
| `AND` | 且（都满足） |
| `OR` | 或（满足一个） |
| `NOT` | 非（排除） |
| `XOR` | 异或（二选一） |

### 2.6 排序与分页

```sql
-- 多列排序
SELECT * FROM employees ORDER BY dept_id ASC, salary DESC;

-- 分页公式：第 page 页，每页 size 条
SELECT * FROM employees LIMIT (page-1)*size, size;
```

> 深分页问题：`LIMIT 100000, 10` 越往后越慢（扫描前 100010 条再丢弃），后续会学到优化方法。

---

## 三、容易出错的地方

1. `= NULL` 是错的，要用 `IS NULL`
2. `SELECT *` 在生产环境尽量避免
3. `LIMIT` 的偏移量从 0 开始，不是 1
4. 字符串比较默认不区分大小写（取决于 Collation）

---

## 四、和其他知识点的联系

| 关联知识 | 关系 |
|---|---|
| 多表查询 JOIN | WHERE 和 ON 的区别 |
| 索引 | WHERE 条件列建索引加速查询 |
| EXPLAIN | 查看 SQL 执行计划 |

---

## 五、面试可能怎么问

1. SQL 的执行顺序是什么？
2. `NULL` 和空字符串有什么区别？
3. `LIMIT` 的深分页问题怎么解决？

---

## AI辅助思考

### 这个知识点真正重要的地方

SELECT 执行顺序是**地基中的地基**，后续的 GROUP BY、HAVING、JOIN 都建立在此之上。NULL 的处理是新手最常踩的坑。

### 下一步可以学什么

1. 多表查询（7 种 JOIN）
2. 函数（单行函数 + 聚合函数）

### AI 给我的学习建议

SQL 基础查询是每天都会用到的技能，建议多写多练。SELECT 执行顺序要牢记——理解它才能理解为什么 WHERE 里不能用列别名、HAVING 里可以用聚合函数。
