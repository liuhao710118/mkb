下面给你一份**偏实战、可直接落地执行的 MySQL 学习计划**。

我会像 Oracle 那套一样，按 **“学习点 + 该阶段你能做到什么”** 来组织，并给你一个 **6～8 周可执行时间表**。

------

## 一、MySQL 学习总览（先建立地图）

### MySQL 的核心特点（和 Oracle 对照更容易理解）

- **开源关系型数据库**，轻量、易部署、社区活跃

- 默认引擎 **InnoDB**：支持事务、行锁、外键、崩溃恢复

- 架构更简单：

  -   无多实例共享存储（不像 Oracle RAC）

  -   单机为主，高可用靠 **主从复制 / MGR / 第三方方案**

- 适用场景：Web 应用、业务系统、中台服务、数据分析前置库

------

## 二、MySQL 学习模块 & 关键学习点

------

## 0. 先建立全局认知（别急着敲命令）

### 学习点

- MySQL 在数据库生态中的位置

- 常见版本选择：

  -   **MySQL 8.0（强烈推荐）**

  -   了解 Percona / MariaDB 与官方 MySQL 的区别（概念即可）

- 架构直觉：

  -   客户端 → 连接层 → SQL 层 → 存储引擎层 → 磁盘

- 两种常见部署形态：

  -   单机

  -   主从（后面再学）

### 你会做的

- 能说出：

  -   InnoDB 和 MyISAM 的最大区别（事务 / 锁 / 崩溃恢复）

  -   MySQL 为什么不适合做“重型分析型数据库”

------

## 1. 环境准备（越早动手越好）

### 学习点

- 安装方式选择：

  -   **Linux（推荐）**：CentOS / Rocky / Ubuntu

  -   Windows：MySQL Installer（适合初学）

- 关键目录与文件：

  -   `datadir`

  -   配置文件：`my.cnf / my.ini`

  -   错误日志、慢查询日志位置

- 核心工具：

  -   `mysql`（命令行客户端）

  -   `mysqldump`

  -   MySQL Workbench / Navicat / DBeaver（任选一个）

### 你会做的

- 完成一次 MySQL 安装

- 会用：

  ```
  mysql -uroot -p
  show databases;
  ```

- 会启停服务：

  -   Linux：`systemctl start/stop/status mysqld`

  -   Windows：服务面板

------

## 2. SQL 基础（这是地基）

### 2.1 查询

学习点

- `SELECT ... FROM ... WHERE / ORDER BY / LIMIT`

- 过滤：`=,<>,NULL（IS NULL / IS NOT NULL）`

- 范围：`BETWEEN / IN / LIKE`

- 聚合：`GROUP BY / HAVING`

- 连接（非常重要）：

  -   `INNER JOIN / LEFT JOIN / RIGHT JOIN`

- 子查询：`IN / EXISTS / 派生表`

- CTE（MySQL 8.0. ：`WITH`

练习

- 建立 `user / order / order_item`表

- 写查询：

  -   从未下单的用户

  -   每个用户的订单总金额

  -   金额大于平均订单金额的订单

------

### 2.2 DML / DDL / 事务

学习点

- DML：`INSERT / UPDATE / DELETE / REPLACE`

- DDL：`CREATE TABLE / ALTER / DROP / TRUNCATE`

- 约束：

  -   `PRIMARY KEY / UNIQUE / NOT NULL / FOREIGN KEY`

- 自增：`AUTO_INCREMENT`

- 事务：

  -   `START TRANSACTION / COMMIT / ROLLBACK`

  -   InnoDB 默认 **可重复读（RR）**

- 存储引擎对比：`SHOW TABLE STATUS`

练习

- 模拟转账事务：

  -   扣 A 余额

  -   加 B 余额

  -   成功 commit，失败 rollback

------

## 3. 存储引擎 & 事务机制（MySQL 的核心）

### 学习点

