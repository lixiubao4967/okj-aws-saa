# Week 5：数据库服务 — DynamoDB

> 学习日期：2026-09-06（P1 补完数据库）
> 随堂测验：**5/6（83%）**，错题见 `practice/saa/02-week5-errors.md`

## 1. 定位

| 维度 | 说明 |
|------|------|
| 类型 | **NoSQL**，键值 + 文档 |
| 运维 | **全托管 Serverless**，无实例、无补丁、无版本 |
| 可用性 | 天生跨 **3 个 AZ** 复制（不需要配 Multi-AZ） |
| 扩展 | 存储无上限，自动分区 |
| 延迟 | 个位数**毫秒**（DAX 加持 → **微秒**） |

**信号词 → DynamoDB**：`millisecond latency`、`serverless`、`unpredictable/massive scale`、`key-value`、`session state`、`no schema`

**反向排除**：出现 `complex JOIN`、`SQL reporting`、`跨多表事务分析` → 选 RDS / Aurora

---

## 2. 数据模型

```
Table
 └── Item（一行，最大 400 KB ← 记死）
      └── Attribute（列，除主键外无需预定义）
```

### 主键两种形态

| 形态 | 组成 | 唯一性要求 |
|------|------|-----------|
| Simple | **Partition Key (PK)** | PK 值必须唯一 |
| Composite | **Partition Key + Sort Key (SK)** | PK 可重复，PK+SK 组合唯一 |

PK 经内部 hash → 决定数据落在哪个物理分区。

### ⚠️ 热分区（Hot Partition）

PK 基数（cardinality）低 → 流量集中到少数分区 → **被限流**。

- ❌ 错误示范：用 `status`（只有 active/inactive）当 PK
- ✅ 正确：选**高基数、访问均匀**的属性，如 `user_id` / `order_id` / `device_id`

> 题目出现 `throttling` / `ProvisionedThroughputExceededException` / `uneven access pattern`
> → 先怀疑 **Partition Key 设计**

---

## 3. 容量模式

### On-Demand vs Provisioned

| 维度 | On-Demand | Provisioned |
|------|-----------|-------------|
| 计费 | 按**请求数** | 按**预置容量** |
| 扩缩 | 瞬时自动 | 配 RCU/WCU，可开 Auto Scaling |
| 成本 | 单价贵约 **5-7 倍** | 便宜，可买 Reserved Capacity 再省 |
| 适用 | **流量不可预测**、新应用、突发尖峰 | **流量可预测**、稳定负载、成本敏感 |

> `unpredictable / spiky / unknown traffic` → **On-Demand**
> `predictable / steady + cost optimization` → **Provisioned + Auto Scaling**

### RCU / WCU 计算（**考试会算，必须固定顺序**）

**基本单位**
- 1 **WCU** = 每秒 1 次写入，每次最大 **1 KB**
- 1 **RCU** = 每秒 1 次**强一致读**，每次最大 **4 KB**
  = 每秒 2 次**最终一致读**（打对折）
- **事务操作（Transactional）= 双倍消耗**

**固定三步（顺序不能乱）**

```
① 向上取整成块   写：ceil(size ÷ 1KB)   读：ceil(size ÷ 4KB)
② 乘以每秒次数
③ 最终一致读 → ÷ 2      事务 → × 2
```

> 🚨 **绝对不要先做除法再乘**。容量单位是离散的块，不存在 1.5 RCU——
> 读 5KB 和读 8KB 收费一样（都是 2 RCU）。

**例题**

| 题目 | 计算 | 答案 |
|------|------|------|
| 每秒 100 次读，6 KB，最终一致 | `ceil(6÷4)=2` → `2×100=200` → `÷2` | **100 RCU** |
| 每秒 10 次写，2 KB | `ceil(2÷1)=2` → `2×10` | **20 WCU** |
| 每秒 20 次写，3.5 KB | `ceil(3.5÷1)=4` → `4×20` | **80 WCU** |
| 每秒 50 次读，10 KB，强一致 | `ceil(10÷4)=3` → `3×50` | **150 RCU** |

---

## 4. 索引：GSI vs LSI（**最高频考点**）

| 维度 | **LSI**（Local Secondary Index） | **GSI**（Global Secondary Index） |
|------|------|------|
| Partition Key | **必须和主表相同** | **可以不同**（任意属性） |
| Sort Key | 换一个 | 可选，任意属性 |
| 创建时机 | **只能建表时创建**，之后不能加/删 ⚠️ | **随时创建/删除** ✅ |
| 数量上限 | 5 个 | 20 个（可申请提升） |
| 容量 | **共享主表 RCU/WCU** | **有独立 RCU/WCU** |
| 一致性 | 支持**强一致读** | **只有最终一致读** |
| 大小限制 | 每个 PK 值下所有 item ≤ **10 GB** | 无限制 |

