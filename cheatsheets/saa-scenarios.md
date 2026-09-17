# SAA-C03 场景题解题思路 + 个人对策

> 考前速查 · 2026-09-14
> **这份比服务表更重要 —— 你的失分几乎全在「审题」而非「知识」**

---

## 🚨 零、考前必读：我最顽固的失分点

分模块测验成绩：P2 95%、P3 93%、P4 93%、P5 100%。
**但 6 道错题里有 3 道是同一个毛病。**

### ⚠️ 第一大问题：漏掉后半句的「限定条件」（已栽 3 次）

| 错题 | 题干的两层 | 我抓住的 | 我漏掉的 |
|---|---|---|---|
| 错题 1（Aurora） | 快速恢复 + **不影响现有库** | 快速恢复 | **不影响现有库** |
| 错题 12（Kinesis） | 分散均匀 + **严格顺序** | 分散均匀 | **严格顺序** |
| 错题 15（EC2） | 故障自愈 + **保留 ID/IP** | 故障自愈 | **保留 ID/IP** |

**规律**：抓住了「主需求」，漏掉了后半句的「限定条件」。
**而主需求往往有好几个方案都满足，限定条件才是唯一的筛子。**

### ✅ 考场动作（每题必做）—— 🔬 已用模拟卷验证有效

> **选出答案后，不要立刻进入下一题。**
> **回到题干，找出每一句带要求的话，逐句问：「我选的这个，满足这句吗？」**
> **任何一句不满足，换选项。**

**为什么是这个动作，而不是「读题时圈约束词」**：
圈约束词是**注意力指令**，撑不过 50 分钟（模拟卷第 1、2 批各 63.6%，指令无效）；
回读验证是**输出后的机械检查**，不依赖状态。

**实测数据（混合模拟卷①，2026-09）**：
```
第 1 批  14/22  63.6%   无验证机制
第 2 批  14/22  63.6%   无验证机制
第 3 批  19/21  90.5%   ← 执行回读验证，+27 个百分点
```

**走一遍（卷① Q23）**：
```
选了 Pilot Light
  → 回读要求句①「RTO 4 小时」                     满足 ✓
  → 回读要求句②「RPO 1 小时」                     满足 ✓
  → 回读要求句③「不接受第二区域长期运行任何计算资源」 ✗ ← 停住，换选项
```

> 🚨 **稳定性才是真问题**：21 题漏 1 次（Q64），65 题会漏 3–4 次。
> 考场疲劳更高 → **把这个动作做到不用想。**

**正面示范（P5 Q7 做对了）**：
```
题干：稳定负载 + 可能更换实例类型 + 覆盖 Lambda 和 Fargate

约束①「可能更换实例类型」→ 砍掉 Standard RI、EC2 Instance SP
约束②「覆盖 Lambda/Fargate」→ 砍掉 Convertible RI
                              → 只剩 Compute Savings Plans ✅

只看约束① 的话，Convertible RI 和 Compute SP 都像对的。
```

### ⚠️ 其他 4 条

| # | 毛病 | 对策 |
|---|---|---|
| 2 | **把服务属性套到整个架构**（错题 13：Firehose 不存储 ≠ 数据丢了） | **先画数据路径，再问「数据现在躺在哪」** |
| 3 | **警惕「XX 不支持 YY」类选项**（错题 14） | AWS 极少用「功能不存在」当答案；**选项越像具体操作步骤越可能对** |
| 4 | **「对了一半」的选项**（错题 13、防护五件套 Q5） | 答案「看起来就是它」时多问：**这个服务在这个具体位置能用吗？** |
| 5 | **计算题跳步**（错题 4 RCU） | **写出三步，不心算** |

### 🔢 RCU/WCU 固定三步（绝不心算）

```
① 向上取整成块   读：ceil(size ÷ 4KB)   写：ceil(size ÷ 1KB)
② 乘以每秒次数
③ 最终一致读 → ÷2       事务 → ×2
```
> 🚨 **绝不先除再乘**。读 5KB 和 8KB 收费一样（都 2 RCU）。

### ⚠️ 反向题（已出现 5 次，真考预计 3–6 道）

> 看到 **「下列错误的是」「以下哪项不是」「除了」** 就在心里标红，
> **答完回头确认一次方向没反。**

---

## 🧭 一、通用解题流程

```
① 圈【频率词】    偶尔 / 持续 / 实时 / 近实时 / 每天 / 一次性
② 圈【约束词】    成本最低 / 运维最小 / 无代码 / 零停机 / 严格顺序 /
                  不影响现有 / 保留 XX / 几天内上线 / 加密 / 合规
③ 先【排除】      每个服务都有「一票否决」的硬限制，先砍掉不可能的
④ 再从剩下的选【最简单的】—— 不要过度设计
```

> **两个选项都「技术可行」时，判据永远是约束词，不是技术可行性。**
> 例：Athena CTAS 和 Glue ETL 都能把 CSV 转 Parquet，
> 　　但题干说「每天跑一次」→ 需要调度 → **Glue ETL**

