# SAA-C03 服务与数字速查表

> 考前速查 · 2026-09-14 整理自 `notes/saa/01`～`17`
> **考试当天只看这两份 cheatsheet + 错题本**

---

## 🔢 一、数字速查（送分题重灾区）

### 计算 / 无服务器

| 项目 | 数字 |
|---|---|
| **Lambda 超时** | **15 分钟** |
| Lambda 内存 | 128 MB – 10,240 MB（**CPU 随内存成正比，不能单配**） |
| Lambda /tmp | 512 MB – 10 GB |
| Lambda 部署包 | zip 50 MB / 解压 250 MB / **容器镜像 10 GB** |
| Lambda Layers | 最多 **5 层** |
| Lambda 默认并发 | **1000**（账户级，可提） |
| Lambda 异步重试 | **2 次**（共 3 次） |
| **API Gateway 集成超时** | **29 秒** |
| ALB 目标响应超时 | 默认 60 秒（可调） |
| **Step Functions Standard** | 最长 **1 年**，精确一次 |
| **Step Functions Express** | 最长 **5 分钟**，至少一次，10 万/秒 |

### 消息 / 流

| 项目 | 数字 |
|---|---|
| SQS 消息保留 | 默认 4 天 / 最长 **14 天** |
| SQS 消息大小 | **256 KB**（更大→ S3 + Extended Client） |
| **SQS 可见性超时** | 默认 **30 秒** / 最长 12 小时 |
| SQS 延迟队列 | 最长 **15 分钟** |
| SQS 长轮询 | 1–20 秒 |
| SQS FIFO 吞吐 | 300 msg/s（批处理 3000/s） |
| FIFO 去重窗口 | **5 分钟** |
| **Kinesis 分片** | 写 **1 MB/s 或 1000 条/s**；读共享 **2 MB/s** |
| Kinesis EFO | **每消费者独占 2 MB/s** |
| Kinesis 保留 | 默认 1 天 / 最长 **365 天** |
| **Firehose 缓冲** | 最低 **60 秒**（或 1–128 MB） |

### 网络

| 项目 | 数字 |
|---|---|
| VPC CIDR 范围 | `/16`（65536）～ `/28`（16） |
| **每子网保留 IP** | **5 个**（`.0` `.1` `.2` `.3` + 最后一个）<br>→ `/24` = **251** 可用 |
| VPC Peering 全互通 | **N(N−1)/2** 条 |
| VPN 单隧道带宽 | **1.25 Gbps**，每连接 **2 条隧道** |
| **Direct Connect 交付周期** | 🚨 **1 个月以上** |
| DX 带宽 | Dedicated 1/10/100 Gbps；Hosted 50 Mbps–10 Gbps |
| NAT GW 带宽 | 5 Gbps 自动扩到 **100 Gbps** |

### 数据库

| 项目 | 数字 |
|---|---|
| **RCU**（4 KB 块） | 强一致 1 RCU = 1 次 4KB 读；**最终一致 ÷2**；**事务 ×2** |
| **WCU**（1 KB 块） | 1 WCU = 1 次 1KB 写；**事务 ×2** |
| RDS 备份保留 | 0–35 天 |
| Aurora 副本 | 最多 **15 个** |
| DynamoDB item | 最大 **400 KB** |

### 安全 / 成本

| 项目 | 数字 |
|---|---|
| **KMS Encrypt API 上限** | **4 KB** → 超过用**信封加密** |
| CMK 费用 | $1/月 |
| Secrets Manager | **$0.40/密钥/月** |
| Parameter Store | 标准层**免费**，4 KB |
| **Shield Advanced** | **$3000/月** |
| **EKS 控制平面** | **$0.10/小时/集群**（约 $73/月） |
| RI / Savings Plans 折扣 | 最多 **72%**（**同一档**） |
| **Spot 折扣** | 最多 **90%**，**2 分钟**中断通知 |
| CloudTrail 控制台保留 | **90 天** |
| Trusted Advisor 免费版 | **7 项**检查 |

---

## 🚨 二、默认值陷阱（最容易失分）

