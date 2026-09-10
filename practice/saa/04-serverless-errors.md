# P3 无服务器 + 集成 — 测验记录

## 成绩趋势

| 日期 | 模块 | 得分 |
|------|------|------|
| 2026-09-09 | Lambda + API Gateway | **10/10（100%）** 🎯 |
| 2026-09-11 | 应用集成（SQS/SNS/EventBridge/Step Functions） | **12/12（100%）** 🎯 |

---

# Lambda + API Gateway：10/10 — 2026-09-09

**无错题。** 出题时未给格式示例、选项未加粗答案 → 数据有效。

全部答对的考点：

| # | 考点 | 答案 |
|---|---|---|
| 1 | 2 小时任务 | **ECS on Fargate**（Lambda 上限 15 分钟） |
| 2 | 某函数吃光账户并发 | **Reserved Concurrency**（限流/隔离） |
| 3 | 消除冷启动 | **Provisioned Concurrency**（保温/降延迟） |
| 4 | VPC 内 Lambda 连不上外部 API | **缺 NAT Gateway** |
| 5 | 想要更多 CPU | **增加内存**（CPU 随内存成正比，不能单配） |
| 6 | 异步调用失败不丢 | **DLQ / On-failure Destination** |
| 7 | 边缘 URL 重写，最低延迟成本 | **CloudFront Functions** |
| 8 | 按客户分配配额并计费 | **REST API + API Key + 用量计划** |
| 9 | 终端用户注册登录，最省运维 | **Cognito User Pools** |
| 10 | API GW → Lambda 5 分钟报 504 | **API GW 集成超时 29 秒 → 改异步** |

## 📌 本轮巩固的关键区分

**Reserved vs Provisioned 并发（用目的记，不用名字记）**
```
Reserved    = 限流 / 隔离（管配额）  → 防止某函数吃光账户 1000 并发；设 0 可紧急停用
Provisioned = 保温 / 降延迟（管性能）→ 消除冷启动
```

**Q3 的诱饵值得单独记**：C「增加内存」确实能让 Lambda 跑更快（CPU 随内存增长），
**但它缩短的是执行时间，不是冷启动时间**。冷启动 = 启动运行时 + 跑初始化代码，
加内存只能略微加速，无法消除。题干「不能接受」= 要根治 → 只有 Provisioned。

**Q10 的 A 选项是「治错了病」的典型**：调 Lambda 超时到 15 分钟完全无效，
因为卡点在 API Gateway 的 29 秒，不在 Lambda。真考里这种选项很常见。

## ⭐ 三个超时数字（串起来记）
```
API Gateway 集成   →  29 秒
Lambda 执行        →  15 分钟
ALB 目标响应       →  默认 60 秒（可调）
```

## ⭐ Lambda 排错四步法
```
① 超过 15 分钟了吗？             → 换 ECS/Fargate/Batch/Step Functions
② 前面是 API GW 且超过 29 秒吗？  → 改异步（SQS/Step Functions）
③ 放进 VPC 了但缺 NAT GW 吗？     → 加 NAT GW 或 VPC Endpoint
④ 是冷启动导致的偶发慢吗？         → Provisioned Concurrency
```

---

# 应用集成：12/12（100%）— 2026-09-11 🎯

**无错题。** 未给格式示例、选项未加粗答案 → 数据有效。

全部答对的考点：

| # | 考点 | 答案 |
|---|---|---|
| 1 | 按序 + 不重复扣款 | **SQS FIFO** |
| 2 | 消息被处理多次 | **可见性超时(默认30s) < 处理时间(2min)** |
| 3 | 大量空响应、API 费高 | **长轮询** |
| 4 | 一事件触发多个互不影响的系统 | **SNS → 多个 SQS → 各自 Lambda** |
| 5 | 5MB 消息体 | **存 S3 + 队列放引用**（SQS 上限 256KB） |
| 6 | 8 步骤 + 分支 + 45 分钟 | **Step Functions Standard** |
| 7 | 每天凌晨 3 点触发 | **EventBridge cron 规则** |
| 8 | Zendesk 工单触发 AWS 流程 | **EventBridge Partner Event Bus** |
| 9 | 回放过去 30 天事件 | **EventBridge Archive & Replay** |
| 10 | ASG 按队列积压扩缩容 | **ApproximateNumberOfMessagesVisible** |
| 11 | 暂停等人工审批（几天） | **Step Functions + Wait for Callback (Task Token)** |
| 12 | 毒消息阻塞队列 | **DLQ + maxReceiveCount** |

## 📌 本轮巩固的关键区分

### 「症状相似，病因不同」—— Q2 vs Q12

| | 症状 | 病因 | 修复 |
|---|---|---|---|
| **Q2** | 消息被处理**多次** | 处理时间 > 可见性超时，消息重新出现 | 调长可见性超时 |
| **Q12** | 消息**阻塞队列** | 消息本身有问题，永远处理不成功 | DLQ 把它捞出去 |

> 关键区别：**Q2 的消息其实处理成功了**（只是没来得及删除）；**Q12 的消息永远不会成功**。
> 一个是调参数，一个是隔离。

### 时长约束一票否决（Q11）

「可能等几天」直接砍掉 Express（5 分钟上限）和 SQS 延迟队列（15 分钟上限）。
→ **先看时长约束，能砍掉一半选项。**

### Q6 的 A 选项是双重错误

「一个 Lambda 里顺序调用 8 步」既超 15 分钟上限，又把重试/分支塞进代码。
→ **凡「多步骤 + 需要重试/分支」，答案基本都是 Step Functions，不要用 Lambda 硬扛。**
