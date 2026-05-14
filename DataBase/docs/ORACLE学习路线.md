下面是一份**偏“生产可用”的 Oracle 数据库学习路线**，我把知识点拆成 **学习模块 + 关键学习点 + 你该会做什么（练习）**。你可以按模块顺序推进（1→6），每一块打扎实再往下走。

------

## 0) 先建立全局认知（别急着敲命令）

学习点

- Oracle 在“数据库界”的定位：**关系型 + 强事务 + 行级锁 + 共享存储架构**（通常与企业核心系统绑定）

- 典型部署形态：

  -   **单机（Single Instance）**

  -   **RAC（多节点集群共享存储）**

  -   **物理备库 Data Guard**

- 核心进程/组件名字先混脸熟（不必死背）：**SGA/PGA、后台进程（PMON/SMON/DBWn/LGWR/CKPT/ARCn）、Listener、Oracle Home、Instance vs Database**

你会做的练习

- 画一张图：**SQL 从客户端 → Listener → Server Process → SGA → Disk（redo/data）** 的简化路径

- 区分清楚：`Oracle实例(内存+进程)`≠ `Oracle数据库(数据文件/控制文件/日志)`

------

## 1) 环境准备（越早动手越稳）

学习点

- 安装方式选择

  -   **Linux/Windows 本地安装**（最推荐：Oracle Linux / RHEL 系 + RPM或runInstaller）

  -   **VirtualBox/VMware**里装 Oracle Linux 再装 DB（更像生产）

- 关键目录与文件体系

  -   `ORACLE_BASE / ORACLE_HOME`

  -   数据文件、控制文件、redo log、参数文件（`spfile/pfile`）

  -   `tnsnames.ora / listener.ora`

- 两个工具栈：

  -   服务端：`sqlplus`（必须会）+ 可选 `rman`

  -   客户端：**SQL Developer**（图形化更友好）

你会做的练习

- 完成一次“干净安装 + 建库（DBCA）+ 能连上 sqlplus/SQLDev”

- 会启停：`startup / shutdown immediate`

- 会查：`select * from v$instance; select * from v$database;`

- 会用 `lsnrctl status`看监听状态

------

## 2) SQL 基础（这是地基，别跳）

### 2.1 查询与集合

学习点

- `SELECT ... FROM ... WHERE / ORDER BY / DISTINCT`

- 过滤：`=,<>,NULL`处理（`IS NULL / IS NOT NULL`，以及 `NVL / COALESCE`）

- 范围/模式：`BETWEEN`、`IN`、`LIKE`

- 连接（必会）：

  -   `INNER JOIN / LEFT JOIN / RIGHT JOIN`（标准写法优先）

  -   旧式 `(+)`能看懂但别常用

- 聚合：`GROUP BY / HAVING`，常见聚合函数 `COUNT/SUM/AVG/MAX/MIN`

- 子查询：标量子查询、IN/EXISTS、`WITH`（CTE）

你会做的练习

- 自己造一张 `emp/dept`风格小表（或 HR 示例 schema）

- 写查询：列出“没有员工的部门”“每个部门薪水最高的员工”“薪水高于部门平均的人”

### 2.2 DML / DDL / TCL（这里开始进入“事务世界观”）

学习点

- DML：`INSERT / UPDATE / DELETE / MERGE`

- DDL：`CREATE TABLE / ALTER / DROP`、`TRUNCATE`（知道它不等同于DELETE）

- 约束：`PRIMARY KEY / FOREIGN KEY / UNIQUE / NOT NULL / CHECK`

- 序列：`SEQUENCE`（Oracle 里自增主键的经典做法；12c+也可用 identity column）

- 事务控制：`COMMIT / ROLLBACK / SAVEPOINT`

- 关键意识：Oracle 默认是**语句级读一致**，DML 会产生 redo/undo

你会做的练习

- 做一个小业务：订单头/订单行（`order_header / order_line`），建 PK/FK/UK/NOT NULL

- 写几个事务场景：插入头+行 → commit；插入失败 → rollback；用 savepoint 做部分回滚

------

## 3) Oracle 核心机制：事务 / UNDO / REDO / SCN（理解后很多现象就通了）

学习点（重点：形成因果链）

- **事务**：begin → DML → commit/rollback；未提交对其他会话不可见（按隔离级别机制）

- **UNDO（回滚段）**：用于回滚 + 一致性读（读不阻塞写，写不阻塞读，靠 undo 实现）