| 服务 | 默认行为 | 陷阱 |
|---|---|---|
| **CloudWatch EC2 指标** | **无内存、无磁盘** | 必须装 **CloudWatch Agent** |
| **CloudWatch Logs** | **默认永久保留** | 不设过期一直计费 |
| **CloudTrail** | 默认开，只留 90 天 | 长期保留要建 Trail 投递 S3 |
| **CloudTrail Data Events** | **默认不记录** | 审计 S3 对象操作必须手动开 |
| **VPC Flow Logs** | **默认不开启** | 需手动启用 |
| **Trusted Advisor** | 只有 7 项 | 全量需 **Business/Enterprise** |
| **Shield Standard** | **免费自动开启** | 不用做任何配置 |
| **GuardDuty** | 自己读日志 | **不需要先开 Flow Logs/DNS 日志** |
| **NAT Gateway** | **单 AZ 高可用** | 🚨 **每个 AZ 各放一个** |
| **Lambda in VPC** | **失去公网访问** | 需 NAT GW 或 VPC Endpoint |
| **SG** | 入站全拒/出站全允 | 有状态，无需配响应规则 |
| **新建 NACL** | **全拒** | 无状态，**出站必开 1024-65535** |

---

## 📊 三、核心对比表

### SG vs NACL

| | **Security Group** | **NACL** |
|---|---|---|
| 层级 | 实例/ENI | **子网** |
| 状态 | **有状态** | **无状态** |
| 规则 | **只有 Allow** | Allow + **Deny** |
| 求值 | 所有规则取并集 | **按规则号从小到大，首个匹配生效** |
| 引用 | ✅ 可引用其他 SG | ❌ 只能 CIDR |
| 数量 | 实例可挂多个 | 子网只能挂**一个** |

> 🔑 **封单个恶意 IP → 只能 NACL**（SG 没有 deny）
> 🔑 **应用层访问 DB → SG 引用 SG**（扩缩容无需改规则）

### Gateway vs Interface Endpoint

| | **Gateway Endpoint** | **Interface Endpoint** |
|---|---|---|
| 服务 | 🚨 **只有 S3 和 DynamoDB** | 几乎所有其他服务 |
| 费用 | 🚨 **免费** | 按小时 + 流量 |
| 实现 | 路由表前缀列表 | 子网内 **ENI** |
| 挂 SG | ❌ | ✅ |
| **本地 DC 可达** | 🚨 **不能** | 🚨 **能**（经 DX/VPN） |

### SQS vs SNS vs Kinesis

```
SQS      = 消息被【一个】消费者拿走就没了      （分任务）
SNS      = 消息推给【所有】订阅者，推完就没了   （广播）
Kinesis  = 记录【留在流里】，谁都能反复读       （数据管道）
```

| | SQS | Kinesis Data Streams |
|---|---|---|
| 模型 | 队列，消费后删除 | 流，**可重放** |
| 消费者 | 一个消费者组 | **多个独立消费者各读全量** |
| 顺序 | 仅 FIFO | **分片内按 partition key 有序** |
| 保留 | 最长 14 天 | 最长 **365 天** |

### KDS vs Firehose

| | **Kinesis Data Streams** | **Firehose** |
|---|---|---|
| 定位 | 存储/管道 | **投递（搬运工）** |
| 消费者 | **自己写** | AWS 托管 |
| 目的地 | 任意 | S3/Redshift/OpenSearch/Splunk |
| 延迟 | **实时** ~200ms | **近实时**，最低 60 秒 |
| 重放 | ✅ | ❌ **不存储** |

### Lambda 两种并发

```
Reserved    = 限流 / 隔离（管配额）  → 防某函数吃光账户 1000 并发；设 0 可紧急停用
Provisioned = 保温 / 降延迟（管性能）→ 消除冷启动
```

### CloudTrail vs Config

```
CloudTrail  记录【动作】—— "Alice 在 10:23 调用了 ModifySecurityGroup"  → 问「谁」
Config      记录【状态】—— "这个 SG 现在是 B 配置，B 不合规"            → 问「合不合规」
Drift Detection —— "改得和 CloudFormation 模板一致吗"                  → 问「一致吗」
```

### GuardDuty vs Inspector

```
Inspector  = 我自己有没有洞？  （已知 CVE，事前预防）
GuardDuty  = 有人在攻击我吗？  （行为异常，事中检测）
```

### Secrets Manager vs Parameter Store

