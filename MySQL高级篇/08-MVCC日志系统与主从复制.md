---
title: MVCC、日志系统与主从复制
tags:
  - 标签/MySQL
  - 标签/MVCC
  - 标签/binlog
  - 标签/主从复制
  - 标签/备份恢复
  - 标签/面试高频
source: 尚硅谷·宋红康 MySQL高级篇 P183~P199
created: 2025-01-01
---

# MVCC、日志系统与主从复制

## 一、这个知识点是什么

本模块涵盖 MySQL 高级篇的最后三个核心主题：
1. **MVCC**——多版本并发控制，实现读写不阻塞
2. **日志系统**——六大日志文件的作用与配置
3. **主从复制与备份**——生产环境高可用基础

## 二、为什么要学它

- MVCC 是**面试最难最核心**的部分，但理解了就超越大部分候选人
- binlog + 主从复制是生产环境标配
- 备份恢复是 DBA 的保命技能

---

## 三、核心内容

### 3.1 MVCC 多版本并发控制（P183~P186）

#### 核心思想

每行数据保存多个版本，读操作读历史版本，写操作创建新版本，**读写互不阻塞**。

```
传统并发：写锁阻塞读 → 读不了
MVCC 并发：写操作写新版本 → 读操作读旧版本 → 互不干扰
```

#### 两类读操作

| 读类型 | 含义 | 示例 |
|---|---|---|
| **快照读** | 读取历史版本 | 普通 SELECT |
| **当前读** | 读取最新版本 | `SELECT ... FOR UPDATE`、INSERT/UPDATE/DELETE |

#### MVCC 三剑客

**1. 隐藏字段**（每行自动添加）：

| 字段 | 大小 | 作用 |
|---|---|---|
| `DB_TRX_ID` | 6 字节 | 最后修改该记录的事务 ID |
| `DB_ROLL_PTR` | 7 字节 | 回滚指针，指向 Undo Log |
| `DB_ROW_ID` | 6 字节 | 隐藏主键（无显式主键时使用） |

**2. Undo Log 版本链**：

```
id=1 的版本链：
┌──────────────────────────────────────────┐
│  最新版本（当前行）                        │
│  salary=6000, trx_id=3, roll_ptr=→指针2   │
│          │                               │
│          └──→ Undo Log 2                  │
│                salary=5000, trx_id=2      │
│                roll_ptr=→指针1             │
│                      │                   │
│                      └──→ Undo Log 1     │
│                            salary=3000   │
└──────────────────────────────────────────┘
```

**3. ReadView（读视图）**：

| 字段 | 说明 |
|---|---|
| **m_ids** | 活跃事务 ID 列表 |
| **min_trx_id** | 最小活跃事务 ID |
| **max_trx_id** | 创建 ReadView 时最大事务 ID + 1 |
| **creator_trx_id** | 创建该 ReadView 的事务 ID |

**可见性判断规则（面试必考！）**：

```
读取一行的 trx_id：
  trx_id == creator_trx_id  → ✅ 可见（自己改的）
  trx_id < min_trx_id       → ✅ 可见（已提交事务）
  trx_id >= max_trx_id      → ❌ 不可见（未开始的事务）
  trx_id 在 m_ids 中         → ❌ 不可见（活跃事务，未提交）
  否则                       → ✅ 可见（已提交）
```

#### 两种隔离级别下 MVCC 的差异

| 隔离级别 | ReadView 创建时机 | 效果 |
|---|---|---|
| **READ COMMITTED** | **每次 SELECT 都创建新 ReadView** | 能看到其他事务已提交的最新数据 |
| **REPEATABLE READ** | **事务第一次 SELECT 时创建，后续复用** | 始终看到事务开始时的快照 |

#### MVCC 如何解决幻读

- **快照读**（普通 SELECT）：MVCC 保证看不到其他事务插入的新行
- **当前读**（`SELECT ... FOR UPDATE`）：间隙锁阻止其他事务插入

---

### 3.2 MySQL 六大日志（P187~P194）

| 日志 | 文件 | 作用 |
|---|---|---|
| **错误日志** | hostname.err | 记录启动/运行/停止的错误 |
| **通用查询日志** | hostname.log | 记录所有 SQL（通常关闭） |
| **慢查询日志** | hostname-slow.log | 记录执行慢的 SQL |
| **二进制日志（binlog）** | binlog.000001 | 记录数据变更（主从复制、恢复） |
| **Redo 日志** | ib_logfile0/1 | 事务持久性，崩溃恢复 |
| **Undo 日志** | Undo 表空间 | 事务回滚，MVCC |

#### binlog 详解（P189~P190）

**binlog 三大作用**：
1. **主从复制**：从库读取 binlog 重放
2. **数据恢复**：按时间点恢复
3. **审计**：追踪数据变更

**binlog 格式**：

| 格式 | 说明 | 优点 | 缺点 |
|---|---|---|---|
| **STATEMENT** | 记录 SQL 语句 | 日志小 | 某些函数主从不一致 |
| **ROW** | 记录行变更（默认） | 准确 | 日志大 |
| **MIXED** | 混合模式 | 自动选择 | — |

```sql
-- 查看 binlog
SHOW BINARY LOGS;
SHOW MASTER STATUS;

-- 删除旧 binlog
PURGE BINARY LOGS BEFORE '2024-01-01';
SET GLOBAL expire_logs_days = 7;
```

#### 两阶段提交（2PC）

```
1. Prepare 阶段：InnoDB 写 Redo Log（prepare 状态）
2. Commit 阶段：写 binlog → InnoDB 写 Redo Log（commit 状态）
```

**为什么需要 2PC？** 保证 Redo Log 和 binlog 的一致性，用于崩溃恢复。

---

### 3.3 主从复制（P191~P194）

