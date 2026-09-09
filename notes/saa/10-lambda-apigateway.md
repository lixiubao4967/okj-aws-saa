# Lambda + API Gateway

> P3 无服务器 · 2026-09-08（原定 9/12，提前 4 天）

---

## 1. Lambda 硬限制 —— 全是考点

| 项目 | 限制 |
|---|---|
| 内存 | 128 MB – **10,240 MB**（10 GB） |
| **CPU** | 🚨 **不能单独配，与内存成正比**（要更多 CPU 只能加内存） |
| **超时** | 🚨 **最长 15 分钟** |
| `/tmp` 临时磁盘 | 512 MB – 10 GB |
| 部署包 | zip 直传 50 MB / 解压后 250 MB；**容器镜像 10 GB** |
| Layers | 最多 **5 层** |
| 默认账户并发 | **1000**（可申请提升） |

### 🚨 「15 分钟」是最高频的一条

题干说任务要跑 **1 小时 / 几小时** → 答案绝不是 Lambda：

```
> 15 分钟的任务 → ECS / Fargate（长时运行任务）
                 → AWS Batch（批处理作业）
                 → Step Functions 拆成多个 Lambda 串联
```

---

## 2. ⭐ 三种并发概念（极易混淆）

| | 作用 | 用途 |
|---|---|---|
| **账户并发上限** | 默认 1000，全账户共享 | 超了返回 **429 TooManyRequestsException** |
| **Reserved Concurrency**<br>预留并发 | 给某函数**划出专属额度**，同时也是它的**硬上限** | 🔑 防止一个函数吃光全账户配额<br>设为 **0** 可**紧急停用**某函数 |
| **Provisioned Concurrency**<br>预置并发 | **预先初始化好的实例**，一直热着 | 🔑 **消除冷启动**，用于延迟敏感场景 |

> ⚠️ 名字太像，用**目的**区分：
> **Reserved = 限流 / 隔离（管配额）**
> **Provisioned = 保温 / 降延迟（管性能）**

### 冷启动（Cold Start）

首次调用时下载代码 + 启动运行时 + 跑初始化代码 → 额外几百毫秒到几秒。

降低手段：
1. **Provisioned Concurrency**（最直接）
2. 减小部署包体积
3. 把初始化代码（DB 连接、SDK client）放在 **handler 外面**，复用执行环境

---

## 3. 🚨 Lambda 与 VPC（高频陷阱）

**默认 Lambda 运行在 AWS 托管的 VPC** —— 能访问公网，但**访问不到你 VPC 内的 RDS / ElastiCache**。

要访问 → 把 Lambda **配置到你的 VPC**（指定子网 + SG）。副作用：

```
🚨 Lambda 一旦放进你的 VPC，就失去了默认的公网访问能力

  → 要同时访问 RDS 和 外部 API？
      必须放在【私有子网】+ 配置【NAT Gateway】

  → 只是要访问 S3 / DynamoDB？
      用 Gateway Endpoint（免费，不用 NAT GW）
```

> 典型考题：「VPC 内的 Lambda 能连 RDS，但调用外部支付 API 超时」→ **缺 NAT Gateway**

---

## 4. 三种调用模型 —— 决定错误如何处理

| 模型 | 触发源 | 重试行为 |
|---|---|---|
| **同步** | API Gateway、ALB、CLI、SDK | ❌ **不自动重试**，错误直接返回调用方 |
| **异步** | S3、SNS、EventBridge | ✅ **自动重试 2 次**（共 3 次），失败后可进 **DLQ / On-failure Destination** |
| **事件源映射** | SQS、Kinesis、DynamoDB Streams | Lambda **主动轮询**；SQS 失败回队列直到进 DLQ |

> 🔑 题干问「Lambda 处理失败的事件如何不丢」→ **DLQ（SQS/SNS）或 On-failure Destination**

---

## 5. Lambda@Edge vs CloudFront Functions

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| 语言 | 仅 **JavaScript** | Node.js / Python |
| 延迟 | **亚毫秒级** | 毫秒级 |
| 触发点 | 仅 **Viewer** Request/Response | Viewer + **Origin** Request/Response（共 4 个） |
| 访问网络/请求体 | ❌ 不能 | ✅ 可以 |
| 并发 | 极高（百万级） | 较低（千级） |
| 成本 | 便宜 ~1/6 | 较贵 |

> 🔑 **简单 header 改写 / URL 重写 / 跳转** → **CloudFront Functions**（更快更便宜）
> **需调外部服务 / 访问请求体 / 操作 origin 请求** → **Lambda@Edge**

---

## 6. API Gateway：三种类型

| | **HTTP API** | **REST API** | **WebSocket API** |
|---|---|---|---|
| 定位 | 简单、便宜、快 | **全功能** | 双向实时 |
| 成本 | 🔑 **比 REST 便宜约 70%** | 贵 | — |
| 独有功能 | — | **API Key + 用量计划**、请求验证、**缓存**、WAF、Canary 部署 | 持久连接、服务端推送 |