| | **Secrets Manager** | **Parameter Store** |
|---|---|---|
| **自动轮换** | 🔑 **✅ 内置** | ❌ 无 |
| 费用 | **$0.40/密钥/月** | **标准层免费** |

> 判据只看两条：**要不要自动轮换** + **在不在乎那 $0.40**

### ECS vs EKS / EC2 vs Fargate

```
编排层：ECS（AWS 自研，免费）  或  EKS（托管 K8s，$73/月）
        判据 = 题干提没提 K8s 生态（kubectl/Helm/多云）—— 与规模无关

算力层：EC2 模式  或  Fargate
        Fargate 是默认答案；只有三个信号选 EC2：GPU / 特殊实例 / Spot
```

**ECS 两层角色**（按时间先后记）：
```
Task Execution Role = 【启动前】ECS 代理拉 ECR 镜像、建日志流
Task Role           = 【运行后】你的代码读 S3、写 DynamoDB
```

### RI vs Savings Plans vs Spot

```
RI             承诺【特定实例类型 + Region】       折扣高，不灵活
               Standard（不可改）/ Convertible（可换类型）
Savings Plans  承诺【每小时花多少钱】               灵活
               Compute SP  = 最灵活，🔑 覆盖 Lambda 和 Fargate
               EC2 Instance SP = 折扣更高但锁实例族
Spot           可被 2 分钟通知中断                  省 90%
```

### Beanstalk 部署策略

| 策略 | 停机 | 特点 |
|---|---|---|
| All at once | 🚨 **有** | 最快最便宜 |
| Rolling | 无 | **容量下降** |
| Rolling + additional batch | 无 | 容量不降，多花钱 |
| **Immutable** | 无 | 🔑 **零停机 + 最快回滚** |
| **Blue/Green** | 无 | 🔑 **Swap CNAME** |

---

## 🌐 四、VPC 互联排除法

| 看到这个约束 | 立刻排除 |
|---|---|
| **CIDR 重叠** | Peering、TGW → 只能 **PrivateLink** |
| 需要传递路由 | Peering（不可传递） |
| 「几天内上线」 | **Direct Connect**（要 1 个月） |
| S3/DynamoDB 之外的服务 | Gateway Endpoint |
| 本地 DC 要私有访问 | Gateway Endpoint |
| 要挂 Security Group | Gateway Endpoint、NAT GW |
| 要免费 | Interface Endpoint |
| 要加密 | 裸 Direct Connect（默认不加密） |
| 组播 multicast | 除 **TGW** 外全部 |

**DX 三条铁律**：
1. 「最快」→ 不选 DX，选 **VPN 先顶**
2. 「要加密」→ **DX 上跑 IPsec VPN**
3. 「最低成本消除单点」→ **DX + VPN 备份**（不是买第二条 DX）

---

## 🔐 五、WAF 五个挂载点（背死）

**CloudFront / ALB / API Gateway / AppSync / Cognito User Pool**

🚨 **挂不上 NLB** → 要在 NLB 前加 CloudFront，或换成 ALB

---

## ⏱️ 六、超时 / 时长决策

```
任务 < 15 分钟                → Lambda
任务 > 15 分钟                → ECS/Fargate、AWS Batch、Step Functions
前面挂 API GW 且 > 29 秒      → 改异步（同步返回「已接收」+ SQS/Step Functions）
流程需暂停等人工审批（几天）    → Step Functions + Wait for Callback (Task Token)
流程 ≤ 5 分钟 + 超高吞吐      → Step Functions Express
```

---

## 🔧 七、Lambda 排错四步法

```
① 超过 15 分钟了吗？             → 换 ECS/Fargate/Batch/Step Functions
② 前面是 API GW 且超过 29 秒吗？  → 改异步
③ 放进 VPC 了但缺 NAT GW 吗？     → 加 NAT GW 或 VPC Endpoint
④ 是冷启动导致的偶发慢吗？         → Provisioned Concurrency
```

## 🔧 八、EC2 上不了网排错

```
① 实例有公有 IP / EIP 吗？        ← 最常被忽略
② 路由表有 0.0.0.0/0 → IGW 吗？
③ IGW 附加到 VPC 了吗？
④ SG 出站放开了吗？
⑤ NACL 出站放开 1024-65535 了吗？ ← 无状态陷阱
```
