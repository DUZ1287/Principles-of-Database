---
title: 索引创建优化与SQL性能分析
tags:
  - 标签/MySQL
  - 标签/索引优化
  - 标签/EXPLAIN
  - 标签/SQL优化
  - 标签/面试高频
source: 尚硅谷·宋红康 MySQL高级篇 P128~P149
created: 2025-01-01
---

# 索引创建优化与SQL性能分析

## 一、这个知识点是什么

本模块是 MySQL **实战优化**的核心内容，涵盖两大主题：
1. **索引的分类、创建与优化**——什么时候该建索引、什么时候不该建
2. **SQL 性能分析工具**——EXPLAIN、慢查询日志、optimizer_trace

## 二、为什么要学它

- EXPLAIN 是 SQL 优化的**核心工具**，几乎每个优化问题都要先 EXPLAIN
- 索引失效的 11 种情况是**面试常考**
- 覆盖索引、索引下推是实际项目中常用的优化手段

---

## 三、核心内容

### 3.1 索引分类（P128）

**按数据结构分类**：

| 索引类型 | 数据结构 | 说明 |
|---|---|---|
| B+ 树索引 | B+ 树 | 默认、最常用 |
| Hash 索引 | 哈希表 | MEMORY 引擎 |
| Full-text 索引 | 倒排索引 | 全文检索 |

**按功能分类**：

| 索引类型 | 说明 | 是否唯一 |
|---|---|---|
| **主键索引** | 主键上的索引 | 唯一 |
| **唯一索引** | UNIQUE 约束的列 | 唯一 |
| **普通索引** | 普通列上的索引 | 不唯一 |
| **联合索引** | 多列组合索引 | 视情况 |

### 3.2 索引创建方式（P129~P130）

```sql
-- 方式1：CREATE INDEX（推荐）
CREATE INDEX idx_name ON employees(salary);
CREATE UNIQUE INDEX idx_email ON employees(email);

-- 方式2：ALTER TABLE
ALTER TABLE employees ADD INDEX idx_name(salary);

-- 方式3：建表时直接定义
CREATE TABLE employees (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  INDEX idx_salary (salary)
);

-- 删除索引
DROP INDEX idx_name ON employees;
```

**MySQL 8.0 索引新特性**：

```sql
-- 降序索引（8.0 真正支持）
CREATE INDEX idx_dept_sal ON employees(dept_id ASC, salary DESC);

-- 隐藏索引（测试索引是否有用）
ALTER TABLE employees ALTER INDEX idx_salary INVISIBLE;  -- 隐藏
ALTER TABLE employees ALTER INDEX idx_salary VISIBLE;   -- 恢复
```

### 3.3 适合创建索引的 11 种情况（P131~P132）

| # | 场景 | 原因 |
|---|---|---|
| 1 | **WHERE 条件中的列** | 减少扫描行数 |
| 2 | **JOIN 连接中的列** | 加速表连接 |
| 3 | **ORDER BY 的列** | 避免 filesort |
| 4 | **GROUP BY 的列** | 避免临时表和 filesort |
| 5 | **SELECT 中频繁查询的列** | 考虑覆盖索引 |
| 6 | **高选择性的列**（重复值少） | 区分度高，扫描行数少 |
| 7 | **UPDATE/DELETE 的 WHERE 列** | 加速更新和删除 |
| 8 | **数据类型小的列** | 索引页能存更多条目 |
| 9 | **字符串列建前缀索引** | 节省空间 |

```sql
-- 前缀索引示例
CREATE INDEX idx_email_prefix ON employees(email(10));
-- 只存 email 前 10 个字符
```

### 3.4 不适合创建索引的 7 种情况（P133）

| # | 场景 | 原因 |
|---|---|---|
| 1 | **数据量小的表** | 全表扫描反而更快 |
| 2 | **更新频繁的列** | 维护索引成本高 |
| 3 | **低选择性的列**（如性别） | 索引区分度极低 |
| 4 | **TEXT/BLOB 大型字段** | 索引文件过大 |
| 5 | **不参与 WHERE/ORDER BY 的列** | 根本用不上 |

### 3.5 EXPLAIN 详解（P136~P139）

```sql
EXPLAIN SELECT * FROM employees WHERE dept_id = 10;
```

**EXPLAIN 各字段速查**：

