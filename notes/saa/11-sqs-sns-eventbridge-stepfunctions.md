# SQS / SNS / EventBridge / Step Functions（应用集成）

> P3 无服务器 · 2026-09-10
> 这四个是「应用集成」的全部，考试以**场景选型题**为主。

---

## 1. SQS —— 队列（一对一，拉模式）

**核心价值：解耦 + 缓冲削峰。** 生产者写入即返回，消费者按自己速度拉取。

### 标准队列 vs FIFO 队列

| | **标准队列** | **FIFO 队列** |
|---|---|---|
| 吞吐 | **无限** | 300 msg/s（批处理 3000/s，高吞吐模式更高） |
| 顺序 | **尽力而为**（可能乱序） | 🔑 **严格保证顺序** |
| 投递 | **至少一次**（可能重复） | 🔑 **精确一次**（自动去重） |
| 队列名 | 任意 | 🚨 必须以 **`.fifo`** 结尾 |

FIFO 的两个必填参数：
- **MessageGroupId** —— 同组内保证顺序（不同组可并行）
- **MessageDeduplicationId** —— **5 分钟去重窗口**

> 🔑 判据：`顺序很重要`、`不能重复处理`、`交易/订单必须按序` → **FIFO**

### 四个必考参数

| 参数 | 值 | 用途 |
|---|---|---|
| **消息保留** | 默认 4 天，最长 **14 天** | 消费者宕机时的缓冲期 |
| **消息大小** | 上限 **256 KB** | 更大 → **S3 + SQS Extended Client Library** |
| **可见性超时** | 默认 30 秒，最长 **12 小时** | 见下 🚨 |
| **长轮询** | 1–20 秒 | 见下 🚨 |

### 🚨 可见性超时（Visibility Timeout）—— 最高频

```
消费者取走消息 → 消息被"隐藏"（其他消费者看不到）
  ├─ 在超时内删除消息 → 正常结束
  └─ 超时了还没删除    → 消息重新出现 → 被别人再处理一次（重复！）

症状：「消息被处理了多次」
原因：处理时间 > 可见性超时
修复：调长可见性超时，或代码里调 ChangeMessageVisibility 续期
```

### 🚨 长轮询（Long Polling）

短轮询会不停返回空响应，白白产生 API 调用费。
设 `ReceiveMessageWaitTimeSeconds = 1~20 秒` → **减少空响应、降低成本、降低延迟**。

> 题干「降低 SQS 成本 / 减少空响应」→ **长轮询**

### DLQ（死信队列）

消息被接收超过 `maxReceiveCount` 次仍未成功 → 转入 DLQ。
用途：**隔离「毒消息」并排查**，避免它无限阻塞队列。

### 与 ASG 联动（考点）

用 CloudWatch 指标 **`ApproximateNumberOfMessagesVisible`** 做目标追踪扩缩容
—— 队列积压就加机器。

---

## 2. SNS —— 主题（一对多，推模式）

**核心价值：扇出（Fan-out）。** 一条消息推给所有订阅者。

订阅者类型：**Lambda、SQS、HTTP/S、Email、SMS、Kinesis Firehose、移动推送**

### ⭐ 经典扇出架构（考试最爱）

```
                    ┌→ SQS-A → Lambda-A（发货系统）
S3 事件 → SNS Topic ─┼→ SQS-B → Lambda-B（数据分析）
                    └→ SQS-C → Lambda-C（邮件通知）
```

**为什么中间要插 SQS，不直接 SNS → Lambda？**（高频追问）

- ✅ **数据持久化** —— Lambda 挂了消息还在队列里，不会丢
- ✅ **独立重试** —— 每个消费者按自己节奏重试，互不影响
- ✅ **削峰** —— 突发流量在队列里缓冲

### 消息过滤（Filter Policy）

按消息属性把不同消息只发给关心它的订阅者，
避免每个订阅者都收全量再自己过滤。

### 标准 vs FIFO Topic

FIFO Topic **只能给 SQS FIFO 队列**订阅。

---

## 3. EventBridge —— 事件总线（升级版 SNS）

原名 CloudWatch Events。**规则匹配 + 路由**的事件中枢。

### 它比 SNS 多出来的能力 = 它的全部考点

| 能力 | 说明 |
|---|---|
| **内容级过滤** | 匹配复杂 JSON 事件模式（不只是属性） |
| **第三方 SaaS 集成** | 🔑 Zendesk、Datadog、Shopify… 直接接入 |
| **Archive & Replay** | 🔑 **归档事件并重放**（审计 / 故障恢复 / 测试） |
| **Schema Registry** | 自动发现事件结构、生成代码绑定 |
| **定时调度** | 🔑 **cron / rate 表达式** → 替代 crontab |

三种总线：**Default**（AWS 服务自身事件）、**Custom**（你的应用）、**Partner**（SaaS）

> 🔑 **SNS vs EventBridge 判据**
> 简单扇出 + 超高吞吐 + 低延迟 → **SNS**
> `第三方 SaaS`、`定时任务/cron`、`复杂事件过滤`、`事件重放`、`审计` → **EventBridge**

---

## 4. Step Functions —— 工作流编排

把多个 Lambda / 服务串成**可视化状态机**（用 ASL — Amazon States Language 定义）。

**状态类型**：`Task`（干活）、`Choice`（分支）、`Parallel`（并行）、
`Map`（并行迭代数组）、`Wait`（等待）、`Succeed`/`Fail`、`Pass`

| | **Standard** | **Express** |
|---|---|---|
| 最长时长 | 🔑 **1 年** | 🔑 **5 分钟** |
| 执行语义 | **精确一次** | 至少一次 |
| 吞吐 | 较低 | **超高**（10 万/秒） |
| 执行历史 | ✅ 完整可审计 | 只进 CloudWatch Logs |
| 适合 | 长流程、**人工审批**、需审计 | 流式处理、IoT、高频短任务 |

