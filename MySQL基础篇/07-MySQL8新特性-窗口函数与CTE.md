---
title: MySQL 8 新特性——窗口函数与CTE
tags:
  - 标签/MySQL
  - 标签/窗口函数
  - 标签/CTE
  - 标签/MySQL8
  - 标签/面试高频
source: 尚硅谷·宋红康 MySQL基础篇 P94~P95
created: 2025-01-01
---

# MySQL 8 新特性——窗口函数与 CTE

## 一、窗口函数（P94）

### 1.1 什么是窗口函数

窗口函数在**不改变行数**的前提下，对每一行计算一个基于"窗口"的聚合值。和 GROUP BY 的区别：GROUP BY 会合并行，窗口函数保留所有行。

```sql
-- GROUP BY：合并行，每个部门一行
SELECT department_id, AVG(salary) FROM employees GROUP BY department_id;

-- 窗口函数：保留每一行，同时显示部门平均值
SELECT name, department_id, salary,
  AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary
FROM employees;
```

### 1.2 常用窗口函数

| 函数 | 说明 | 示例 |
|---|---|---|
| **ROW_NUMBER()** | 行号（不重复，1,2,3...） | 排名 |
| **RANK()** | 排名（有并列会跳号，1,2,2,4） | 成绩排名 |
| **DENSE_RANK()** | 排名（有并列不跳号，1,2,2,3） | 成绩排名 |
| **NTILE(n)** | 分成 n 组 | 数据分桶 |
| **LAG(col, n)** | 前 n 行的值 | 环比分析 |
| **LEAD(col, n)** | 后 n 行的值 | 环比分析 |
| **FIRST_VALUE(col)** | 窗口内第一个值 | 最早记录 |
| **LAST_VALUE(col)** | 窗口内最后一个值 | 最新记录 |

### 1.3 语法结构

```sql
函数名() OVER (
  [PARTITION BY 分组列]     -- 按什么分组（窗口）
  [ORDER BY 排序列]         -- 组内怎么排序
  [ROWS/RANGE 范围]         -- 窗口范围（可选）
)
```

### 1.4 实战示例

```sql
-- 需求：每个部门内按薪资排名
SELECT name, department_id, salary,
  ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank_num
FROM employees;

-- 需求：取每个部门薪资前 3 名
SELECT * FROM (
  SELECT name, department_id, salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
  FROM employees
) t WHERE rn <= 3;

-- 需求：计算薪资的部门内累计求和
SELECT name, department_id, salary,
  SUM(salary) OVER (PARTITION BY department_id ORDER BY salary) AS cumulative_sum
FROM employees;

-- 需求：计算环比（与上一行的差值）
SELECT name, salary,
  salary - LAG(salary, 1) OVER (ORDER BY employee_id) AS salary_diff
FROM employees;
```

### 1.5 三种排名函数对比

| 函数 | 并列排名 | 排名值 | 适用场景 |
|---|---|---|---|
| ROW_NUMBER() | 不处理 | 1,2,3,4,5 | 取 Top N |
| RANK() | 并列跳号 | 1,2,2,4,5 | 成绩排名 |
| DENSE_RANK() | 并列不跳 | 1,2,2,3,4 | 密集排名 |

```sql
-- 假设薪资为：10000, 10000, 8000, 7000
-- ROW_NUMBER: 1, 2, 3, 4
-- RANK:       1, 1, 3, 4
-- DENSE_RANK: 1, 1, 2, 3
```

---

## 二、CTE 公用表表达式（P95）

### 2.1 非递归 CTE

```sql
-- 基本语法
WITH cte_name AS (
  SELECT ...
)
SELECT * FROM cte_name;

-- 实例：查询每个部门平均薪资高于 8000 的部门
WITH dept_avg AS (
  SELECT department_id, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY department_id
)
SELECT d.department_name, da.avg_sal
FROM dept_avg da
JOIN departments d ON da.department_id = d.department_id
WHERE da.avg_sal > 8000;
```

### 2.2 多个 CTE 链式使用

```sql
WITH
  cte1 AS (SELECT ... FROM table1),
  cte2 AS (SELECT ... FROM cte1),
  cte3 AS (SELECT ... FROM cte2)
SELECT * FROM cte3;
```

### 2.3 递归 CTE

```sql
-- 查找员工的完整管理层级（从底层到顶层）
WITH RECURSIVE emp_hierarchy AS (
  -- 锚定成员（基础查询）
  SELECT employee_id, first_name, manager_id, 1 AS level
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- 递归成员
  SELECT e.employee_id, e.first_name, e.manager_id, h.level + 1
  FROM employees e
  JOIN emp_hierarchy h ON e.manager_id = h.employee_id
)
SELECT * FROM emp_hierarchy ORDER BY level;
```

**递归 CTE 结构**：
1. **锚定成员**：非递归的初始查询
2. **UNION ALL**
3. **递归成员**：引用自身的查询
4. 终止条件：递归成员返回空集时自动停止

---

## 三、CTE vs 子查询 vs 临时表

| 方式 | 可读性 | 复用性 | 性能 |
|---|---|---|---|
| **CTE** | ✅ 最好（先定义后使用） | ✅ 可多次引用 | 与子查询相同 |
| 子查询 | ❌ 嵌套深时难读 | ❌ 不能复用 | 相同 |
| 临时表 | 一般 | ✅ 可复用 | 需要额外 IO |

---

## 四、容易出错的地方

1. **窗口函数不能在 WHERE 中使用**：`WHERE ROW_NUMBER() > 3` 是错的，要用子查询包一层
2. **PARTITION BY 可省略**：省略则整个结果集是一个窗口
3. **ORDER BY 在 OVER 内外含义不同**：OVER 内是窗口排序，外是最终结果排序
4. **递归 CTE 必须有终止条件**：否则无限循环
5. **CTE 是 MySQL 8.0+ 特性**：5.7 不支持

---

## 五、面试可能怎么问

1. 窗口函数和 GROUP BY 的区别？
2. ROW_NUMBER、RANK、DENSE_RANK 的区别？
3. CTE 和子查询哪个更好？
4. 递归 CTE 的使用场景？

---

## 六、总结

- 窗口函数：保留所有行的同时计算聚合值
- 三种排名：ROW_NUMBER（不重复）、RANK（跳号）、DENSE_RANK（不跳号）
- CTE：用 WITH 定义临时命名查询，提高可读性
- 递归 CTE：处理层级数据（组织架构、树形结构）

---

## AI辅助思考

### 这个知识点真正重要的地方

窗口函数是 **MySQL 8.0 最重要的新特性**，面试中出现频率非常高。"每个部门取薪资前 3 名" 这类问题用窗口函数一行搞定，用传统 SQL 要写很长的子查询。CTE 让复杂 SQL 更易读。

### 和哪些知识点有关

- **GROUP BY** → 窗口函数是 GROUP BY 的"升级版"
- **子查询** → CTE 是子查询的"可读版"
- **EXPLAIN** → 窗口函数的执行计划分析

### 实际项目中可能怎么用

- 排名查询：成绩排名、销量排名
- Top N 问题：每组取前 N 条
- 环比分析：LAG/LEAD 计算同比环比
- 层级查询：递归 CTE 查组织架构

### 下一步可以学什么

1. MySQL 高级篇（索引、事务、锁、MVCC）
2. EXPLAIN 执行计划分析

### AI 给我的学习建议

窗口函数是**必学必练**的内容。建议你在测试库上多写几个实战需求：排名、Top N、累计求和、环比。CTE 主要提升可读性，复杂 SQL 建议优先用 CTE 组织。