| 字段 | 说明 | 重点关注 |
|---|---|---|
| **id** | 查询序号 | id 越大越先执行 |
| **select_type** | 查询类型 | SIMPLE/PRIMARY/SUBQUERY |
| **type** | **连接类型（重点！）** | const > eq_ref > ref > range > index > ALL |
| **possible_keys** | 可能用到的索引 | — |
| **key** | **实际用到的索引** | NULL = 没走索引 |
| **key_len** | 索引使用字节数 | 越小越好 |
| **rows** | 估算扫描行数 | **越小越好** |
| **Extra** | **额外信息（重点！）** | 见下表 |

**type 字段值（从好到差）**：

| type | 含义 | 优化目标 |
|---|---|---|
| **system** | 表只有一行 | 最优 |
| **const** | 主键/唯一索引等值查找 | 极好 |
| **eq_ref** | JOIN 时主键/唯一索引匹配 | 很好 |
| **ref** | 非唯一索引等值查找 | 好 |
| **range** | 索引范围扫描 | 可接受 |
| **index** | 全索引扫描 | 需优化 |
| **ALL** | **全表扫描（最差！）** | 必须优化 |

**Extra 常见值**：

| Extra | 含义 | 是否需要优化 |
|---|---|---|
| **Using index** | 覆盖索引，不用回表 | ✅ 最理想 |
| **Using index condition** | 索引下推（ICP） | ✅ 好 |
| **Using where** | 用 WHERE 过滤 | 正常 |
| **Using temporary** | 使用临时表 | ⚠️ 需要优化 |
| **Using filesort** | 文件排序（外排序） | ⚠️⚠️ 需要优化 |
| **Impossible WHERE** | 条件永远为 FALSE | ❌ SQL 写错了 |

**EXPLAIN 四种格式**：

```sql
-- 传统格式（表格）
EXPLAIN SELECT ...;

-- JSON 格式（最详细，含成本估算）
EXPLAIN FORMAT=JSON SELECT ...;

-- TREE 格式（MySQL 8.0+，显示执行顺序树）
EXPLAIN FORMAT=TREE SELECT ...;

-- 实际执行并显示真实时间
EXPLAIN ANALYZE SELECT ...;
```

### 3.6 慢查询日志（P134~P135）

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';

-- 设置阈值（秒）
SET GLOBAL long_query_time = 1;

-- 查看慢查询日志文件位置
SHOW VARIABLES LIKE 'slow_query_log_file';
```

**SHOW PROFILE**（MySQL 5.7 及以下）：

```sql
SET profiling = 1;
SELECT * FROM employees WHERE dept_id = 10;
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;  -- 查看各阶段耗时
```

### 3.7 索引失效的 11 种情况（P141~P142）

> **这是面试最常考的优化知识点！**

| # | 索引失效的情况 | 原因 | 示例 |
|---|---|---|---|
| 1 | **以 % 开头的 LIKE** | 无法利用索引有序性 | `LIKE '%abc'` |
| 2 | **OR 前后不全是索引列** | 部分列没索引，放弃索引 | `WHERE name='张' OR salary>5000` |
| 3 | **数据类型转换** | 隐式类型转换 | `WHERE age='25'`（age 是 INT） |
| 4 | **函数/表达式** | 对索引列使用函数 | `WHERE YEAR(hire_date)=2020` |
| 5 | **不等于（<> / !=）** | 范围太大 | `WHERE dept_id <> 10` |
| 6 | **IS NULL / IS NOT NULL** | NULL 处理特殊 | 可能不走索引 |
| 7 | **NOT IN / NOT EXISTS** | 非等值，范围大 | — |
| 8 | **最左前缀不满足** | 联合索引跳过最左列 | — |
| 9 | **范围查询右边列** | 范围后的列索引用不上 | `WHERE a>1 AND b=2`（b 用不上） |
| 10 | **索引列参与计算** | 破坏有序性 | `WHERE salary+100>5000` |
| 11 | **全表扫描更快** | 表很小或统计信息不准 | — |

### 3.8 查询优化策略（P143~P149）

#### JOIN 优化

- **小表驱动大表**：结果集小的表做驱动表
- **给被驱动表的连接列建索引**
- **尽量用 INNER JOIN** 替代 LEFT JOIN

#### JOIN 底层算法（P144）

| 算法 | 说明 | 适用场景 |
|---|---|---|
| **NLJ**（Nested-Loop Join） | 驱动表每行逐个匹配 | 小表做驱动 |
| **BNL**（Block Nested-Loop） | 整批读到 join_buffer 再匹配 | 无索引 JOIN |
| **INL**（Index Nested-Loop） | 被驱动表有索引 | 被驱动表有索引 |

#### 子查询优化（P145）

```sql
-- ❌ 低效：子查询
SELECT * FROM orders WHERE user_id IN (
  SELECT id FROM users WHERE status = 1
);

