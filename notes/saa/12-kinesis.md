# Kinesis 三兄弟 + SQS/SNS/Kinesis 终极区分

> P3 无服务器收尾 · 2026-09-11
> KDS vs Firehose 的核心区分见 `practice/saa/02-week5-errors.md` **错题 10**（曾栽过）

---

## 1. Kinesis Data Streams —— 分片是理解一切的钥匙

容量单位是 **Shard（分片）**，所有性能数字都是「每分片」：

| 每个分片 | 容量 |
|---|---|
| 写入 | **1 MB/s** 或 **1000 条记录/s** |
| 读取（共享） | **2 MB/s**，所有消费者**共享** |
| 读取（Enhanced Fan-Out） | 🔑 **每个消费者独占 2 MB/s** |

### 🔑 Enhanced Fan-Out 考点

多个消费者都要读同一个流，共享的 2MB/s 不够用 → 开 EFO：
- 每个消费者独享 2 MB/s
- 推模式（不用轮询）→ 延迟更低

### Partition Key 决定数据进哪个分片

```
✅ 好处：同一 partition key 的记录必然进同一分片
        → 保证【该 key 内部】的顺序
        （例：用 device_id 作 key，则每台设备的数据有序）

🚨 风险：partition key 分布不均 → 热分片（Hot Shard）
        与 DynamoDB 热分区同一个病
        → 选高基数、分布均匀的 key
```

### 保留与重放

**默认 1 天，最长 365 天，且可重放** —— 这是它 vs Firehose 的根本区别。

### 两种容量模式

| 模式 | 说明 |
|---|---|
| **Provisioned** | 自己算需要几个分片、手动扩缩 |
| **On-Demand** | 自动扩缩（上限 200 MB/s），流量不可预测时用 |

---

## 2. Firehose 补充（核心见错题 10）

- **缓冲**：按大小（1–128 MB）或时间（**最低 60 秒**）触发投递 → 「近实时」的来源
- **可内联转换**：调 Lambda 做格式转换/清洗；**可转 Parquet/ORC**（需配合 Glue Data Catalog 拿 schema）
- 🚨 **不存储、不能重放** —— 投递完即弃
- 目的地：**S3 / Redshift / OpenSearch / Splunk / HTTP 端点**

---

## 3. Managed Service for Apache Flink（原 Kinesis Data Analytics）

对流数据做**实时分析**：SQL 或 Flink 代码，做窗口聚合、异常检测、实时指标。

```
输入：KDS / MSK  →  Flink 实时计算  →  输出：KDS / Firehose / Lambda
```

> 🔑 题干「对流数据做**实时聚合 / 异常检测**」→ **Managed Service for Apache Flink**
> ⚠️ 不是 Athena —— Athena 查的是 **S3 上的静态数据**

**Kinesis Video Streams**：视频流摄取，SAA 极少考，知道存在即可。

---

## 4. ⭐ SQS vs Kinesis Data Streams（最高频对比）

都是「中间缓冲」，但设计哲学完全不同：

| | **SQS** | **Kinesis Data Streams** |
|---|---|---|
| 模型 | **队列** —— 消息被消费后**删除** | **流** —— 记录保留，**可重放** |
| 消费者 | 一个消费者组（消息只被处理一次） | 🔑 **多个独立消费者各读全量** |
| 顺序 | 只有 FIFO 才保证 | 🔑 **分片内按 partition key 有序** |
| 重放 | ❌ 不能 | ✅ **能**（保留期内任意位置重读） |
| 扩展 | 自动，无限 | 管 shard（或 On-Demand） |
| 保留 | 最长 **14 天** | 最长 **365 天** |
| 典型场景 | 任务解耦、削峰 | 实时数据管道、多方消费同一数据 |

> 🔑 **判据**
> `多个下游各要读全量` / `需要重放` / `严格时序的时序数据` → **Kinesis**
> `任务分发` / `削峰` / `一次性处理` → **SQS**

---

## 5. ⭐ SQS vs SNS vs Kinesis —— 一句话区分

```
SQS      = 消息被【一个】消费者拿走就没了      （分任务）
SNS      = 消息推给【所有】订阅者，推完就没了   （广播）
Kinesis  = 记录【留在流里】，谁都能反复读       （数据管道）
```

**SNS 和 Kinesis 都是「多消费者」，差别在持久性**：
- SNS 推送失败靠重试（所以要挂 SQS 兜底）
- Kinesis 数据本来就在那里，消费者自己记 checkpoint，随时回头重读

> 🚨 **这就是为什么「需要重放」永远选 Kinesis 而不是 SNS。**

---

## 6. ⭐ P3 全部时长数字（送分题重灾区）

| 服务 | 关键数字 |
|---|---|
| Lambda 执行 | **15 分钟** |
| API Gateway 集成 | **29 秒** |
| ALB 目标响应 | 默认 60 秒（可调） |
| SQS 消息保留 | 默认 4 天 / 最长 **14 天** |
| SQS 可见性超时 | 默认 **30 秒** / 最长 12 小时 |
| SQS 延迟队列 | 最长 **15 分钟** |
| SQS 长轮询 | 1–20 秒 |
| SQS 消息大小 | 256 KB |
| FIFO 去重窗口 | 5 分钟 |
| Step Functions Standard | 最长 **1 年** |
| Step Functions Express | 最长 **5 分钟** |
| Kinesis 保留 | 默认 1 天 / 最长 **365 天** |
| Firehose 缓冲 | 最低 **60 秒** |
| Lambda 异步重试 | **2 次**（共 3 次） |

---

## 7. 测验（8 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 一个 Kinesis 流有 4 个分片。理论最大写入吞吐是多少？
A) 1 MB/s　B) 2 MB/s　C) 4 MB/s　D) 8 MB/s

**Q2.** 5 个不同团队的应用都需要**读取同一个 Kinesis 流的全量数据**，且都抱怨读取吞吐不足、延迟高。最佳方案？
A) 增加分片数　B) 启用 Enhanced Fan-Out　C) 改用 SQS　D) 改用 Firehose

**Q3.** IoT 平台要求**每台设备的数据严格按时间顺序**处理，设备数上万。Kinesis 该怎么配？
A) 用随机 UUID 作 partition key　B) 用固定常量作 partition key　C) 用 device_id 作 partition key　D) 用时间戳作 partition key

**Q4.** 团队用 device_type 作 partition key，只有 3 种取值，结果**某个分片吞吐打满而其他空闲**。这是什么问题？
A) 分片数不足　B) 热分片（partition key 基数太低）　C) 保留期太短　D) 消费者太少

**Q5.** 需要对流入的交易数据做**实时窗口聚合并检测异常**。选？
A) Athena　B) Managed Service for Apache Flink　C) Firehose → S3 → Athena　D) Redshift

**Q6.** 下游系统出了 bug，需要**把过去 3 天的数据重新处理一遍**。当前用的是 Firehose → S3。能否直接重放 Firehose？
A) 能，Firehose 保留 7 天　B) 不能，Firehose 不存储数据；但可以从 S3 重新读取　C) 能，调用 Replay API　D) 不能，数据已永久丢失

**Q7.** 订单系统需要把任务分发给一组后台 worker，每个任务**只需被处理一次**，流量有明显峰谷。选？
A) Kinesis Data Streams　B) SNS　C) SQS　D) EventBridge

**Q8.** 用户行为日志需要同时给：实时推荐引擎、离线数仓、风控系统三方消费，且**风控需要能回溯过去 7 天重算**。选？
A) SQS + 三个消费者　B) SNS + 三个 SQS　C) Kinesis Data Streams（保留 7 天）　D) Firehose → S3