- InnoDB 核心机制：

  -   事务 ACID

  -   undo log（回滚 + 一致性读）

  -   redo log（crash-safe）

- 锁：

  -   行锁（InnoDB）

  -   表锁（MyISAM）

  -   间隙锁（RR 下防止幻读）

- 隔离级别：

  -   READ UNCOMMITTED

  -   READ COMMITTED

  -   REPEATABLE READ（默认）

  -   SERIALIZABLE

练习

- 两个终端：

  -   一个更新某行不提交

  -   另一个更新同一行，观察阻塞

- 用：

  ```
  SHOW ENGINE INNODB STATUS\G
  ```

------

## 4. 用户 / 权限 / 安全

### 学习点

- 用户管理：

  ```
  CREATE USER 'app'@'%' IDENTIFIED BY 'pwd';
  GRANT SELECT,INSERT ON db.* TO 'app'@'%';
  ```

- 权限分类：

  -   全局权限

  -   库级、表级权限

- 删除/回收权限：`DROP USER / REVOKE`

练习

- 创建：

  -   一个只读账号

  -   一个业务读写账号

- 验证权限生效

------

## 5. 索引与性能（重点中的重点）

### 学习点

- B+Tree 索引（直觉即可）

- 索引类型：

  -   普通索引

  -   唯一索引

  -   联合索引（最左前缀原则）

- 索引失效常见原因：

  -   对列使用函数

  -   隐式类型转换

  -   `LIKE '%xxx'`

- 执行计划：

  ```
  EXPLAIN SELECT ...
  ```

  关注：`type / key / rows / Extra`

练习

- 给查询加索引前后对比 `EXPLAIN`

- 验证联合索引的“最左匹配”

------

## 6. 备份恢复（生产必备）

### 学习点

- 逻辑备份：

  -   `mysqldump`（全量）

- 物理备份（了解）：

  -   Xtrabackup（Percona）

- 恢复流程：

  -   dump → source

- binlog（概念）：

  -   用于增量恢复、主从同步

练习

- 用 `mysqldump`备份一个库

- 删除数据 → 用备份恢复

------

## 7. 主从复制 & 高可用（面试常考）

### 学习点（先概念）

- 主从复制原理：

  -   binlog → relay log → SQL thread

- 用途：

  -   读写分离

  -   容灾

- 高可用方案（了解）：

  -   MHA

  -   Orchestrator

  -   MySQL InnoDB Cluster（MGR）

练习

- 搭一套最简单的一主一从

------

## 三、6～8 周可执行学习计划（推荐节奏）

| 周   | 主题                        | 产出                   |
| ---- | --------------------------- | ---------------------- |
| W1   | 安装 + SQL 查询 + 聚合      | 能写复杂查询           |
| W2   | DML/DDL/事务/隔离级别       | 会写事务代码           |
| W3   | 存储引擎 / 锁 / undo / redo | 能解释 RR 为什么防幻读 |
| W4   | 用户权限 + 索引设计         | 至少 3 个索引优化案例  |
| W5   | EXPLAIN / 性能分析          | 会看执行计划           |
| W6   | mysqldump / binlog 概念     | 一套备份恢复流程       |
| W7   | 主从复制搭建                | 一主一从可运行         |
| W8   | 综合项目（订单系统）        | 完整建库 + SQL + 优化  |

------

## 四、你可以直接告诉我 3 件事，我帮你定制

1. 你的系统：**Windows / Linux / macOS**

2. 内存情况：8G / 16G / 更大

3. 学习目标：

   -    ✅ 后端开发（写 SQL / 会用就行）

   -    ✅ 初级 DBA（运维 / 备份 / 主从）

   -    ✅ 面试突击 / 跳槽

   -    ✅ 考认证（如 MySQL OCP）

我可以基于你的答案，给你一份 **“每天具体做什么 + 练习表结构 + 检查点”** 的执行版计划。