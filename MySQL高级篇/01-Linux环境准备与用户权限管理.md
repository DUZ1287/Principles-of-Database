---
title: Linux环境准备与用户权限管理
tags:
  - 标签/MySQL
  - 标签/Linux
  - 标签/用户管理
  - 标签/权限管理
source: 尚硅谷·宋红康 MySQL高级篇 P96~P108
created: 2025-01-01
---

# Linux环境准备与用户权限管理

## 一、这个知识点是什么

本模块是 MySQL 高级篇的起点，涵盖两个核心主题：
1. **Linux 环境下的 MySQL 安装与配置**——企业生产环境几乎 100% 使用 Linux
2. **用户管理与权限控制**——保证数据库安全访问

---

## 二、为什么要学它

| 原因 | 说明 |
|---|---|
| 企业标配 | 生产环境几乎都是 Linux 服务器 |
| 面试必问 | 权限管理、最小权限原则 |
| 安全基础 | 用户和权限是数据库安全的第一道防线 |

---

## 三、核心内容

### 3.1 Linux 环境准备（P97~P100）

#### CentOS 环境搭建

**为什么用 Linux？**

| 对比维度 | Windows MySQL | Linux MySQL |
|---|---|---|
| 企业使用率 | 仅开发测试 | **生产环境 100%** |
| 性能 | 受限 | 完全发挥 |
| 管理工具 | 图形界面 | 命令行为主 |
| 稳定性 | 一般 | 极高 |

#### MySQL 安装（P98~P99）

| 安装方式 | 说明 | 特点 |
|---|---|---|
| **RPM/YUM** | 包管理器安装 | 简单快捷，适合快速部署 |
| **源码编译** | 从源码编译 | 可定制参数，适合深度优化 |

**重要操作**：

```bash
# 查找已安装的 MySQL 包
rpm -qa | grep mysql

# 卸载旧版本
yum remove mysql-*

# 安装 MySQL 8.0（YUM 方式）
yum install mysql-community-server

# 启动服务
systemctl start mysqld
systemctl enable mysqld  # 开机自启
```

#### 远程连接配置（P100）

```bash
# 防火墙开放 3306 端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

```sql
-- 授权远程访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

---

### 3.2 字符集与 SQL 规范（P101~P103）

#### 字符集（P101）

**字符集层级**：

```
服务器级（server） → 数据库级（database） → 表级（table） → 列级（column）
```

优先级：列级 > 表级 > 数据库级 > 服务器级

**常见字符集对比**：

| 字符集 | 支持语言 | 每字符占字节 |
|---|---|---|
| **utf8mb4** | 全部 Unicode（含 Emoji） | 1~4 字节 |
| utf8（3字节版） | 中英日韩等 | 1~3 字节 |
| latin1 | 西欧语言 | 1 字节 |
| gbk | 中文简体 | 1~2 字节 |

> **重要提醒**：MySQL 的 `utf8` 不等于标准 UTF-8！MySQL 的 `utf8` 是阉割版，最多 3 字节，Emoji 存不了。**必须用 `utf8mb4`**。

**编码链路**：

```
客户端(UTF-8) → MySQL(character_set_client 接收)
  → character_set_connection 处理 → Collation 比较
  → character_set_results 返回 → 客户端显示
```

#### 比较规则（P102）

| 规则后缀 | 含义 | 特点 |
|---|---|---|
| `_ci` | Case Insensitive | 大小写不敏感（A=a） |
| `_cs` | Case Sensitive | 大小写敏感（A≠a） |
| `_bin` | Binary | 按字节值比较（最快） |

#### SQL 大小写规范与 sql_mode（P103）

**大小写敏感规则**：

| 参数 | 为 0（Linux 默认） | 为 1（Windows 默认） |
|---|---|---|
| `lower_case_table_names` | 库名表名大小写敏感 | 统一转为小写 |

