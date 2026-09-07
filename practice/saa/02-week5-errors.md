# Week 5 数据库 — 错题记录

## 成绩趋势

| 日期 | 模块 | 得分 |
|------|------|------|
| — | Aurora | 2/5（40%） |
| 2026-09-06 | DynamoDB | **5/6（83%）** ⬆️ |
| 2026-09-07 | ElastiCache | **5/5（100%）** ⬆️ |
| 2026-09-07 | 分析服务（Redshift/Athena/Glue） | **0/5（0%）** 🚨 |
| 2026-09-07 | 分析服务 **重测** | **4/5（80%）** ⬆️ P1 收工 |

---

## Aurora 练习：2/5（40%）

---

## 错题 1：Aurora 误删数据恢复

**场景：** 误删关键表，需尽快恢复且不影响现有数据库可用性

**我的答案：** A) Aurora Backtrack  
**正确答案：** B) Point-in-Time Recovery（PITR）恢复到新集群

### 错因分析
- 只记住了 Backtrack 速度快，忽略了它的副作用
- 没有注意"不影响现有数据库"这个关键约束

### 要点
- **Backtrack** = 整个库回退，后续所有正常数据也会丢失 + 短暂不可用
- **PITR** = 恢复到新集群，原库完全不受影响
- 题目出现"不影响现有数据库" → 排除 Backtrack，选 PITR

---

## 错题 2：Aurora 主实例故障恢复

**场景：** 主实例故障后 30 秒内自动恢复

**我的答案：** D) Aurora Global Database  
**正确答案：** B) Aurora Replica + 配置故障转移优先级

### 错因分析
- 过度设计：把"单实例故障"当成了"Region 故障"
- 没注意 Global Database 的 failover 需要手动触发

### 要点
- 单实例故障 → **Aurora Replica**（同 Region，自动 Failover，< 30 秒）
- Region 故障 → **Global Database**（跨 Region，手动提升，RTO < 1 分钟）
- 考试原则：**选满足需求的最简方案**，不要过度设计

---

## 错题 3：Aurora 读取瓶颈 + 热点查询

**场景：** 已有 3 个读副本但性能仍不足，大量重复热点查询

**我的答案：** A) 增加读副本至 15 个  
**正确答案：** B) ElastiCache（Redis）缓存热点查询

### 错因分析
- 没抓住"重复热点查询"这个关键线索
- 加副本是水平扩展思路，但重复查询应该用缓存拦截

### 要点
- 重复/热点查询 → **ElastiCache**（缓存层，微秒级返回）
- 多样化读取负载 → **增加读副本**（水平扩展）
- 缓存命中后请求根本不到数据库 → 从根本上解放数据库压力

---

# DynamoDB 练习：5/6（83%）— 2026-09-06

答对：GSI 选型、GSI 反噬主表、Global Tables 多活、TTL、DAX
答错：RCU 计算

---

## 错题 4：RCU 计算（最终一致读）

**场景：** 每秒读取 100 个 item，每个 item **6 KB**，可接受**最终一致性**，需要多少 RCU？

**我的答案：** B) 150
**正确答案：** A) 100

### 错因分析

我的算法是 `6 ÷ 4 = 1.5` → `1.5 × 100 = 150`，连犯两个错，正好抵消了一半：

1. ❌ **漏了向上取整** —— 容量单位是**离散的块**，不存在 1.5 RCU
2. ❌ **漏了最终一致读除以 2**

### 正确的固定三步

```
① 向上取整成块   读：ceil(size ÷ 4KB)   写：ceil(size ÷ 1KB)
② 乘以每秒次数
③ 最终一致读 → ÷ 2      事务 → × 2
```

本题：`ceil(6÷4)=2` → `2×100=200` → `200÷2=` **100 RCU**

### 要点

- 🚨 **绝对不要先做除法再乘**。读 5KB 和读 8KB 收费一样（都是 2 RCU）
- 强一致读是"全价"，**最终一致读打对折**，事务是"双倍价"
- 自测已通过：20 次/秒 × 3.5KB 写 = **80 WCU**；50 次/秒 × 10KB 强一致读 = **150 RCU**

---

## 本轮暴露的模式（与 Aurora 轮对比）

