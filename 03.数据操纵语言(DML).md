# 📝 数据操纵语言（DML）| 增删改查之“改”字诀
 
> **前置**：已掌握 DDL 建库建表，MySQL 环境就绪

**DML = Data Manipulation Language**，数据操纵语言。  
它不碰表结构，只负责 **表里的数据** —— 也就是 **增（INSERT）、删（DELETE）、改（UPDATE）**。查询（SELECT）属于 DQL，我们下一章再见。

> 比喻时间：  
> - DDL 是盖房子（砌墙、铺管）  
> - DML 是往房子里搬家具、扔垃圾、换沙发  
> - DQL 是你进屋找东西  
>   
> 本章讲的就是 **搬家具、扔垃圾、换沙发** 的操作。

---

## 📌 目录

1. [DML 核心概念：操作的最小单位是“行”](#1-dml-核心概念操作的最小单位是行)
2. [插入数据（INSERT）—— 给表“喂”数据](#2-插入数据insert--给表喂数据)
3. [修改数据（UPDATE）—— 让数据“变心”](#3-修改数据update--让数据变心)
4. [删除数据（DELETE）—— 让数据“消失”](#4-删除数据delete--让数据消失)
5. [实战练习题（附答案）](#5-实战练习题附答案)
6. [总结 & 避坑指南](#6-总结--避坑指南)

---

## 1. DML 核心概念：操作的最小单位是“行”

在关系型数据库中，**数据操作的最基本单位永远是“行”**（一条记录）。

- 插入 → 一次插一行或多行
- 修改 → 修改某一行（或一批行）的某些列
- 删除 → 删除某一行（或一批行）

你不能只修改某一列的“半个值”，也不能只删除某一列的“一半内容”。  
**行是原子操作单位**。

```text
+--------+------+-----+
| name   | age  | sex |
+--------+------+-----+
| 张三   | 18   | 男  |   ← 这是一行
| 李四   | 20   | 女  |   ← 这是另一行
+--------+------+-----+
```

---

## 2. 插入数据（INSERT）—— 给表“喂”数据

### 2.1 语法全览

```sql
-- 1. 为所有列插入一行（值的顺序必须和表结构完全一致）
INSERT INTO 表名 VALUES (值1, 值2, ...);

-- 2. 为指定列插入一行（推荐，更安全）
INSERT INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...);

-- 3. 一次插入多行（逗号分隔多个括号）
INSERT INTO 表名 (列1, 列2, ...) VALUES 
    (值1a, 值2a, ...),
    (值1b, 值2b, ...),
    ...;
```

**核心细节：**
- `VALUES` 也可以写成 `VALUE`，但官方推荐 `VALUES`。
- 字符串和日期必须用 **单引号** 括起来，比如 `'张三'`, `'2025-01-01'`。
- 如果某列有默认值（`DEFAULT`）或者允许 `NULL`，你可以在指定列插入时省略它。
- 自增列（`AUTO_INCREMENT`）通常不用手动写值，让数据库自动生成。

### 2.2 实战表准备

```sql
CREATE DATABASE IF NOT EXISTS dml_db;
USE dml_db;

CREATE TABLE students (
    stu_id INT COMMENT '学号',
    stu_name VARCHAR(100) COMMENT '姓名',
    stu_age TINYINT UNSIGNED COMMENT '年龄（0~255）',
    stu_birthday DATE COMMENT '生日',
    stu_height DECIMAL(4,1) DEFAULT 200 COMMENT '身高，保留一位小数'
);
```

### 2.3 插入示例（对应 PPT 练习）

```sql
-- 1. 插入一名学生的所有信息（列顺序必须与表定义一致）
INSERT INTO students VALUES (1, '张三', 20, '2004-03-15', 175.5);

-- 2. 只插入部分列（学号、名字、年龄），其他用默认值或 NULL
INSERT INTO students (stu_id, stu_name, stu_age) VALUES (2, '李四', 22);

-- 3. 一次插入多行（所有列）
INSERT INTO students (stu_id, stu_name, stu_age, stu_birthday, stu_height) VALUES
    (3, '王五', 21, '2003-07-20', 180.0),
    (4, '赵六', 19, '2005-01-10', 168.5);

-- 4. 插入时某些列为 NULL（前提是列允许 NULL）
INSERT INTO students (stu_id, stu_name, stu_age) VALUES (5, '孙七', 23);
-- 此时 stu_birthday 和 stu_height 为 NULL（因为没有默认值且未指定）
```

> ⚠️ **注意**：如果列有 `NOT NULL` 约束且没有 `DEFAULT`，那么插入时必须给值，否则报错。

---

## 3. 修改数据（UPDATE）—— 让数据“变心”

### 3.1 语法全览

```sql
-- 1. 修改表中所有行（危险！务必谨慎）
UPDATE 表名 SET 列1 = 新值1, 列2 = 新值2, ...;

-- 2. 只修改符合条件的行（推荐，必须加 WHERE）
UPDATE 表名 SET 列1 = 新值1, 列2 = 新值2, ... WHERE 条件;
```

**WHERE 条件** 可以是：
- 单一条件：`WHERE stu_id = 8`
- 复合条件：`WHERE stu_age < 20 AND stu_height > 170`
- 使用 `IN`、`BETWEEN`、`LIKE` 等运算符（后面 DQL 会细讲）

### 3.2 实战示例（基于 PPT 练习题）

先插入一批数据（PPT 给的样例）：

```sql
INSERT INTO students (stu_id, stu_name, stu_age, stu_birthday, stu_height) VALUES
    (6, '张三', 21, '2002-05-10', 175.5),
    (7, '李四', 20, '2003-02-15', 168.0),
    (8, '王五', 22, '2001-09-20', 180.2),
    (9, '赵六', 19, '2004-03-08', 165.8),
    (10, '钱七', 23, '2000-12-01', 172.3),
    (11, '孙八', 20, '2003-06-25', 160.5),
    (12, '周九', 21, '2002-11-18', 175.0),
    (13, '吴十', 22, '2001-04-30', 168.7),
    (14, '郑十一', 19, '2004-08-12', 185.5),
    (15, '王十二', 23, '2000-07-05', 170.1);
```

现在做更新练习：

```sql
-- 1. 将学号为 8 的学生的姓名改为 '黄六'
UPDATE students SET stu_name = '黄六' WHERE stu_id = 8;

-- 2. 将年龄小于 20 岁的学生的身高增加 2.0
UPDATE students SET stu_height = stu_height + 2.0 WHERE stu_age < 20;

-- 3. 将学号为 11 的学生的生日改为 '2003-07-10'，且年龄改为 21
UPDATE students SET stu_birthday = '2003-07-10', stu_age = 21 WHERE stu_id = 11;

-- 4. 将所有学生的年龄减少 1 岁（注意：没有 WHERE 就是全表修改）
UPDATE students SET stu_age = stu_age - 1;
```

> ⚠️ **全表更新非常危险**！如果你忘了写 `WHERE`，整个表的该列都会被改掉。  
> 建议先 `SELECT * FROM students WHERE 条件` 确认目标行，再执行 `UPDATE`。

---

## 4. 删除数据（DELETE）—— 让数据“消失”

### 4.1 语法全览

```sql
-- 1. 删除表中所有行（清空表，但表结构还在）
DELETE FROM 表名;

-- 2. 删除符合条件的行（加 WHERE）
DELETE FROM 表名 WHERE 条件;
```

**注意：**  
- `DELETE` 是逐行删除，会产生事务日志，可以回滚（前提是没 `COMMIT`）。  
- 如果你想快速清空全表且不需要回滚，用 `TRUNCATE TABLE 表名` 更快，但它属于 DDL，不可回滚。

### 4.2 实战示例

基于上面的 `students` 表：

```sql
-- 1. 删除年龄大于 23 的学员
DELETE FROM students WHERE stu_age > 23;

-- 2. 删除身高高于 200 且学号大于 10 的数据
DELETE FROM students WHERE stu_height > 200 AND stu_id > 10;

-- 3. 删除身高高于 200 或学号大于 10 的数据
DELETE FROM students WHERE stu_height > 200 OR stu_id > 10;

-- 4. 删除所有学生数据（走火入魔行为）
DELETE FROM students;   -- 全表删除，谨慎！
```

> ⚠️ `DELETE` 不加 `WHERE` 就是 **删全表**，数据量大的时候会很慢（因为逐行删除并记录日志）。  
> 如果你真的想清空表，而且不关心回滚，用 `TRUNCATE TABLE students` 会快得多。

---

## 5. 实战练习题（附答案）

为了巩固，以下题目请自己先思考，再对照答案。

### 5.1 插入题

基于上面的 `students` 表结构，完成：

1. 插入一个完整学生：学号 16，姓名“小明”，年龄 18，生日 `2006-08-22`，身高 170.0。
2. 插入两个学生，只提供学号和姓名：学号 17 姓名“小红”，学号 18 姓名“小刚”。
3. 插入一个学生，学号 19，姓名“大雄”，年龄 10，其他字段用默认值。

<details>
<summary>点击查看答案</summary>

```sql
-- 1
INSERT INTO students (stu_id, stu_name, stu_age, stu_birthday, stu_height)
VALUES (16, '小明', 18, '2006-08-22', 170.0);

-- 2
INSERT INTO students (stu_id, stu_name) VALUES (17, '小红'), (18, '小刚');

-- 3
INSERT INTO students (stu_id, stu_name, stu_age) VALUES (19, '大雄', 10);
-- 此时 stu_birthday 为 NULL，stu_height 为默认值 200（因为 DEFAULT 200）
```
</details>

### 5.2 更新题

1. 将学号 17 的学生年龄改为 19。
2. 将所有年龄为 NULL 的学生年龄设置为 0（假设 NULL 代表未知）。
3. 将身高低于 160 的学生身高增加 5.0。

<details>
<summary>点击查看答案</summary>

```sql
-- 1
UPDATE students SET stu_age = 19 WHERE stu_id = 17;

-- 2
UPDATE students SET stu_age = 0 WHERE stu_age IS NULL;

-- 3
UPDATE students SET stu_height = stu_height + 5.0 WHERE stu_height < 160;
```
</details>

### 5.3 删除题

1. 删除学号为 18 的学生。
2. 删除所有年龄大于 30 的学生。
3. 删除所有身高为 NULL 的学生。

<details>
<summary>点击查看答案</summary>

```sql
-- 1
DELETE FROM students WHERE stu_id = 18;

-- 2
DELETE FROM students WHERE stu_age > 30;

-- 3
DELETE FROM students WHERE stu_height IS NULL;
```
</details>

---

## 6. 总结 & 避坑指南

### 6.1 核心命令速查表

| 操作 | 关键字 | 示例 | 注意事项 |
|------|--------|------|----------|
| 插入 | `INSERT INTO ... VALUES` | `INSERT INTO t (a,b) VALUES (1,2);` | 字符/日期加单引号；列名与值顺序对应 |
| 插入多行 | 同上，多个 `(),()` | `VALUES (1,2), (3,4);` | 性能比多次单行插入好 |
| 更新 | `UPDATE ... SET ... WHERE` | `UPDATE t SET a=10 WHERE b=2;` | **一定要写 WHERE**，否则全表更新 |
| 删除 | `DELETE FROM ... WHERE` | `DELETE FROM t WHERE a=5;` | **一定要写 WHERE**，否则全表删除 |

### 6.2 血泪教训区

- ❌ **忘记 WHERE**：  
  执行 `UPDATE students SET stu_age = 18` 后，所有人年龄都变 18，你无法挽回（如果没有备份或事务没提交）。
  ✅ **先查后改**：  
  `SELECT * FROM students WHERE stu_id = 8;` 确认再 `UPDATE`。

- ❌ **字符串/日期不加引号**：  
  `INSERT ... VALUES (张三)` → 报错，必须 `'张三'`。

- ❌ **插入时列数与值数不匹配**：  
  `INSERT INTO students (stu_id, stu_name) VALUES (1)` → 报错，值个数必须跟列名个数一致。

- ❌ **用 DELETE 删除大表**：  
  如果表有几十万行，`DELETE FROM table` 非常慢，而且锁表严重。  
  ✅ 用 `TRUNCATE TABLE table` 极快，但不能回滚。

- ❌ **更新/删除时条件写错逻辑**：  
  `WHERE stu_age > 20 AND stu_age < 10` 永远为 false，不会报错但也不会改任何数据 —— 你可能会以为“操作成功了”，其实啥也没干。

### 6.3 DML 与事务的关系（预告）

- `INSERT`、`UPDATE`、`DELETE` 都是可以被 **事务** 控制的（`START TRANSACTION` / `COMMIT` / `ROLLBACK`）。
- 在自动提交模式下（MySQL 默认），每条 DML 都会立即生效。你可以关闭自动提交来练习：
  ```sql
  SET autocommit = 0;   -- 关闭自动提交
  UPDATE ...;           -- 不会立刻生效
  ROLLBACK;             -- 撤销刚才的更新
  SET autocommit = 1;   -- 恢复自动提交
  ```
  事务的内容会在 TCL 章节详细讲，现在你只需要知道 **DML 不是“一发入魂”不可逆的**。

---

## 🎉 最后一句

DML 是你和数据库交互最频繁的操作。  
**INSERT** 让你给表喂数据，**UPDATE** 让你修改错误，**DELETE** 让你清理垃圾 —— 但请永远记住：**写 WHERE 之前，先想想有没有备份**。

> 如果你敢在生产环境 `DELETE FROM important_table` 不带 `WHERE`，那么恭喜你，你已经集齐了“提桶跑路”的所有条件。

**下一章预告：** DQL 查询（SELECT）—— 从表里优雅地捞出你要的数据，不用再靠眼睛扫行数了！

---

📌 这份笔记同样开源在 [你的 GitHub 仓库]。如果发现错误或有更好玩的例子，欢迎提 PR 或 Issue。  
**动手敲一遍所有例子，比你读十遍都管用！** 现在就打开 MySQL 试试吧～