> **注意**：`lower_case_table_names` 必须在 MySQL 初始化前设置好，之后修改会导致数据不一致！

**sql_mode 常见模式**：

| 模式 | 作用 |
|---|---|
| `ONLY_FULL_GROUP_BY` | SELECT 中未在 GROUP BY 中的列不能出现 |
| `STRICT_TRANS_TABLES` | 严格模式，不允许截断数据 |
| `NO_ZERO_IN_DATE` | 不允许日期中出现 0 |
| `NO_ZERO_DATE` | 不允许日期为 '0000-00-00' |

---

### 3.3 MySQL 目录结构（P104）

MySQL 安装后的目录结构（Linux 默认 `/var/lib/mysql/`）：

```
/var/lib/mysql/
├── ibdata1          # 系统表空间（共享）
├── ib_logfile0      # Redo 日志文件 0
├── ib_logfile1      # Redo 日志文件 1
├── mysql/           # 系统数据库
├── performance_schema/  # 性能监控数据库
├── mydb/            # 用户数据库
│   ├── employees.frm   # 表结构定义文件（MySQL 5.7）
│   └── employees.ibd   # 表数据和索引文件（独立表空间）
└── binlog.000001    # 二进制日志
```

---

### 3.4 用户管理（P105~P106）

#### 创建、修改、删除用户（P105）

```sql
-- 创建用户（推荐写法）
CREATE USER 'zhangsan'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'lisi'@'%' IDENTIFIED BY 'password123';  -- % 表示任意主机

-- 修改用户名
RENAME USER 'zhangsan'@'localhost' TO 'zhangsan_new'@'localhost';

-- 删除用户
DROP USER 'zhangsan'@'localhost';

-- 查看所有用户
SELECT user, host FROM mysql.user;
```

**@'localhost' vs @'%'**：
- `'localhost'`：只能本机连接
- `'%'`：任意主机连接（**生产环境谨慎使用！**）
- `'192.168.1.%'`：只允许特定网段连接

#### 密码管理（P106）

```sql
-- 修改当前用户密码
SET PASSWORD = 'newpass456';
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpass456';

-- 修改其他用户密码
SET PASSWORD FOR 'user1'@'localhost' = 'newpass789';
```

**MySQL 8 认证插件**：

| 插件 | 说明 |
|---|---|
| `caching_sha2_password` | MySQL 8 默认，更安全 |
| `mysql_native_password` | MySQL 5.7 默认，兼容性好 |

**忘记 root 密码的恢复方法**：
1. 停止 MySQL 服务
2. 以 `skip-grant-tables` 模式启动
3. 无密码登录后修改密码
4. 重启 MySQL 正常模式

---

### 3.5 权限管理（P107）

```sql
-- 授予权限
GRANT SELECT, INSERT ON mydb.employees TO 'user1'@'localhost';
GRANT ALL PRIVILEGES ON mydb.* TO 'admin'@'%';       -- 数据库级
GRANT ALL PRIVILEGES ON *.* TO 'root2'@'%';          -- 全局级（谨慎！）

-- 撤销权限
REVOKE INSERT ON mydb.employees FROM 'user1'@'localhost';

-- 查看权限
SHOW GRANTS FOR 'user1'@'localhost';
SHOW GRANTS;  -- 当前用户权限

-- 刷新权限
FLUSH PRIVILEGES;
```

**权限层级**：

```
全局级（*.*）→ 数据库级（mydb.*）→ 表级（mydb.employees）→ 列级
```

| 常见权限 | 说明 |
|---|---|
| SELECT / INSERT / UPDATE / DELETE | 基础 DML 权限 |
| CREATE / DROP | DDL 权限 |
| ALTER | 修改表结构 |
| INDEX | 创建/删除索引 |
| GRANT OPTION | 授予他人权限 |

> **最小权限原则**：只给够用的权限，不要多给。

---

### 3.6 角色管理（P108）