---

## 🔑 二、关键词 → 服务 映射总表

### 分析

| 信号词 | 答案 |
|---|---|
| S3 + `偶尔`/`ad-hoc` + `最小运维` | **Athena** |
| `data warehouse` / `复杂 join` / `PB 级` / `BI 报表` / `OLAP` | **Redshift** |
| 已有 Redshift + `不加载`查 S3 | **Redshift Spectrum** |
| `结构未知` / `自动发现 schema` | **Glue Crawler → Data Catalog** |
| `清洗/转换格式` + 无服务器 + **定期调度** | **Glue ETL** |
| `看板` / `可视化` / `BI` | **QuickSight** |
| `Hadoop` / `Spark` / 自定义框架调优 | **EMR** |
| 流数据 `近实时` 进 S3/Redshift + 无代码 | **Firehose** |
| 流数据 `实时` / 多消费者 / 可重放 | **Kinesis Data Streams** |
| 流数据 `实时聚合 / 异常检测` | **Managed Service for Apache Flink** |
| `全文搜索` / 日志分析看板 | **OpenSearch** |
| **Athena 降本** | **Parquet/ORC + 压缩 + 分区** |

### 数据库

| 信号词 | 答案 |
|---|---|
| 关系型 + 托管 | RDS |
| 关系型 + 高性能 + 全球 | Aurora（Global Database 跨 Region） |
| 单实例故障自动切换 <30s | **Aurora Replica + 故障转移优先级** |
| 误删数据 + **不影响现有库** | **PITR 恢复到新集群**（不是 Backtrack） |
| 键值 / 毫秒级 / 无服务器 | **DynamoDB** |
| DynamoDB 微秒级读 | **DAX** |
| **重复/热点查询** | **ElastiCache** |
| 多样化读负载 | **增加读副本** |
| 会话存储 / 排行榜 / Pub-Sub | **ElastiCache Redis** |
| 纯缓存 / 多线程 / 可水平分片 | **ElastiCache Memcached** |

### 应用集成

| 信号词 | 答案 |
|---|---|
| 解耦 + 削峰，**一个**消费者组 | **SQS** |
| **严格顺序** / 不能重复 | **SQS FIFO** |
| 一事件给**多个**互不影响的下游 | **SNS → 多个 SQS → 各自消费者** |
| 第三方 SaaS / cron 定时 / 复杂过滤 / **事件重放** | **EventBridge** |
| 多步骤工作流 / 分支 / 人工审批 / **>15 分钟** | **Step Functions** |
| 消息被处理**多次** | **可见性超时 < 处理时间** |
| 大量空响应、API 费高 | **长轮询** |
| 毒消息阻塞队列 | **DLQ + maxReceiveCount** |
| ASG 按队列积压扩缩 | **ApproximateNumberOfMessagesVisible** |

### 安全

| 信号词 | 答案 |
|---|---|
| `控制密钥策略` / `审计密钥使用` / `跨账户` | **Customer Managed Key (CMK)** |
| 加密**大文件** / 超 4KB | **信封加密 / GenerateDataKey** |
| KMS AccessDenied 但 IAM 看起来对 | **检查 Key Policy**（双重授权） |
| `自动轮换` / `rotate` | **Secrets Manager** |
| 配置参数 + `成本最低` | **Parameter Store** |
| EC2/Lambda/ECS 访问 AWS 服务 | 🔑 **IAM Role**（永远不用 access key） |
| 第三方访问你的账户 | **AssumeRole + External ID** |
| `SQL 注入` / `XSS` / `地理封禁` / `IP 限流` | **WAF** |
| `DDoS` + 专家支持 + 费用补偿 | **Shield Advanced** |
| 挖矿 / 恶意 IP 通信 / 凭证泄露 | **GuardDuty** |
| EC2/镜像/Lambda 的 **CVE 漏洞** | **Inspector** |
| S3 里的 **PII / 身份证号 / 信用卡号** | **Macie** |
| 多账户集中查看所有安全发现 | **Security Hub** |

### 监控

| 信号词 | 答案 |
|---|---|
| 「**谁**在什么时候做了什么」 | **CloudTrail** |
| EC2 **内存 / 磁盘**使用率 | **CloudWatch + Agent** |
| 「配置**合规**吗 / 什么时候变的」 | **AWS Config** |
| 「CloudFormation 资源被手动改了」 | **Drift Detection** |
| 成本优化 / 服务限额建议 | **Trusted Advisor**（全量需 Business） |
| 「为什么两台 EC2 不通」 | **VPC Flow Logs** |
| 日志里 ERROR 超 N 次告警 | **Metric Filter + Alarm** |
| EC2 硬件故障自愈 + **保留 ID/IP** | **CloudWatch Alarm + Recover** |
| EC2 故障自愈（无 ID/IP 要求） | **ASG 自动替换** |

### 成本