- **REDO / Online Redo Logs**：保证持久性；理解 `lgwr/ckpt/archiver`的关系

- **SCN（System Change Number）**：时间线/一致性点的概念（后面 RMAN/恢复会用到）

- **锁的粒度**：行级锁为主；不同操作对锁的影响；`FOR UPDATE`；避免长时间持有锁

- `deadlock`现象识别（不是背诵，而是知道怎么排查思路）

你会做的练习

- 开两个会话：一个更新某行不提交；另一个更新同一行——观察阻塞/等待现象

- 用 `V$LOCK / V$SESSION / V$TRANSACTION`（只要求会读线索）理解“谁锁谁”

- 实验：`ROLLBACK`能把 UPDATE 还原；`COMMIT`之后不能

------

## 4) 对象模型：表空间/数据文件、表类型、索引、分区（Oracle 的“工程面”）

### 4.1 存储抽象（必须懂）

学习点

- 层级：`数据库 → 表空间(tablespace) → 数据文件(datafile)`

- SYSTEM/SYSAUX/UNDOTBS/TEMP/USERS 这些默认表空间分别干嘛

- 临时表空间与临时段（`TEMP`不是放“临时表数据”那么简单，排序/哈希会吃它）

练习

- 创建一个业务表空间：

```
CREATE TABLESPACE ts_app
  DATAFILE '/u01/oradata/ORCL/ts_app01.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE 2G;
```

- 把用户默认表空间指向它

### 4.2 表设计关键点

学习点

- 普通堆表、临时表（`ON COMMIT DELETE ROWS / PRESERVE ROWS`）

- 约束与索引的关系：PK 自动建唯一索引；FK 对性能/锁的潜在影响

- 数据类型选型：`NUMBER/VARCHAR2/DATE/TIMESTAMP/CHAR/CLOB/BLOB`（别滥用 CHAR）

### 4.3 索引（非常关键）

学习点

- B-Tree 索引原理直觉：等值/范围快；不适合低选择性 or 过度写入热点

- 唯一索引 / 非唯一索引

- 函数索引：`upper(col)`、`col+0`这类写法会废掉索引（要理解为什么）

- 复合索引：前导列规则、索引跳跃扫描（概念即可）

- 何时可能用不到索引：隐式类型转换、`NULL`相关、对列做函数包裹、`<>`之类

练习

- 为查询场景建索引前后，用 `EXPLAIN PLAN`看执行计划变化（先学会看“全表扫描 vs INDEX RANGE SCAN”）

### 4.4 分区（当表变“大”时的必修课）

学习点（概念优先）

- Range / List / Hash 分区

- 分区裁剪（Partition Pruning）直觉

- Local vs Global 索引（概念：local 和分区对齐更利于维护）

------

## 5) PL/SQL（写“过程化逻辑”的标准方式）

学习点（不建议一上来精读语法书，建议边用边学）

- 匿名块结构：`DECLARE/BEGIN/EXCEPTION/END`

- 变量声明、赋值（`:=`）、`SELECT ... INTO`

- 控制流：`IF/CASE`、`LOOP/WHILE/FOR`

- 游标：`CURSOR / FETCH / EXIT WHEN`（以及 `FOR rec IN (sql) LOOP`这种更常用的）

- 异常处理：`OTHERS`、`SQLCODE/SQLERRM`、记录日志表

- 存储过程 / 函数 / 包（Package）：规范拆分：spec 与 body

- 绑定变量意识（在 PLSQL 里通常不是大问题，但在 Java/其他客户端里很关键）

你会做的练习

- 写一个过程：输入部门编号，输出该部门员工清单（游标循环）

- 写一个审计触发器（理解触发器代价与风险，生产里别滥用）

------

## 6) 用户/权限/角色/安全（生产红线区）

学习点

- 用户：`CREATE USER ... IDENTIFIED BY ... DEFAULT TABLESPACE ...`

- 权限：`GRANT/REVOKE`：系统权限 vs 对象权限

- 角色：`CREATE ROLE / GRANT role TO user`

- Schema（模式）= 用户（在Oracle里很多人这么说）：`scott.emp`这种前缀就是 schema

- 网络与安全概念：密码文件、`sqlnet.ora`、listener 密码/访问控制（点到为止）

- 审计：`AUDIT`（概念层面）+ 统一审计（若你学到较新版本再深入）

你会做的练习

- 建一个 app 用户（不给他 DBA），只给 `CREATE SESSION + 业务表权限`

