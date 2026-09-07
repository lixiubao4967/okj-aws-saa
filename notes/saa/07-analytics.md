# Week 5 收尾：分析服务速览 — Redshift / Athena / Glue

> 学习日期：2026-09-07（P1 收尾）
> ⏳ **状态：内容已过，随堂测验未做 → 回家先做文末 5 题再进 VPC**
>
> SAA 里这三个权重不高，但**几乎必出 1-2 题选型题**。只考「该用哪个」，不考深度调优。

---

## 1. 大前提：OLTP vs OLAP

| | **OLTP**（联机事务处理） | **OLAP**（联机分析处理） |
|---|---|---|
| 干什么 | 处理**业务交易** | 做**分析报表** |
| 典型查询 | 「查用户 12345 的订单」 | 「统计过去 3 年各地区销售额同比」 |
| 特征 | 高并发、小数据量、读写混合 | 低并发、**扫描海量数据**、几乎只读 |
| AWS 服务 | RDS / Aurora / DynamoDB | **Redshift / Athena** |

### 为什么 RDS 做不了分析：存储方式不同

```
行式存储（RDS/Aurora）——按行存放
[id=1, name=张, region=东京, amount=100]
[id=2, name=李, region=大阪, amount=200]
→ 只想统计 amount 总和？也得把 name、region 全读出来。浪费 I/O。

列式存储（Redshift）——按列存放
id:     [1, 2, 3, ...]
name:   [张, 李, 王, ...]
amount: [100, 200, 300, ...]
→ 统计 amount？只读这一列。I/O 省 90%+，且同列数据类型相同、压缩率极高。
```

> 🔑 **记忆锚点：Redshift 是「仓库」，RDS 是「柜台」。**
> 仓库适合盘点全部库存，柜台适合快速取一件货。

⚠️ **Redshift 绝对不适合 OLTP**。题目出现「高并发单行读写」「作为应用主数据库」而选项有 Redshift → 一定是干扰项。

---

## 2. Redshift：托管数据仓库

**定位**：PB 级数据仓库，跑复杂 SQL 分析。快的两大原因：**列式存储 + MPP（大规模并行处理）**。

| 要点 | 内容 |
|---|---|
| 架构 | 1 个 **Leader Node**（接收查询、制定计划）+ N 个 **Compute Node**（并行执行）。加节点 = 提升性能 |
| **不是实时** | 数据要先用 `COPY` 从 S3 加载进来。题目强调 `real-time` / `sub-second` → 不是 Redshift，考虑 Kinesis |
| **Redshift Spectrum** ⭐ | **直接查 S3 数据，不用加载进集群**。热数据在集群、冷数据留 S3，一条 SQL 查两边 |
| **Redshift Serverless** | 不管集群容量，按用量付费 |
| 备份 | 自动快照到 S3，可跨区复制 |
| 加密 | KMS，**创建集群时**启用 |

### 关键词映射

| 题目出现 | 答案 |
|---|---|
| `columnar` / `petabyte-scale` / `complex joins` / `BI dashboard` | **Redshift** |
| `query data in S3 without loading it` + 已在用 Redshift | **Redshift Spectrum** |
| `unpredictable workload` / `no cluster management` | **Redshift Serverless** |

---

## 3. Athena：S3 上的无服务器 SQL

**定位**：直接用标准 SQL 查 S3 里的文件，**零基础设施**。

- **完全 serverless**：没有集群、没有节点，不查不花钱
- **计费模型** ⭐：**按扫描的数据量收费**（约 $5 / TB）→ 这是所有 Athena 优化题的根源
- **支持格式**：CSV、JSON、Parquet、ORC、Avro

### ⭐ 最高频考点：如何降低 Athena 成本 / 提升性能

| 优化手段 | 原理 |
|---|---|
| ⭐ **转成列式格式（Parquet / ORC）** | 只扫需要的列，扫描量降 30-90%，**通常是最优答案** |
| **压缩数据**（Snappy / gzip） | 文件变小 → 扫描字节数变少 |
| **按分区组织**（`s3://bucket/year=2026/month=09/`） | 查 9 月只扫 9 月目录，不碰其他分区 |

### 典型场景

查 **CloudTrail 日志 / VPC Flow Logs / ALB 访问日志 / S3 访问日志** —— 这些日志天生就在 S3，Athena 是标准答案。

### Athena vs Redshift 分界线