> 🔑 「**最低成本**的简单 Lambda 代理」→ **HTTP API**
> 需要 **API Key / 用量配额 / 缓存 / WAF** → **REST API**

---

## 7. ⭐ API Gateway 端点类型与授权

### 端点类型

| 类型 | 场景 |
|---|---|
| **Edge-Optimized** | 全球分布的客户端（自动走 CloudFront 边缘） |
| **Regional** | 客户端集中在同一 Region |
| **Private** | 🚨 **仅能从 VPC 内经 Interface Endpoint 访问**（内部 API） |

### 授权方式（高频对比）

| 方式 | 适用 |
|---|---|
| **IAM**（SigV4 签名） | AWS 内部服务 / 有 IAM 身份的调用方 |
| **Cognito User Pools** | 🔑 **面向终端用户的注册登录**（托管，最省事） |
| **Lambda Authorizer** | 🔑 **第三方身份系统 / 自定义 token 逻辑**（OAuth、自定义 JWT 校验） |
| **API Key** | 🚨 **不是认证手段！** 只用于**用量计划 / 限流 / 计费** |

### 🚨 集成超时 29 秒

```
API Gateway 集成超时上限 = 29 秒

  Lambda 能跑 15 分钟，但前面挂了 API Gateway 的话，
  超过 29 秒 API GW 就先超时返回 504。

  → 长任务正确姿势：API GW 同步返回「已接收」，
     实际处理交给 SQS / Step Functions 异步做
```

---

## 8. ⭐ 三个超时数字（串起来记）

```
API Gateway 集成   →  29 秒
Lambda 执行        →  15 分钟
ALB 目标响应       →  默认 60 秒（可调）
```

题干只要给出「任务需跑 XX 时间」，立刻能判断架构对不对 —— **送分题**。

## 9. ⭐ Lambda 排错四步法

凡出现「超时 / 连不上 / 间歇性失败」，按顺序查：

```
① 超过 15 分钟了吗？              → 换 ECS/Fargate/Batch/Step Functions
② 前面是 API GW 且超过 29 秒吗？   → 改异步（SQS/Step Functions）
③ 放进 VPC 了但缺 NAT GW 吗？      → 加 NAT GW 或 VPC Endpoint
④ 是冷启动导致的偶发慢吗？          → Provisioned Concurrency
```

覆盖绝大多数 Lambda 排错题，比逐个选项分析快得多。

---

## 10. 测验（10 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 数据处理任务需要连续运行约 **2 小时**。最合适的方案？
A) Lambda 配 10GB 内存　B) ECS on Fargate　C) Lambda + Provisioned Concurrency　D) Lambda@Edge

**Q2.** 某 Lambda 函数流量突增，把账户 1000 并发几乎全占满，导致其他关键函数被限流。如何隔离？
A) 给它设置 Reserved Concurrency　B) 给它设置 Provisioned Concurrency　C) 提高账户并发上限　D) 前面加 SQS 缓冲

**Q3.** 支付类 API 对延迟极敏感，不能接受**冷启动**带来的偶发几秒延迟。
A) Reserved Concurrency　B) Provisioned Concurrency　C) 增加内存　D) 改用 HTTP API

**Q4.** Lambda 配置进 VPC 私有子网后能连上 RDS，但**调用外部第三方 API 全部超时**。原因？
A) SG 没放开 443 出站　B) Lambda 超时设置太短　C) 私有子网缺少 NAT Gateway　D) 需要 Interface Endpoint

**Q5.** 想要让 Lambda 获得**更多 CPU**算力，应该怎么做？
A) 选择更大的实例类型　B) 调高 vCPU 参数　C) 增加分配的内存　D) 开启多线程选项

**Q6.** S3 上传事件触发 Lambda（异步调用），少量事件处理失败后丢失。如何保证不丢？
A) 改成同步调用　B) 提高超时时间　C) 开启 S3 版本控制　D) 配置 DLQ / On-failure Destination

**Q7.** CloudFront 上需要做一个简单的 **URL 重写**，要求延迟最低、成本最低。
A) Lambda@Edge　B) CloudFront Functions　C) API Gateway　D) ALB 监听器规则

**Q8.** 公司要对外提供 API，需要给**不同客户分配不同的调用配额并计费**。应选？
A) HTTP API　B) WebSocket API　C) ALB + Lambda　D) REST API + API Key + 用量计划

**Q9.** 移动 App 需要**用户注册登录**后调用 API Gateway，希望**最省开发和运维**。
A) IAM 授权（SigV4）　B) Lambda Authorizer　C) Cognito User Pools　D) API Key

**Q10.** 客户端调用 API Gateway → Lambda，Lambda 需要处理约 **5 分钟**。用户反馈总是收到 504。原因与修复？
A) Lambda 超时太短，调到 15 分钟　B) 加 Provisioned Concurrency　C) 换 Edge-Optimized 端点　D) API GW 集成超时上限 29 秒，应改为异步（同步返回「已接收」+ SQS/Step Functions 处理）