| 信号词 | 答案 |
|---|---|
| **可中断** / 容错 / 批处理 | **Spot**（省 90%） |
| 稳定负载 + 最大折扣 | **RI 或 EC2 Instance SP** |
| 要折扣 + 保留灵活 + 覆盖 Lambda/Fargate | **Compute Savings Plans** |
| S3 **访问模式不可预测** + 无检索费 | **Intelligent-Tiering** |
| S3 访问模式**已知**（如 30 天后不用） | **生命周期策略** |
| 私有子网访问 S3 的 **NAT 流量费高** | **S3 Gateway Endpoint**（免费） |
| 「钱花在哪 / 预测」 | **Cost Explorer** |
| 「超支告警」 | **AWS Budgets** |
| 物理隔离 / BYOL 许可 | **Dedicated Host** |

### 容器 / IaC

| 信号词 | 答案 |
|---|---|
| `Kubernetes` / `kubectl` / `多云` | **EKS** |
| 跑容器 + `不管服务器` / `运维最小` | **ECS on Fargate** |
| `GPU` / 特殊实例 / `Spot` | **ECS/EKS on EC2** |
| 容器内应用读 S3 | **Task Role**（不是 Execution Role） |
| 同机多任务端口冲突 | **ALB 动态端口映射** |
| 最快上线 Web API，什么都不想管 | **App Runner** |
| 批处理作业（跑完退出，几小时） | **AWS Batch** |
| 多账户多 Region 统一部署模板 | **StackSets** |
| 更新前预览影响 | **Change Sets** |
| 零停机 + **最快回滚** | **Immutable** |
| 全新环境再切流量 | **Blue/Green（Swap CNAME）** |

---

## 🌍 三、高频架构母题

### ① 三层 Web 架构的标准答案

```
Route 53 → CloudFront → ALB（公有子网）
                          ↓
                  ASG + EC2（私有子网）  ← SG 引用 ALB 的 SG
                          ↓
                  RDS Multi-AZ（私有子网）← SG 引用 App 层的 SG
                          ↑
                  ElastiCache（热点查询）
```
- 私有子网出网 → **每个 AZ 一个 NAT Gateway**
- 访问 S3/DynamoDB → **Gateway Endpoint**（免费，省 NAT 费）
- 静态资源 → **S3 + CloudFront**

### ② 高可用 / 灾备（RTO/RPO 从松到紧）

| 策略 | RTO/RPO | 做法 |
|---|---|---|
| **Backup & Restore** | 小时级 | 快照跨 Region 复制，最便宜 |
| **Pilot Light** | 十分钟级 | 核心（数据库）常开，其余关机 |
| **Warm Standby** | 分钟级 | 全套缩小版常开，故障时扩容 |
| **Multi-Site Active/Active** | 秒级 | 两地全量运行，最贵 |

> 判据：题干给 RTO/RPO 数字 → 直接对号入座；给「最低成本」→ Backup & Restore

### ③ 「事件触发多个系统」

```
S3/事件源 → SNS → 多个 SQS → 各自 Lambda
                    ↑
            为什么插 SQS：持久化 + 独立重试 + 削峰
```
**但如果要求「能回溯重放」→ 改用 Kinesis Data Streams**（差别只在这一个词）

### ④ 长任务改造

```
客户端 → API Gateway（29 秒上限）
           ↓ 同步立即返回「已接收 + 任务 ID」
         SQS / Step Functions
           ↓ 异步处理（可以跑很久）
         结果写 S3/DynamoDB，客户端轮询或用 WebSocket 推送
```

### ⑤ 混合云连接

```
快速上线 / 预算有限        → Site-to-Site VPN
稳定低延迟 / 大流量降本    → Direct Connect（1 个月交付）
专线 + 加密               → DX + IPsec VPN
DX 的经济型冗余           → DX + VPN 备份
多 VPC + 本地 DC 集中管理  → Transit Gateway
```

---

## ✅ 四、考试当天操作清单

**考试信息**：2026-09-26（土）11:30 JST｜新桥国际会馆考场｜English｜**170 分钟**（含 ESL +30）

```
□ 65 题 / 160 分钟答题 ≈ 每题 2.4 分钟 —— 不要在单题耗超 3 分钟，先标记跳过
□ 第一遍：会的立刻答，不确定的标记（Flag）后跳过
□ 第二遍：回头做标记题，用排除法
□ 每题必做：圈频率词 + 圈约束词，尤其是【后半句的限定条件】
□ 看到「错误的是 / 不是 / 除了」→ 心里标红，答完确认方向
□ 计算题写三步，不心算
□ 两个选项都可行时 → 看约束词，选最简单的
□ 50 题计分 + 15 题不计分，不认识的题不要慌，可能就是不计分的
□ 720/1000 及格，补偿计分，单域不及格不影响总分
```

**考前最后一天（9/25）只做两件事**：
1. 读这两份 cheatsheet
2. 读 `practice/saa/01`～`06` 错题本，**重点是本文件第零节**
