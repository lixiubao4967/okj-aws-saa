# Week 5：数据库服务 — ElastiCache

> 学习日期：2026-09-06（内容）+ 2026-09-07（重讲 + 测验）
> ✅ **状态：完成。随堂测验 5/6 → 5/5 全对**（测验与解析见文末）

## 1. 定位

托管的**内存缓存**（Redis / Memcached），**亚毫秒级**延迟。

**核心考试逻辑：ElastiCache 出现在题目里，几乎总是为了「卸载数据库压力」。**

```
读密集 + 重复查询多 → 加缓存层，请求根本不落到 DB
```
（对应 Week 5 错题 3：Aurora 已有 3 个读副本仍不够 + 重复热点查询 → 答案是 ElastiCache）

⚠️ **ElastiCache 需要修改应用代码**（要自己写缓存查询/写入逻辑）。
这是它和 DAX 最大的区别 → 题目出现 `without changing application code` 就排除它。

---

## 2. Redis vs Memcached（**必考对比**）

| 维度 | **Redis** | **Memcached** |
|------|-----------|---------------|
| 数据结构 | 丰富：String / List / **Set** / **Sorted Set** / Hash / Bitmap / **Pub-Sub** / Stream | **只有简单 key-value** |
| 多线程 | 单线程（6.0+ 部分多线程 I/O） | ✅ **真·多线程**，吃满多核 |
| 副本 / 高可用 | ✅ **Read Replica + Multi-AZ 自动故障转移** | ❌ 无副本、**无故障转移** |
| 持久化 | ✅ **快照（RDB）+ AOF**，可备份恢复 | ❌ **纯内存，节点挂了数据全丢** |
| 分片 | ✅ Cluster Mode（最多 500 分片） | ✅ 多节点自动分片 |
| 事务 | ✅ | ❌ |
| 加密 / 认证 | ✅ 传输+静态加密、Redis AUTH、IAM | ❌ 基本没有 |
| 合规 | ✅ HIPAA / PCI-DSS | ❌ |

### 选型口诀

> 提到 **高可用 / 持久化 / 数据不能丢 / 排行榜 / 会话 / 发布订阅 / 地理空间 / 加密合规** → **Redis**
> 提到 **最简单的缓存 / 多核水平扩展 / 数据丢了无所谓 / 低成本** → **Memcached**

### 关键词直映射

| 题目出现 | 答案 |
|---|---|
| Leaderboard / ranking / top-N | **Redis Sorted Set** |
| Session store（用户会话） | **Redis**（要副本 + 持久化） |
| Pub/Sub、消息通知 | **Redis** |
| Geospatial（附近的人/店） | **Redis** |
| `simple, multi-threaded, scale out/in easily` | **Memcached** |
| `cannot afford to lose cached data` | **Redis** |

> 💡 **应试策略**：Memcached 在真题里出现频率远低于 Redis。
> **看到 Memcached，先检查题目有没有提到高可用或持久化——有就直接划掉。**

---

## 3. Redis 两种集群模式

| | **Cluster Mode Disabled** | **Cluster Mode Enabled** |
|---|---|---|
| 分片数 | **1 个**（1 主 + 最多 5 副本） | **多分片**（数据切分到多个主节点） |
| 扩展方式 | **纵向**（换更大机型）+ 加读副本 | **横向**（加分片） |
| 数据量 | 受单节点内存限制 | 可超大 |
| 用途 | 读扩展、高可用 | **写扩展 + 超大数据集** |

> `scale writes` / `dataset larger than a single node` → **Cluster Mode Enabled（分片）**
> `scale reads` / `high availability` → Cluster Mode Disabled + Read Replica + Multi-AZ

---

## 4. 缓存策略

| 策略 | 做法 | 优点 | 缺点 |
|------|------|------|------|
| **Lazy Loading**（Cache-Aside） | 先查缓存，miss 再查 DB 并写回 | 只缓存真正用到的数据，省内存；节点故障不致命 | **首次访问必然 miss**（延迟高）；数据可能陈旧 |
| **Write-Through** | 写 DB 时**同时**写缓存 | 缓存**永远最新**，无陈旧数据 | 写延迟增加；缓存里堆积**从没被读过**的冷数据 |
| **TTL** | 给缓存设过期时间 | 缓解陈旧问题，两种策略都该配 | 需调优 TTL 值 |