### 解题决策树

```
要用「非主键属性」做查询条件？
 ├─ 需要换 Partition Key            → GSI
 └─ Partition Key 不变，只换 Sort Key → LSI

表已经上线了，现在才要加索引？      → 只能 GSI
```

### ⚠️ GSI 反噬主表（爱考）

**GSI 的写容量不足被限流时，会反过来导致主表写不进去。**

> 题目：主表写入开始 throttle，但主表 WCU 使用率只有 40%，表上有 GSI
> → 答案是 **提升 GSI 的 WCU**，不是改主表

LSI 没这个问题（共享主表容量），但有 10GB 硬限制。

---

## 5. DAX（DynamoDB Accelerator）

- **专为 DynamoDB 设计的内存缓存**：毫秒 → **微秒**
- **应用代码几乎不用改**（API 完全兼容，只换 endpoint）← 对比 ElastiCache 的最大卖点
- 缓存两类：**item cache**（GetItem）+ **query cache**（Query/Scan）
- 默认 TTL 5 分钟，部署在 **VPC 内**

### DAX vs ElastiCache

| 场景 | 选谁 |
|------|------|
| 缓存 DynamoDB 的读，且**不想改代码** | **DAX** |
| 缓存**聚合结果 / 计算结果 / 跨数据源** | **ElastiCache** |
| 需要 Redis 数据结构（排行榜、Pub/Sub） | **ElastiCache Redis** |

> `microsecond` + `DynamoDB` + `minimal application changes` → **DAX**（送分题）

---

## 6. TTL / Streams / Global Tables

### TTL
- item 上加一个存 **Unix 时间戳（秒）** 的属性，到期自动删除
- **删除不消耗 WCU（免费）** ← 考点
- 删除有延迟，通常 **48 小时内**完成，**不保证准时**
- 场景：session、购物车、日志清理、合规数据保留策略

### DynamoDB Streams
- 捕获 **item 级变更**（INSERT / MODIFY / REMOVE）
- 保留 **24 小时**
- 最常见组合：**Streams → Lambda**（变更后发通知、同步到 OpenSearch）
- 需要更长保留（**1 年**）→ **Kinesis Data Streams for DynamoDB**

### Global Tables
- **多 Region 多活（multi-active）**：每个 Region 都能**读写**
- **前提：必须先启用 DynamoDB Streams** ← 考点
- 冲突解决：**Last Writer Wins**
- 关键词：`globally distributed users with low-latency local read AND write`

> 📌 **与 Aurora 对照记（Week 5 错题 2 的延伸）**
> - **Aurora Global Database** = 单写多读（次要 Region 只读），failover **手动**
> - **DynamoDB Global Tables** = **多活**，各 Region 都可写
> - 题目出现 `write in multiple regions` → 排除 Aurora Global Database

### 备份
- **PITR**：最长 **35 天**，可恢复到任意秒（同 RDS）
- **On-Demand Backup**：手动，永久保留，不影响性能
- **Export to S3**：**不消耗 RCU**（走 PITR 快照）

---

## 7. Query vs Scan

| | Query | Scan |
|---|---|---|
| 方式 | 按 **PK（+SK 条件）** 精确定位 | **全表扫描**逐条过滤 |
| 效率 | 高 | 极低，**消耗大量 RCU** |
| 考试态度 | ✅ 首选 | ❌ 尽量避免；必须用就开 **Parallel Scan** + 限制 page size |

> 题目出现「Scan 导致成本高 / 性能差」→ 答案通常是 **建 GSI 改用 Query**

---

## 8. 安全

- 静态加密**默认开启**（KMS）
- IAM Policy 的 **`dynamodb:LeadingKeys` 条件键** → 实现**行级权限**
  （只允许用户访问自己 PK 对应的数据）← 细节考点
- VPC 内私网访问 → **VPC Gateway Endpoint**
  （**DynamoDB 和 S3 是仅有的两个 Gateway 型 Endpoint**，其余全是 Interface 型）

---

## 9. 一句话速查

| 题目关键词 | 答案 |
|-----------|------|
| unpredictable traffic | On-Demand |
| predictable + 省钱 | Provisioned + Auto Scaling |
| 上线后要加索引 | GSI |
| 只换 Sort Key，建表时就规划好 | LSI |
| 主表 throttle 但容量够 | 提升 GSI WCU |
| microsecond + 不改代码 | DAX |
| 自动过期清理、不花 WCU | TTL |
| 变更触发下游处理 | Streams → Lambda |
| 多 Region 都要写 | Global Tables |
| Scan 太慢太贵 | 建 GSI 改 Query |
| 行级权限 | `dynamodb:LeadingKeys` |
</content>
</invoke>