### 四个高频用途

1. 🔑 **流程超过 Lambda 的 15 分钟** → 拆成多个 Lambda 由 Step Functions 串联
2. 🔑 **需要重试 / 错误捕获 / 条件分支** → 内置 `Retry` / `Catch`，不用在 Lambda 里手写
3. 🔑 **需要人工审批** → **Wait for Callback（Task Token）**，流程暂停等外部回调
4. 需要可视化流程状态，便于排查哪一步失败

---

## 5. ⭐ 五服务选型总表（背这张就够）

| 需求 | 服务 |
|---|---|
| 解耦 + 缓冲削峰，**一个**消费者组处理 | **SQS** |
| **严格顺序** / 不能重复 | **SQS FIFO** |
| 一条消息给**多个**消费者（扇出） | **SNS**（+ 每个下游挂 SQS） |
| 第三方 SaaS 事件 / cron 定时 / 复杂过滤 / **事件重放** | **EventBridge** |
| **多步骤工作流** / 分支 / 人工审批 / **>15 分钟** | **Step Functions** |
| 实时流数据、多消费者、**可重放** | **Kinesis Data Streams** |
| 流数据近实时投递到 S3/Redshift，无代码 | **Kinesis Firehose** |

---

## 6. ⭐ 两个易混点

### SQS 还是 SNS？本质是问「几个消费者」

```
1 个消费者要处理    → SQS（消息被取走就没了）
N 个消费者都要一份  → SNS（每个订阅者都收到副本）
```

但真考更常见的是「两个都要」= **SNS + SQS 扇出模式**。
看到「一个事件要触发多个互不相关的下游系统，且任一系统故障不能影响其他」
→ 答案永远是 **SNS → 多个 SQS → 各自消费者**。

### SQS 的 DLQ ≠ Lambda 的 DLQ

| | 触发条件 | 去向 |
|---|---|---|
| **SQS DLQ** | 消息被反复接收但处理不成功（超 `maxReceiveCount`） | 另一个 SQS 队列 |
| **Lambda DLQ / On-failure Destination** | **异步调用**的 Lambda 重试 2 次仍失败 | SQS / SNS |

注意题干说的是「队列里的消息」还是「Lambda 的调用事件」。

---

## 7. 测验（12 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 电商订单必须**按下单顺序**处理，且**绝不能重复扣款**。应选？
A) SQS 标准队列　B) SNS Topic　C) SQS FIFO 队列　D) EventBridge

**Q2.** SQS 消费者处理一条消息需要约 **2 分钟**，但运维发现**同一条消息被处理了多次**。最可能原因？
A) 用了标准队列所以必然重复　B) 可见性超时是默认 30 秒，小于处理时间　C) 没开长轮询　D) 消息保留时间太短

**Q3.** 团队发现 SQS 的 API 调用费用偏高，日志里大量**空响应**。最直接的优化？
A) 改用 FIFO 队列　B) 缩短消息保留时间　C) 启用长轮询（ReceiveMessageWaitTimeSeconds）　D) 增加消费者数量

**Q4.** 一个 S3 上传事件需要**同时触发**发货、分析、通知三个互不相关的系统，且**任一系统故障不能影响其他系统**。
A) S3 → Lambda 里依次调用三个系统　B) S3 → SQS → 三个消费者　C) S3 → SNS → 三个 SQS → 各自 Lambda　D) S3 → EventBridge → Lambda

**Q5.** 需要处理的消息体是 **5 MB** 的 JSON。SQS 该怎么用？
A) 直接发，SQS 支持 10MB　B) 压缩后发　C) 消息体存 S3，队列里只放引用（SQS Extended Client Library）　D) 改用 SNS

**Q6.** 某业务流程共 8 个步骤、有条件分支和重试需求，**总时长约 45 分钟**。最合适的编排方式？
A) 一个 Lambda 里顺序调用　B) Step Functions Standard 工作流　C) Step Functions Express 工作流　D) SQS 链式队列

**Q7.** 需要在每天凌晨 3 点自动触发一个 Lambda 做清理。最合适？
A) EC2 上配 crontab　B) EventBridge 定时规则（cron 表达式）　C) SQS 延迟队列　D) Step Functions Wait 状态

**Q8.** 公司使用 Zendesk，希望**工单创建时自动触发** AWS 内部流程。
A) SNS　B) SQS　C) EventBridge（Partner Event Bus）　D) API Gateway 轮询

**Q9.** 审计要求：**能把过去 30 天的事件重新回放一遍**用于故障复盘。
A) SNS 消息保留　B) SQS 14 天保留　C) EventBridge Archive & Replay　D) CloudTrail

**Q10.** ASG 应该根据 SQS 队列积压情况扩缩容。用哪个 CloudWatch 指标？
A) CPUUtilization　B) NumberOfMessagesSent　C) ApproximateNumberOfMessagesVisible　D) ApproximateAgeOfOldestMessage 的倒数

**Q11.** 报销流程需要在中间**暂停等待经理人工审批**（可能等几天），审批后继续。
A) Step Functions + Wait for Callback（Task Token）　B) Step Functions Express　C) SQS 延迟队列（最长 15 分钟）　D) Lambda 里 sleep

**Q12.** 某 SQS 队列里有一条格式错误的「毒消息」，反复被消费失败，**阻塞了后续消息处理**。应该？
A) 延长可见性超时　B) 配置 DLQ 并设置 maxReceiveCount　C) 改用 FIFO 队列　D) 缩短消息保留时间
