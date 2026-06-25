---
title: 多表查询与JOIN
tags:
  - 标签/MySQL
  - 标签/JOIN
  - 标签/多表查询
  - 标签/面试高频
source: 尚硅谷·宋红康 MySQL基础篇 P25~P31
created: 2025-01-01
---

# 多表查询与 JOIN

## 一、这个知识点是什么

当数据分布在多张表中时，需要通过 JOIN 把相关数据关联起来查询。这是 SQL 中**最重要、最常考**的操作之一。

## 二、为什么要学它

- 实际项目中几乎没有单表查询，数据都是分散在多张表中的
- 7 种 JOIN 是**面试必考**
- 理解 JOIN 有助于后续学习索引优化

---

## 三、核心内容

### 3.1 笛卡尔积——新手最常犯的错

```sql
-- ❌ 错误：没加连接条件，产生笛卡尔积
SELECT * FROM employees, departments;
-- 107 个员工 × 27 个部门 = 2889 条垃圾数据！

-- ✅ 正确：加上连接条件
SELECT * FROM employees e, departments d
WHERE e.department_id = d.department_id;
```

### 3.2 SQL92 vs SQL99 语法

```sql
-- SQL92（老语法）：连接条件和过滤混在一起
SELECT * FROM employees e, departments d
WHERE e.dept_id = d.id AND e.salary > 5000;

-- SQL99（新语法，推荐）：ON 负责连接，WHERE 负责过滤
SELECT * FROM employees e
INNER JOIN departments d ON e.dept_id = d.id
WHERE e.salary > 5000;
```

### 3.3 7 种 JOIN 全掌握（面试核心）

| # | JOIN 类型 | SQL 写法 | 结果 |
|---|---|---|---|
| 1 | **内连接** | `A INNER JOIN B ON ...` | 两表匹配的（交集） |
| 2 | **左外连接** | `A LEFT JOIN B ON ...` | 左表全要，右表匹配的 |
| 3 | **右外连接** | `A RIGHT JOIN B ON ...` | 右表全要，左表匹配的 |
| 4 | **左独有** | `A LEFT JOIN B ON ... WHERE B.id IS NULL` | 只在左表的 |
| 5 | **右独有** | `A RIGHT JOIN B ON ... WHERE A.id IS NULL` | 只在右表的 |
| 6 | **全外连接** | UNION 模拟（MySQL 不支持 FULL JOIN） | 两表全部 |
| 7 | **全外去交集** | 4 UNION 5 | 排除共同部分 |

```sql
-- 全外连接模拟（MySQL 不支持 FULL JOIN）
SELECT * FROM A LEFT JOIN B ON A.id = B.id WHERE B.id IS NULL
UNION
SELECT * FROM A RIGHT JOIN B ON A.id = B.id WHERE A.id IS NULL;
```

### 3.4 自连接

```sql
-- 找每个员工的上级（上级也在同一张表中）
SELECT e1.name AS 员工, e2.name AS 上级
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;
```

### 3.5 非等值连接

```sql
-- 查薪资等级（BETWEEN 匹配区间）
SELECT e.name, e.salary, g.grade_level
FROM employees e
JOIN salary_grades g ON e.salary BETWEEN g.low_salary AND g.high_salary;
```

### 3.6 NATURAL JOIN 与 USING

```sql
-- NATURAL JOIN：自动匹配同名列（谨慎使用！）
SELECT * FROM employees NATURAL JOIN departments;

-- USING：明确指定连接列
SELECT * FROM employees JOIN departments USING (department_id);
```

---

## 四、容易出错的地方

1. **忘加连接条件** → 笛卡尔积，数据爆炸
2. **ON 和 WHERE 混淆** → ON 是连接条件，WHERE 是过滤条件
3. **LEFT JOIN 的 WHERE 放错位置** → 放在 ON 后面会过滤掉左表数据
4. **MySQL 不支持 FULL JOIN** → 要用 UNION 模拟
5. **自连接必须用别名** → 同一张表必须取两个不同的别名

---

## 五、和其他知识点的联系

| 关联知识 | 关系 |
|---|---|
| 索引优化 | JOIN 的连接列建索引可大幅加速 |
| EXPLAIN | 查看 JOIN 是否走了索引 |
| 子查询 | 有些 JOIN 可以用子查询替代 |

---

## 六、面试可能怎么问

1. 说说 7 种 JOIN 的区别？
2. INNER JOIN 和 LEFT JOIN 的区别？
3. MySQL 为什么不支持 FULL JOIN？
4. ON 和 WHERE 在 JOIN 中的区别？

---

## 七、总结

- JOIN 的本质是按条件组合多张表的数据
- 7 种 JOIN：内连接、左/右外连接、左/右独有、全外连接、全外去交集
- 连接列建索引是优化 JOIN 的关键

---

## AI辅助思考

### 这个知识点真正重要的地方

7 种 JOIN 是**面试高频中的高频**。能画出文氏图并写出每种 JOIN 的 SQL，基本满分。LEFT JOIN 的过滤条件放在 ON 和 WHERE 的区别是经典陷阱题。

### 和哪些知识点有关

- **索引** → JOIN 连接列建索引是优化重点
- **子查询** → 有些场景子查询和 JOIN 可以互换
- **EXPLAIN** → 查看 JOIN 的执行计划

### 下一步可以学什么

1. 函数（单行函数 + 聚合函数）
2. 子查询
3. 索引优化（给 JOIN 列建索引）

### AI 给我的学习建议

建议你自己画 7 种 JOIN 的文氏图，然后在测试库上写 SQL 验证结果。理解 ON 和 WHERE 的区别比死记语法更重要。