| 毛病 | Aurora 轮 | DynamoDB 轮 |
|------|-----------|-------------|
| 过度设计（选最贵最复杂的） | ❌ 错 2 题 | ✅ 已改正（Q3/Q4 都选了最简方案） |
| 漏读约束关键词 | ❌ 错 1 题 | ✅ 已改正 |
| **计算题步骤跳步** | — | ❌ **新暴露** |

> **下轮重点：遇到计算题一律写出三步，不心算。**

---

# 分析服务练习：0/5（0%）— 2026-09-07 🚨

答错：全部（Athena/Redshift 选型、Athena 降本、Spectrum、Glue Catalog、DynamoDB 能力边界）

> 与 ElastiCache 5/5 落差极大。**性质和前两轮不同**：前两轮是审题/方法问题，这轮是**服务能力边界记错**（知识空洞）。

## 错题 5：Athena vs Redshift 选型

**场景：** S3 存 5 年 ALB 日志，安全团队**偶尔**用 SQL 临时排查，要求**运维开销最小**

**我的答案：** A) 导入 Redshift　**正确答案：** B) Athena

### 错因
选了预置集群方案。Redshift 要选节点、管容量、7×24 付费，为「偶尔查」养集群是运维+成本双输。

### 要点
🔑 **Athena vs Redshift 唯一分水岭 = 查询频率**
- 偶尔 / ad-hoc / 临时排查 → **Athena**（Serverless，零运维）
- 持续 / 日常报表 / BI → **Redshift**

---

## 错题 6：Athena 成本优化

**场景：** Athena 查 S3 日志成本过高，最有效优化？

**我的答案：** D) S3 Transfer Acceleration　**正确答案：** B) 转 Parquet + 压缩

### 错因
把「传输加速」当成了「查询降本」。Transfer Acceleration 是走 CloudFront 边缘解决**跨洲上传/下载慢**，和扫描量无关，一分钱不省。

### 要点
```
Athena 费用 = 扫描的数据量 × $5/TB
```
不按时间、不按实例、不按次数 —— **只按扫了多少字节**。降本三招（让它少扫）：
1. **列式格式** Parquet / ORC —— 只读用到的列，省 30–90%
2. **压缩** Snappy / gzip
3. **分区** 按 year/month/day 建前缀

---

## 错题 7：Redshift Spectrum

**场景：** **已有 Redshift 集群**，要偶尔关联查询 S3 归档数据，**不想加载进集群**

**我的答案：** C) Athena 联邦查询　**正确答案：** A) Redshift Spectrum

### 错因 —— 概念性误解
| | 干什么 |
|---|---|
| **Athena 查 S3** | Athena 的**本职工作**，不叫联邦查询 |
| **Athena Federated Query** | 查 **S3 以外**的源：RDS / DynamoDB / Redshift / CloudWatch Logs（靠 Lambda 连接器） |

「联邦查询查 S3」这个说法本身不成立。

### 要点
- 题干「已有 Redshift」+「不加载进集群」同时出现 = **Spectrum** 标准题面
- Spectrum 能让 S3 数据和**集群内表做 join**，且不引入新工具

---

## 错题 8：Glue Crawler + Data Catalog

**场景：** S3 大量**结构未知**的 JSON，需**自动识别 schema** 让 Athena 直接查

**我的答案：** D) DynamoDB 存元数据　**正确答案：** B) Glue Crawler + Data Catalog

### 错因
不知道 **Glue Data Catalog 就是 Athena 的元数据存储**（写死的集成）。自建 DynamoDB 元数据表，Athena 根本不认。

### 要点
```
Glue Crawler 扫 S3 → 推断 schema → 写入 Glue Data Catalog
                                          ↓
                    Athena / Redshift Spectrum / EMR 直接当表查
```
- 信号词 `结构未知` + `自动识别 schema` → **Crawler**（唯一存在理由）
- 手写 DDL 能 work 但不选：凡「自动」对「手动」，选自动

---

## 错题 9：Redshift = 唯一的数据仓库答案

**场景：** 数据仓库跑 PB 级复杂 join 销售分析 + 业务方看可视化看板

**我的答案：** B) DynamoDB + Athena　**正确答案：** C) Redshift + QuickSight

### 错因 —— 方向性错误
**DynamoDB 不能做 join。** 它是 NoSQL 键值/文档库，为「已知 key 的毫秒级单点读写」设计：无 join、无聚合、不做 OLAP。当数据仓库用是根本性误解。