> **Athena** = 偶尔查一下，不想建基础设施
> **Redshift** = 持续高频复杂分析，愿意维护集群换性能

信号词：`ad-hoc queries`、`least operational overhead`、`serverless`、`analyze logs in S3` → **Athena**

> 💡 `least operational overhead` 在 SAA 里出现频率极高，几乎等于「选 serverless 那个」。

---

## 4. Glue：无服务器 ETL + 数据目录

Glue 有**两个几乎独立**的功能，别混：

### ① Glue ETL —— 无服务器数据转换

```
S3(原始 CSV) → Glue Job(转换/清洗/转 Parquet) → S3(整理后) 或 Redshift
```
- 底层跑 Spark，但不用管服务器
- 信号：`serverless ETL`、`transform data`、`convert to Parquet`、`no infrastructure to manage`

### ② Glue Data Catalog —— 元数据中心 ⭐

- 存的是「**表结构定义**」（某 S3 路径有哪些列、什么类型），**不存数据本身**
- **Glue Crawler**：自动扫描 S3 → 推断 schema → 写进 Catalog
- 为什么重要：**Athena、Redshift Spectrum、EMR 都从 Glue Catalog 读表定义**

```
   Glue Crawler 扫描 S3 → 生成表定义 → Glue Data Catalog
                                            ↓
              Athena / Redshift Spectrum / EMR 都来这里查表结构
```

- 信号：`central metadata repository`、`schema discovery`、`automatically infer schema` → **Glue Data Catalog / Crawler**

---

## 5. 配角服务（各记一句话）

| 服务 | 一句话 | 信号词 |
|---|---|---|
| **QuickSight** | 托管 BI 可视化（图表看板） | `dashboard`、`visualize`、`business intelligence` |
| **EMR** | 托管 Hadoop / Spark 集群 | `Hadoop`、`Spark`、`HBase`、需**自定义框架** |
| **Lake Formation** | S3 上建数据湖 + 细粒度权限（列级/行级） | `data lake`、`fine-grained access control` |
| **Kinesis Data Firehose** | 流数据自动送进 S3 / Redshift / OpenSearch | `near real-time`、`load streaming data` |
| **OpenSearch** | 日志搜索 + 分析（原 Elasticsearch） | `full-text search`、`log analytics dashboard` |

---

## 6. ⭐ 选型决策树（考试直接套用）

```
数据在 S3，只想偶尔用 SQL 查一下，不想管基础设施
    → Athena

需要持续跑复杂分析 / BI 报表 / PB 级 join
    → Redshift

已在用 Redshift，还想查 S3 冷数据且不加载
    → Redshift Spectrum

需要清洗/转换数据格式，无服务器
    → Glue ETL

需要自动发现 S3 数据的 schema 给别的服务用
    → Glue Crawler + Data Catalog

需要做图表看板给业务看
    → QuickSight

要跑 Hadoop / Spark 自定义作业
    → EMR

流数据要近实时进 S3/Redshift
    → Kinesis Data Firehose
```

---

## 7. 待完成测验（回家先做这 5 题）

**Q1.** 公司在 S3 存了 5 年的 ALB 访问日志（CSV），安全团队偶尔需要用 SQL 做临时排查。要求**运维开销最小**。
A) 导入 Redshift　B) Athena　C) 起 EMR 集群　D) 导入 RDS

**Q2.** 已在用 Athena 查 S3 日志，但**成本过高**。最有效的优化是？
A) 换更大实例　B) 把数据转成 Parquet 格式并压缩　C) 加 Redshift 集群　D) 开启 S3 Transfer Acceleration

**Q3.** 团队已有 Redshift 集群跑日常报表。现在要**偶尔关联查询 S3 里的历史归档数据**，但不想把它们加载进集群。
A) Redshift Spectrum　B) COPY 命令导入　C) Athena 联邦查询　D) Glue ETL 搬进集群

**Q4.** S3 里有大量结构未知的 JSON 文件，需要**自动识别 schema** 并让 Athena 能直接查。
A) 手写 DDL 建表　B) Glue Crawler + Data Catalog　C) Lambda 解析　D) DynamoDB 存元数据

**Q5.** 电商需要一个数据仓库，跑 PB 级复杂 join 的销售分析，业务方还要看可视化看板。
A) Aurora + QuickSight　B) DynamoDB + Athena　C) Redshift + QuickSight　D) ElastiCache + EMR