-- ✅ 高效：改写成 JOIN
SELECT DISTINCT o.* FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.status = 1;
```

#### 深分页优化（P146）

```sql
-- ❌ 低效：偏移量大了越来越慢
SELECT * FROM orders ORDER BY id LIMIT 100000, 10;

-- ✅ 优化：主键定位
SELECT * FROM orders WHERE id > 100000 ORDER BY id LIMIT 10;

-- ✅ 优化：子查询定位
SELECT o.* FROM orders o
JOIN (SELECT id FROM orders ORDER BY id LIMIT 100000, 10) t
ON o.id = t.id;
```

#### 覆盖索引（P147）

```sql
-- 覆盖索引：SELECT 的列 + WHERE 的列都在索引中
CREATE INDEX idx_salary_name ON employees(salary, name);
SELECT name, salary FROM employees WHERE salary > 5000;
-- Extra 显示：Using index ✅（不用回表）
```

#### 索引条件下推 ICP（P148）

MySQL 5.6+ 优化：把 WHERE 条件下推到存储引擎层，在索引遍历中就过滤数据。

```sql
-- 索引：(dept_id, name)
-- MySQL 5.5：先按 dept_id=10 取所有行，再 Server 层过滤 name
-- MySQL 5.6+：直接在索引层过滤 dept_id=10 AND name LIKE '张%'
```

---

## 四、容易出错的地方

1. **EXPLAIN 的 type**：至少要达到 range，避免 ALL
2. **索引失效**：函数、类型转换、%开头 LIKE 是最常见的陷阱
3. **覆盖索引**：`SELECT *` 无法利用覆盖索引
4. **深分页**：`LIMIT 100000, 10` 越往后越慢
5. **最左前缀**：联合索引 (a,b,c)，查 b=1 不走索引

---

## 五、和其他知识点的联系

| 关联知识 | 关系 |
|---|---|
| B+ 树原理 | 理解索引为什么有时失效 |
| 聚簇/二级索引 | 理解回表和覆盖索引 |
| 事务与锁 | 优化 SQL 减少锁持有时间 |
| 数据库设计 | 范式和索引设计的关系 |

---

## 六、面试可能怎么问

1. 说说索引失效的几种情况？
2. EXPLAIN 中 type 和 Extra 分别怎么看？
3. 什么是覆盖索引？什么是索引下推？
4. 深分页怎么优化？
5. 如何分析一条慢 SQL？

---

## 七、总结

- EXPLAIN 是 SQL 优化的核心工具，重点看 type 和 Extra
- 索引失效 11 种情况要熟记，面试常考
- 覆盖索引 + 索引下推是实用的优化手段
- 深分页用主键定位或子查询优化

---

## AI辅助思考

### 这个知识点真正重要的地方

本模块是**理论到实战的桥梁**。前面学了 B+ 树原理，这里教你怎么用 EXPLAIN 看 SQL 走没走索引、为什么没走。EXPLAIN + 索引失效是**面试双杀组合**——面试官问 SQL 优化，你答 EXPLAIN 分析 + 索引失效场景，基本满分。

### 和哪些知识点有关

- **B+ 树原理** → 理解为什么这些写法会导致索引失效
- **联合索引** → 最左前缀原则是索引失效的常见原因
- **覆盖索引** → 减少回表是性能优化的关键
- **ICP 索引下推** → MySQL 5.6+ 的重要优化

### 初学者容易误解什么

1. 以为"建了索引就一定走索引"——很多写法会导致索引失效
2. 以为 EXPLAIN 只看 key 列——type 和 Extra 才是关键
3. 以为 `SELECT *` 没什么——它会导致无法使用覆盖索引

### 实际项目中可能怎么用

- 上线前对核心 SQL 做 EXPLAIN 分析
- 慢查询日志定期分析，找出需要优化的 SQL
- 联合索引设计时考虑覆盖索引

### 下一步可以学什么

1. 数据库设计规范（范式、反范式）
2. 事务与锁（减少锁竞争也是优化方向）
3. 数据库调优策略

### AI 给我的学习建议

索引失效的 11 种情况建议做成卡片反复复习。EXPLAIN 的用法要多练习——在自己的测试库上建几张表，写不同的 SQL 用 EXPLAIN 看执行计划，比死记硬背有效得多。深分页优化是实际项目中非常常见的问题，值得重点掌握。