### 要点
🔑 **Redshift 是 SAA 里唯一的「数据仓库 / OLAP」答案**
看到这些词直接锁定，不犹豫：
`data warehouse`、`complex join`、`PB scale analytics`、`BI reporting`、`OLAP`

配套：可视化看板 → **QuickSight**

---

## 三轮毛病对比（更新）

| 毛病 | Aurora 轮 | DynamoDB 轮 | ElastiCache 轮 | 分析服务轮 |
|------|-----------|-------------|----------------|-----------|
| 得分 | 2/5 | 5/6 | **5/5** | **0/5** 🚨 |
| 过度设计 | ❌ 错 2 题 | ✅ 已改正 | ✅ | — |
| 漏读约束关键词 | ❌ 错 1 题 | ✅ 已改正 | ✅ | ❌ Q1 漏「偶尔」 |
| 计算题跳步 | — | ❌ 新暴露 | — | — |
| **服务能力边界记错** | — | — | — | ❌ **新暴露（Q3/Q4/Q5）** |

> **结论：** 前几轮是方法问题（靠技巧补），这轮是知识空洞（只能重记）。
> 分析服务在 SAA 占比低（约 1–3 题）且题型极固定 —— 背死关键词映射表即可满分，**投入不超过 30 分钟**。

---

# 分析服务重测：4/5（80%）— 2026-09-07 ✅ P1 收工

答对：EMR（自定义 Spark）、Athena 分区降本、QuickSight、Glue ETL（调度化 ETL 优于 Athena CTAS）
答错：KDS vs Firehose

## 错题 10：Kinesis Data Streams vs Firehose

**场景：** IoT 流数据**近实时**持续写入 S3，要求**全托管、免运维、无需写代码**

**我的答案：** A) KDS + 自建消费者　**正确答案：** B) Kinesis Data Firehose

### 错因
KDS 只负责**存流数据**，它不会自己往 S3 写 —— 需要自己写消费者程序，处理批量/重试/分区/失败落盘。题干「无需写代码 + 免运维」全部违反。

### 要点

| | Kinesis Data Streams | Kinesis Data Firehose |
|---|---|---|
| 定位 | 流数据**存储/管道** | 流数据**投递（ETL 搬运工）** |
| 消费者 | **自己写** | **AWS 托管**，配置即用 |
| 目的地 | 任意（自己实现） | 固定：**S3 / Redshift / OpenSearch / Splunk** |
| 延迟 | **实时** ~200ms | **近实时**，最低 60 秒缓冲 |
| 数据保留 | 1–365 天，**可重放** | ❌ 不存储，投递完即弃 |
| 扩缩容 | 管 shard（或 On-Demand） | 全自动 |

🔑 **判据**：`real-time` / `多个消费者` / `需要重放` → **KDS**
　　　　`near real-time` / `load into S3/Redshift` / `no code` → **Firehose**

---

## 📌 分析服务最终结论（不再投入）

关键词映射表（背死即满分，SAA 只占 1-3 题）：

| 信号词 | 答案 |
|---|---|
| S3 + `偶尔`/`ad-hoc` + `最小运维` | **Athena** |
| `data warehouse` / `复杂 join` / `PB 级` / `BI 报表` / `OLAP` | **Redshift** |
| 已有 Redshift + `不加载`查 S3 | **Redshift Spectrum** |
| `结构未知` / `自动发现 schema` | **Glue Crawler → Data Catalog** |
| `清洗`/`转换格式` + 无服务器 + **定期调度** | **Glue ETL** |
| `看板` / `可视化` / `BI` | **QuickSight** |
| `Hadoop` / `Spark` / 自定义框架调优 | **EMR** |
| 流数据 `近实时` 进 S3/Redshift + 无代码 | **Kinesis Firehose** |
| 流数据 `实时` / 多消费者 / 可重放 | **Kinesis Data Streams** |
| `全文搜索` / 日志分析看板 | **OpenSearch** |
| Athena 降本 | **Parquet/ORC + 压缩 + 分区** |

> 💡 通用解题技巧（这两轮反复验证）：先圈出题干的**频率词**（偶尔/持续/实时/每天）和**约束词**（成本/运维/无代码），答案基本就锁定了。
> 「两个选项技术上都能做」时，判据永远是**约束词**，不是技术可行性。