#### 复制原理

```
主库                      从库
  │                        │
  ├─ 1. 写 binlog ────────┤
  │                        │
  ├─ 2. binlog dump ─────→├─ 3. IO 线程写入 relay log
  │                        │
  │                        ├─ 4. SQL 线程重放 relay log
```

#### 搭建步骤（P192）

```sql
-- 主库配置
-- my.cnf: server-id=1, log-bin=binlog
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';

-- 从库配置
CHANGE MASTER TO
  MASTER_HOST='master_host',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='binlog.000001',
  MASTER_LOG_POS=154;

START SLAVE;
```

#### 主从延迟与一致性（P194）

| 方案 | 说明 |
|---|---|
| **半同步复制** | 主库等至少一个从库确认收到 binlog |
| **GTID** | 全局事务 ID，方便故障切换 |
| **读写分离** | 写主库，读从库，容忍一定延迟 |
| **强制走主库** | 对一致性要求高的读，走主库 |

---

### 3.4 数据备份与恢复（P195~P198）

#### 逻辑备份（mysqldump）

```bash
# 备份单个数据库
mysqldump -u root -p mydb > mydb_backup.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > all_backup.sql

# 恢复
mysql -u root -p mydb < mydb_backup.sql
```

#### 物理备份（xtrabackup）

```bash
# 全量备份
xtrabackup --backup --target-dir=/backup/full

# 恢复
xtrabackup --prepare --target-dir=/backup/full
xtrabackup --copy-back --target-dir=/backup/full
```

**逻辑备份 vs 物理备份**：

| 类型 | 工具 | 优点 | 缺点 |
|---|---|---|---|
| 逻辑备份 | mysqldump | 跨平台、可读、可编辑 | 慢、文件大 |
| 物理备份 | xtrabackup | 快、小、支持增量 | 依赖平台 |

#### 删库防护（P198）

| 措施 | 说明 |
|---|---|
| 权限控制 | 生产环境禁止普通用户 DROP 权限 |
| 延迟复制 | 从库延迟 1 小时，误删可恢复 |
| 定期备份 | 全量 + 增量 |
| binlog | 开启 binlog，可基于时间点恢复 |
| 堡垒机 | 所有操作审计，高危操作二次确认 |

**恢复流程**：停止应用 → 从备份恢复 → 用 binlog 恢复到误删前时间点

---

## 四、容易出错的地方

1. **ReadView 创建时机**：RC 每次 SELECT 创建新 ReadView，RR 只创建一次
2. **快照读 vs 当前读**：普通 SELECT 是快照读，FOR UPDATE 是当前读
3. **binlog vs Redo Log**：binlog 是 Server 层日志，Redo Log 是 InnoDB 引擎层日志
4. **两阶段提交**：保证 binlog 和 Redo Log 一致性
5. **半同步复制**：不是"同步复制"，只等一个从库确认

---

## 五、和其他知识点的联系

| 关联知识 | 关系 |
|---|---|
| 事务隔离级别 | MVCC 是 RC 和 RR 的实现机制 |
| 锁机制 | 当前读依赖锁，快照读依赖 MVCC |
| Undo Log | MVCC 版本链的基础 |
| Redo Log | 与 binlog 配合实现 2PC |
| Buffer Pool | 脏页通过后台线程刷盘 |

---

## 六、面试可能怎么问

1. MVCC 是怎么实现的？三要素是什么？
2. ReadView 在 RC 和 RR 下有什么区别？
3. MVCC 怎么解决幻读？
4. Redo Log 和 binlog 的区别？两阶段提交是什么？
5. 主从复制的原理是什么？
6. binlog 的三种格式有什么区别？
7. 误删数据库怎么恢复？

---

## 七、总结

- MVCC = 隐藏字段 + Undo Log 版本链 + ReadView
- RC 每次 SELECT 新建 ReadView，RR 只创建一次
- Redo Log（引擎层）保证持久性，binlog（Server 层）用于复制和恢复
- 两阶段提交保证 Redo Log 和 binlog 一致
- 主从复制流程：主库写 binlog → 从库 IO 线程拉取 → SQL 线程重放
- 备份策略：定期全量 + 增量 + binlog 增量恢复

---

## AI辅助思考

### 这个知识点真正重要的地方

**MVCC 是面试最高难度的 MySQL 知识点之一。** 能讲清楚 ReadView + 版本链 + 可见性规则，基本能碾压 90% 的候选人。binlog + 主从复制是生产环境必备知识。

### 和哪些知识点有关

- **Undo Log** → MVCC 版本链的基础
- **事务隔离级别** → MVCC 在不同级别下的行为不同
- **锁机制** → 当前读和快照读的区别
- **两阶段提交** → Redo Log + binlog 的协调

### 初学者容易误解什么

1. 以为 MVCC 完全不需要锁——当前读（FOR UPDATE）仍然需要锁
2. 以为 RR 级别完全解决幻读——快照读解决了，当前读还需要间隙锁
3. 以为 binlog 和 Redo Log 是同一个东西——它们属于不同层级

### 实际项目中可能怎么用

- 理解 MVCC 可以帮助解释"为什么读不阻塞写"
- 主从复制是读写分离、高可用的基础
- 备份恢复方案是每个项目的标配

### 下一步可以学什么

1. MySQL 实战：搭建主从复制环境
2. Redis 缓存与数据库一致性
3. 分库分表中间件（ShardingSphere）

### AI 给我的学习建议

MVCC 是本课程最难的内容，建议分步理解：先搞懂隐藏字段 → 再理解版本链 → 最后理解 ReadView 的可见性规则。可以画图模拟事务并发场景，逐步推演。binlog 和主从复制偏运维，了解原理即可，等需要搭建时再实操。备份恢复建议在自己的测试环境练习一遍。