MySQL 8.0 引入角色功能——批量管理一组权限的便捷工具。

```sql
-- 创建角色
CREATE ROLE 'app_developer', 'app_read', 'app_write';

-- 给角色授权
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_write';
GRANT SELECT ON mydb.* TO 'app_read';

-- 把角色授予用户
GRANT 'app_developer' TO 'dev_user'@'localhost';

-- 激活角色（MySQL 8 需要手动激活）
SET DEFAULT ROLE 'app_developer' FOR 'dev_user'@'localhost';
```

**角色 vs 直接授权**：

| 对比 | 直接授权 | 角色 |
|---|---|---|
| 批量管理 | 每个用户单独授权 | 一组权限打包 |
| 权限调整 | 逐个用户修改 | 改角色，所有用户自动生效 |
| 临时禁用 | 需逐个回收 | 禁用角色即可 |
| MySQL 版本 | 5.7 及以下 | **8.0+** |

---

## 四、容易出错的地方

1. **utf8 vs utf8mb4**：MySQL 的 `utf8` 不是真正的 UTF-8，要用 `utf8mb4`
2. **lower_case_table_names**：必须在初始化前设置，之后改会丢数据
3. **@'%' 授权**：生产环境不要随便用 `%`，要限制 IP 段
4. **角色激活**：MySQL 8 中角色授予后需要手动激活才生效
5. **GRANT vs CREATE USER**：`GRANT` 也能创建用户，但推荐先 `CREATE USER` 再 `GRANT`

---

## 五、和其他知识点的联系

| 关联知识 | 关系 |
|---|---|
| MySQL 安装配置 | 本模块是学习环境基础 |
| 存储引擎 | 不同引擎的文件结构不同 |
| 权限与安全 | 用户权限是安全基础 |
| 主从复制 | 复制用户需要特定权限 |

---

## 六、面试可能怎么问

1. MySQL 的 `utf8` 和 `utf8mb4` 有什么区别？
2. 如何实现 MySQL 的最小权限原则？
3. MySQL 8.0 的角色功能有什么优势？
4. 生产环境如何限制数据库的远程访问？

---

## 七、总结

- 生产环境用 Linux，字符集用 `utf8mb4`
- 用户管理遵循最小权限原则
- MySQL 8.0 的角色功能简化了权限管理
- `lower_case_table_names` 要在初始化前设置

---

## AI辅助思考

### 这个知识点真正重要的地方

本模块看起来是"环境搭建"，但实际包含两个面试高频点：**字符集**和**权限管理**。字符集的 `utf8` vs `utf8mb4` 是经典陷阱题，权限管理的最小权限原则是安全面试必答内容。

### 和哪些知识点有关

- **字符集** → 关联到数据存储、索引排序（Collation 影响索引使用）
- **用户权限** → 关联到主从复制（需要创建复制用户）、项目部署
- **sql_mode** → 关联到 SQL 编写规范、GROUP BY 语法

### 初学者容易误解什么

1. 以为 MySQL 的 `utf8` 就是标准 UTF-8（实际上缺了 4 字节字符）
2. 以为 `GRANT` 就够了，忽略 `FLUSH PRIVILEGES`
3. 以为角色授予就自动生效（MySQL 8 需要手动激活）

### 实际项目中可能怎么用

- 项目部署时创建专用数据库用户，不使用 root
- 按模块授权：读写分离时，读库只给 SELECT 权限
- 用角色管理开发团队的不同权限级别

### 下一步可以学什么

1. MySQL 逻辑架构（连接层→服务层→引擎层）
2. 存储引擎（InnoDB vs MyISAM）
3. 索引原理（B+树）

### AI 给我的学习建议

字符集和权限这两块内容看起来简单，但面试经常考细节。建议你重点记住 `utf8mb4` 这个坑，以及权限层级的四个级别。Linux 安装部分如果已经有环境了就不用太纠结，把重心放在原理理解上。
