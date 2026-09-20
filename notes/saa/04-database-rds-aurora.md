# Week 5：数据库服务 — RDS & Aurora

## 1. RDS 核心概念

### Multi-AZ vs Read Replica

| 维度 | Multi-AZ | Read Replica |
|------|----------|-------------|
| 目的 | **高可用**（故障转移） | **读取扩展**（分担读负载） |
| 同步方式 | 同步复制 | 异步复制 |
| 跨 Region | ❌ 同 Region | ✅ 可跨 Region |
| 可读写 | 备用实例不可读 | 可读（只读） |
| Failover | 自动（DNS 切换，60-120 秒） | 手动提升为独立数据库 |
| 数量 | 1 个备用 | 最多 5 个 |

### 备份与恢复
- **自动备份**：保留 0-35 天，支持 Point-in-Time Recovery（PITR）
- **手动快照**：不会过期，需手动删除
- 恢复操作会创建**新的数据库实例**（新 endpoint）

### RDS Proxy
- 管理数据库连接池，减少数据库连接压力
- 适用场景：Lambda 等大量短连接的无服务器应用
- 支持 IAM 认证，强制 TLS 加密

### RDS 安全
- 加密：KMS（静态）+ SSL/TLS（传输中）
- 未加密的数据库不能直接开启加密 → 需快照 → 复制快照并加密 → 从加密快照恢复

---

## 2. Aurora 核心概念

### 存储架构（重点）
- 数据自动复制 **6 个副本**，分布在 **3 个 AZ**（每个 AZ 2 份）
- 存储自动扩展：10GB → 128TB
- 容错：丢 4 副本仍可读，丢 2 副本仍可写
- **存储与计算分离**（shared storage volume）— 与 RDS 的关键架构区别

### Aurora 副本
- 最多 **15 个读副本**（RDS 只有 5 个）
- 复制延迟 < **10ms**（共享存储层）
- 支持**自动故障转移**，优先级可自定义（tier-0 到 tier-15）
- 故障转移时间 < **30 秒**（RDS 需 60-120 秒）

### Aurora Serverless
- 按需自动扩缩容，以 ACU 为单位
- Serverless v2：0.5 ACU 增量，支持读副本
- 适用场景：流量不可预测、间歇性工作负载、开发/测试环境

### Aurora Global Database
- 跨 Region 复制延迟 < **1 秒**
- 1 个主 Region（读写）+ 最多 **5 个次要 Region**（只读）
- 每个次要 Region 最多 **16 个读副本**
- 灾难恢复 RTO < 1 分钟（需手动触发提升）
- 考试关键词：**"global application"、"cross-region reads"**

### Aurora 其他特性
- **Backtrack**：将数据库回退到过去某个时间点（仅 MySQL 兼容版，整个库回退）
- **Clone**：快速克隆当前数据库状态（copy-on-write，适合测试环境）

---

## 3. Aurora vs RDS 对比

| 维度 | RDS | Aurora |
|------|-----|--------|
| 存储架构 | 单 AZ EBS | 6 副本跨 3 AZ 共享存储 |
| 最大存储 | 64 TB | 128 TB |
| 读副本数 | 最多 5 个 | 最多 15 个 |
| 复制延迟 | 秒级（异步） | < 10ms（共享存储） |
| Failover | 60-120 秒 | < 30 秒 |
| Serverless | ❌ | ✅ |
| Global Database | ❌ | ✅（< 1s 跨 Region） |
| 引擎支持 | MySQL/PostgreSQL/Oracle/SQL Server/MariaDB | 仅 MySQL/PostgreSQL 兼容 |
| 成本 | 较低 | 贵约 20%（性能更高） |

---

## 4. 考试高频场景速查

| 场景 | 选什么 |
|------|--------|
| 全球用户 + 低延迟读 + MySQL/PostgreSQL | Aurora Global Database |
| 流量不可预测 + 最小化成本 | Aurora Serverless v2 |
| 不影响现有库 + 恢复误删数据 | PITR 恢复到新集群（不是 Backtrack） |
| 单实例故障 + 快速恢复 | Aurora Replica + 自动 Failover |
| Region 级别灾难恢复 | Aurora Global Database |
| 重复热点查询 + 读取瓶颈 | ElastiCache 缓存（不是加副本） |
| Lambda 大量短连接 | RDS Proxy |
| **MongoDB / 文档数据库 / JSON documents** | **DocumentDB** |

---

## 5. 数据库认证：控制平面 vs 数据平面 🆕

> 补充于 2026-09-20（官方练习题 Q7 错题，原笔记未覆盖 DocumentDB）

```
控制平面 Control Plane —— 管「数据库这个资源」
  创建集群 / 改规格 / 删除 / 备份 / 打标签      ✅ IAM 管得了

数据平面 Data Plane —— 管「数据库里的数据」
  连接 / 查询 / 插入 / 删除                    ❌ 未必归 IAM 管
```

| 数据库 | IAM 管控制平面 | **IAM 能直接认证数据库连接吗** |
|---|---|---|
| **DynamoDB** | ✅ | ✅ **纯 IAM**，没有"数据库用户"概念 |
| **RDS MySQL / PostgreSQL** | ✅ | ✅ IAM database authentication |
| **Aurora MySQL / PostgreSQL** | ✅ | ✅ |
| RDS for Oracle / SQL Server | ✅ | ❌ |
| **DocumentDB** | ✅ | ❌ **只能用户名 + 密码** |
| Redshift | ✅ | ✅ 可用 IAM 换临时凭据 |

> **记忆逻辑**：DynamoDB 是 AWS 原生 → 纯 IAM；
> **兼容开源引擎的**（MongoDB / Oracle / SQL Server）受限于引擎自身认证机制，IAM 插不进去。

### Amazon DocumentDB

| 项目 | 内容 |
|---|---|
| 是什么 | 兼容 **MongoDB** 的托管文档数据库，存 JSON 文档 |
| 架构 | 类 Aurora：存算分离，存储自动扩容，最多 15 只读副本 |
| **题干关键词** | **MongoDB / document database / JSON documents / 从 MongoDB 迁移** |
| 认证 | 用户名 + 密码（IAM 只管控制平面） |
| 网络 | 必须在 VPC 内，不能公网直连 |

⚠️ **IAM policy 只回答"允不允许"，不会主动执行任何动作**（不会去改密码）。
想自动轮换数据库凭据 → **Secrets Manager**（支持 RDS / DocumentDB / Redshift 托管轮换）。