> `data must never be stale` → **Write-Through**
> `minimize memory / only cache what's requested` → **Lazy Loading**
> 实践中两者常**组合** + TTL

---

## 5. 划清「其他缓存类服务」的边界（场景题真正考点）

四个服务都叫缓存，但**位置完全不同**：

| 服务 | 缓存什么 | 放在哪 | 改代码？ |
|---|---|---|---|
| **CloudFront** | 静态/动态 HTTP 响应 | 全球边缘节点，**离用户最近** | 不用 |
| **ElastiCache** | 任意应用数据（DB 查询结果、session） | VPC 内，应用与 DB 之间 | **要** |
| **DAX** | **仅** DynamoDB 的 item/query | DynamoDB 前面（透明代理） | 不用（换 endpoint） |
| **RDS 读副本** | 不是缓存，是完整库拷贝 | — | 要改读写分离 |

排除法：
- 「加速静态图片」→ CloudFront（ElastiCache 在 VPC 里，用户够不着）
- 「DynamoDB 微秒级 + 不改代码」→ DAX
- 「Aurora 已有 3 个读副本仍扛不住重复查询」→ ElastiCache

> 💰 成本：ElastiCache 支持 **Reserved Nodes**，题干问「长期运行缓存集群如何降本」→ 预留节点（同 EC2 RI 逻辑）。

---

## 6. 随堂测验（2026-09-07 完成，5/5 ✅）

**答案：1-B　2-C　3-B　4-B　5-C**

**Q1.** 手游需要实时全球排行榜，展示 Top 100 玩家分数，毫秒内更新和查询。
A) DynamoDB + GSI　B) ElastiCache Redis（Sorted Set）　C) ElastiCache Memcached　D) RDS + 读副本

**Q2.** Web 应用把 session 存在 ElastiCache。要求某个缓存节点故障时**用户不掉线**。
A) Memcached 多节点自动分片　B) Redis Cluster Mode Enabled
C) Redis + Read Replica + Multi-AZ 自动故障转移　D) Redis + 每小时快照备份

**Q3.** DynamoDB 表被反复读取相同少量 item，团队要加缓存但**明确不改应用代码**，需微秒级延迟。
A) ElastiCache Redis　B) DAX　C) CloudFront　D) DynamoDB 最终一致读

**Q4.** 报表系统的价格数据**绝对不能陈旧**，任何写入必须立刻反映到缓存。用哪种缓存策略？
A) Lazy Loading　B) Write-Through　C) 只设 TTL　D) Parallel Scan

**Q5.** 电商缓存数据集已超过单个 Redis 节点内存上限，写入吞吐也成瓶颈。
A) 升级更大节点　B) 增加 Read Replica　C) 启用 Redis Cluster Mode Enabled（分片）　D) 换 Memcached

### 解析（易错点）

| 题 | 答案 | 关键判断信号 | 干扰项为什么错 |
|---|---|---|---|
| Q1 | B | `Top 100 + 实时排名` = Sorted Set 原生能力 | DynamoDB+GSI 能存分数但排序要自己算，做不到毫秒 Top-N |
| Q2 | C | `用户不掉线` = 需要**自动故障转移** | B 的 Cluster Mode 解决容量/写吞吐，不是 HA；D 快照是**恢复**手段，恢复期间用户已掉线 |
| Q3 | B | `不改代码` + `微秒级` + `DynamoDB` 三信号 | ElastiCache 必须自己写 cache-aside 逻辑 |
| Q4 | B | `绝对不能陈旧` = Write-Through | TTL 只缩短陈旧窗口，不能消除 |
| Q5 | C | 同时命中**容量超限 + 写吞吐瓶颈** | 读副本是完整拷贝，省不了内存；升级机型迟早再撞墙 |

> ⚠️ **Q2 / Q5 是最易翻车的两题**，核心分界：**副本扩读、分片扩写+扩容量**。
</content>
</invoke>