- 用角色把权限打包，验证“角色生效需要登录/SET ROLE”

------

## 7) 备份恢复 & 高可用（Oracle 的灵魂卖点之一）

### 7.1 日志/归档基础

学习点

- `ARCHIVELOG`vs `NOARCHIVELOG`（生产几乎必 ARCHIVELOG）

- 归档日志去哪：`LOG_ARCHIVE_DEST`

### 7.2 RMAN（必须练）

学习点（把“流程”练熟比背参数重要）

- RMAN 基本概念：通道（channel）、备份集/备份片、控制文件自动备份

- 常用操作：

  -   `backup database plus archivelog`

  -   `list backup`

  -   不完全恢复场景：到某个时间点（`set until time/SCN`）

- 至少理解两种恢复：

  -   实例崩溃恢复（自动：前滚 redo + 回滚 undo）

  -   介质恢复（需要备份/RMAN）

你会做的练习（最好虚拟机快照做）

1. 开启归档

2. 做一次全备（RMAN）

3. “人为破坏”（删一个数据文件）→ 用 RMAN 做 restore + recover

4. 验证数据库能 open

### 7.3 Data Guard 概念（先当“知识框架”）

- 物理备库：redo apply；主备角色切换（switchover/failover）概念

- 先理解：为什么企业要它（RPO/RTO、容灾、只读报表分流）

------

## 8) 性能诊断：从“会跑”到“跑得稳”

学习点（偏方法论）

- 执行计划：`EXPLAIN PLAN FOR ...`+ `DBMS_XPLAN.DISPLAY*`

  关键看：全表扫描、索引范围扫描、连接方式（NESTED LOOPS / HASH JOIN / SORT MERGE）

- 统计信息：`DBMS_STATS.GATHER_TABLE_STATS`（优化器靠它做判断）

- AWR/ASH（概念）：当你遇到“慢”的时候，先问：是**单次慢**还是**整体扛不住**

- 常见性能杀手：

  -   隐式类型转换

  -   对列包函数

  -   返回太多行却不全表扫描不可避

  -   热点块/频繁 commit 碎片化

  -   缺失索引或索引设计不当

  -   临时表空间爆（大排序/大量 hash）

练习

- 造一张大一点的测试表（十万～百万行即可），跑几种查询：

  -   带条件走索引 vs 不走索引

  -   join 两种写法对比计划（IN vs EXISTS 在某些场景下等价，但你要能从计划看本质）

------

## 9) 进阶/扩展（按需再深挖）

- 多租户（12c+ 的 CDB/PDB）：container 概念、PDB 的 open/close、切换

- Data Pump：`expdp / impdp`（迁移/克隆必备）

- SQL 调优工具链：`SQL Trace / TKPROF`（知道存在即可）

- Oracle 与开发侧：JDBC/OCI 连接串、连接池、大事务危害、批量处理（array fetch/bulk）

------

# 一个“可执行”的 6 周学习节奏（你可以按自己时间缩放）

> 目标：能独立安装、建用户/表空间、写 SQL+PLSQL、做 RMAN 全备与简单恢复、能看执行计划。

| 周   | 主题                                 | 产出物                                |
| ---- | ------------------------------------ | ------------------------------------- |
| W1   | 安装+实例/监听/基本对象              | 能连库；画出 Instance↔Database 关系图 |
| W2   | SQL：查询+聚合+join+子查询           | 做完一组查询作业（含 EXISTS/CTE）     |
| W3   | DML/DDL/约束/事务/undo/redo 直觉     | 两会话锁演示；事务实验报告            |
| W4   | 表空间/索引/分区概念 + EXPLAIN PLAN  | 至少 3 个“加索引前后”对比案例         |
| W5   | PL/SQL：过程/函数/包 + 异常处理      | 写一个小模块：批量处理某业务逻辑      |
| W6   | 用户权限 + RMAN 全备/restore/recover | 一套可复现的备份恢复演练文档          |

------

如果你愿意，我可以按你的情况把它“定制化”：

1. 你用的是 **Windows 还是 Linux**（Oracle Linux 最佳）？

2. 你的机器内存大概多少（8G/16G）？

3. 你学 Oracle 的目标是：**开发（写SQL/PLSQL）**、还是 **运维DBA**、还是 **备考 OCP**？

你回这三点，我就能把上面大纲细化成一份给你电脑可执行的**安装版本选择 + 每天任务清单 + 练习表结构与题目**。