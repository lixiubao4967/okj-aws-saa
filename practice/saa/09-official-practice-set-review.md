# AWS 官方练习题集复盘（Official Practice Question Set, SAA-C03）

> 来源：AWS Skill Builder 免费 20 题 · exam mode · 2026-09-19
> 这是**出题方自己写的题**，代表性高于任何第三方题库。

## 成绩

| 指标 | 结果 | 目标 | 差距 |
|---|---|---|---|
| 原始正确率 | **55%**（11/20） | ~72% | −17 |
| **加权正确率** | **52.8%** | ~72% | **−19** |
| 平均每题耗时 | **3 分 01 秒** | 2 分 00 秒 | **超 50%** |

### 域得分（按真实考试权重加权）

| Domain | 考试权重 | 得分 | 加权贡献 |
|---|---|---|---|
| D1 安全架构 | **30%** | 40%（2/5） | 12.0 |
| D2 弹性架构 | **26%** | 40%（2/5） | 10.4 |
| D3 高性能架构 | 24% | 60%（3/5） | 14.4 |
| D4 成本优化 | 20% | **80%**（4/5） | 16.0 |
| **合计** | 100% | 55% | **52.8%** |

🚨 **加权分低于原始分**：强的 D4 权重最小，弱的 D1+D2（合计占考试 **56%**）都只有 40%。

### Task 级红区

| Task | 成绩 |
|---|---|
| 2.1 可扩展 & 松耦合 | 1/3（33%） |
| 1.3 数据安全控制 | 0/1 |
| 3.2 弹性计算 | 0/1 |
| 3.3 高性能数据库 | 0/1 |

---

## 🚨 核心发现：模块满分 → 官方题不及格

| 涉及模块 | 当时模块测验 | 本卷对应题 |
|---|---|---|
| 容器三兄弟 | **10/10** | Q1 ❌ |
| SQS/SNS/EventBridge | **12/12** | Q1 ❌ |
| 防护五件套 | **10/10** | Q4 ❌ Q18 ❌ |
| VPC 核心 + 互联 | **19/20** | Q18 ❌ |
| DynamoDB（含 DAX） | **5/6** | Q17 ❌ |

**9 道错题中 8 道的知识点，笔记里都有。** Q17（DAX）甚至在笔记里被自己标注为「送分题」。

**结论：不是知识缺口，是 4 个月的遗忘 + 提取失败。**
→ 补救方式是**高频跨模块提取练习**，不是重读笔记。

---

## 9 道错题总览

| # | 题 | 我选 | 正确 | 失误类型 | 耗时 |
|---|---|---|---|---|---|
| 1 | Q1 视频处理架构 | EKS + SNS | ECS Fargate + SQS | 组合调用失败 | 2:45 |
| 2 | Q4 XSS/SQLi 防护 | Shield | **WAF** | 🔴 Shield 边界 | 3:23 |
| 3 | Q7 DocumentDB 认证 | 虚构能力 | IAM + 用户名密码 | **真知识缺口** | 2:57 |
| 4 | Q8 降低成本 | Instance Scheduler | 重构为 Lambda | 不敢换架构 | — |
| 5 | Q9 Pilot Light | Warm Standby | Pilot Light | 术语混淆 | 2:52 |
| 6 | Q10 缩容时 5xx | cooldown | deregistration delay | 概念混淆 | 3:09 |
| 7 | Q14 渲染临时存储 | io2 EBS | **instance store** | 选项缺失 | 2:07 |
| 8 | Q17 DynamoDB 加速 | ElastiCache | **DAX** | 映射粒度粗 | 2:14 |
| 9 | Q18 VPC 数据库安全 | Shield + DX | 私有子网 + SG 引用 | 🔴 Shield 边界 | 4:41 |

---

# 🔑 三条最高价值规律（覆盖 7/9 错题）

## ① 「授权句」模式 —— 命中 Q9 / Q14 / Q17

题干用一句 **"不需要 XX"** 解除某服务的固有限制，从而让它成为最优解。

| 题 | 授权句 | 解锁了什么 |
|---|---|---|
| Q9 | self contained and **does not need to access any databases** | Pilot Light 简化为"EC2 全关机" |
| Q14 | temporary data that is **discarded** after rendering | 允许用会丢数据的 instance store |
| Q17 | application **does not require consistent reads** | 允许用只缓存最终一致读的 DAX |

**✅ 强制动作**：读到 `does not require` / `does not need` / `is not required` / `discarded` / `can be lost` →
**停一秒，问「这句放宽了什么限制？哪个选项因此变得可用？」**

## ② 「数字比对」模式 —— 命中 Q8 / Q10

题干给的时间/容量数字，是让你换算后去和某个服务参数比对的。

| 题 | 题干数字 | 和谁比 | 结论 |
|---|---|---|---|
| Q8 | 最长 10 分钟、4 GB 内存 | Lambda 上限 15 分钟 / 10 GB | 装得下 → 选 Lambda |
| Q10 | 请求耗时 15 分钟 | deregistration delay 默认 **300 秒** | 不够 → 调到 > **900 秒** |

**✅ 强制动作**：题干出现时间/容量数字 → **先换算成选项使用的单位**（分钟→秒），再找配对。
⚠️ 同一个数字在不同上下文含义相反，要记的是"**和谁比**"，不是"看到 X 选 Y"。

## ③ 🔴 Shield 边界 —— 错 2 次，占失分 22%

| 题 | 我用 Shield 想解决 | 实际该用 |
|---|---|---|
| Q4 | XSS / SQL 注入 | **WAF** |
| Q18 | 限制网络访问来源 | **私有子网 + 安全组** |

> **Shield 只做一件事：抗 DDoS（流量型攻击）。**
> 其余任何安全需求，出现 Shield 一律排除。
> 补充：**Shield 不是网络设备，不能作为路由表的 target。**

---

# 逐题知识巩固

## Q1 · 容器 + 解耦（D2 Task 2.1）

**四个约束逐条筛**：

| 题干 | 筛掉 |
|---|---|
| 处理需 **30 分钟** | Lambda（15 分钟上限） |
| **containerized** applications | 指向 ECS / EKS |
| **avoid managing the underlying infrastructure** | ECS on EC2、EKS 托管节点组 |

### ⚠️ EKS managed node groups ≠ 免运维
"managed" 托管的是**节点生命周期**，节点本身仍是**你账号里的 EC2**。
👉 **容器 + 免运维 = Fargate**，没有第二个答案。

### ⚠️ SNS 不能用于任务分发

```
SNS（扇出）  每个订阅者收到【全部】消息  → 3 个容器各处理同一视频一遍
SQS（队列）  每条消息只被【一个】消费者取走 → 才是负载分担
```

### 容器方案选择表

| 需求关键词 | 答案 |
|---|---|
| 容器 + avoid managing infrastructure / serverless | **Fargate** |
| 容器 + 需控制实例类型 / GPU / 更低成本 | ECS on EC2、EKS 节点组 |
| 明确提到 **Kubernetes** | EKS |
| 容器 + AWS 原生最简单 | ECS |
| 丢个镜像就能跑（含 HTTPS + 伸缩） | App Runner |

⚠️ **题干没提 Kubernetes 就别选 EKS。** 本卷 EKS 出现 2 次，2 次都是干扰项。

### SQS / SNS / EventBridge 三选一

| 场景 | 选 |
|---|---|
| 分发任务给一组 worker，每条只处理一次 | **SQS** |
| 削峰填谷、缓冲突发、解耦 | **SQS** |
| 一条消息要多个下游都收到 | **SNS** |
| 多下游都要收到 **且各自需缓冲/重试** | **SNS → 多个 SQS**（扇出） |
| 按事件内容路由 / SaaS 事件 / 定时 | **EventBridge** |

### Lambda 硬边界

| 项目 | 上限 |
|---|---|
| **执行时间** | **15 分钟** ← 最常考 |
| 内存 | 10,240 MB |
| `/tmp` | 10,240 MB |
| 部署包（解压后） | 250 MB |
| 容器镜像 | 10 GB |

---

## Q4 · 防护五件套（D1）

**XSS + SQL injection = 应用层（L7）内容攻击 = WAF。**

### 按攻击类型查表

| 题干出现 | 答案 |
|---|---|
| **SQL injection / XSS / 恶意 bot / 地理封禁 / IP 限速** | **AWS WAF** |
| **DDoS / 流量洪泛 / SYN flood** | **Shield** |
| 跨多账号统一下发 WAF / SG 规则 | **Firewall Manager** |
| VPC 级流量过滤、出站域名白名单、IPS/IDS | **Network Firewall** |
| **发现**异常行为（挖矿、被盗凭证） | **GuardDuty**（只报警不拦截） |
| **扫漏洞**（EC2 / 镜像 / Lambda 的 CVE） | **Inspector** |
| **找 S3 里的敏感数据** | **Macie** |

> **WAF = Web Application Firewall** → 管应用层内容
> **Shield = 盾** → 挡砸过来的流量
> **GuardDuty / Inspector / Macie 只"看"不"拦"** → 要求 block 时全是干扰项

### 两个干扰手法

1. **「容器 + 内容」结构**：`Define an AWS **Shield Advanced** policy in AWS Firewall Manager`
   Firewall Manager 是对的，但里面装的 Shield Advanced 不防 XSS。
   👉 **别只看外壳对不对，要看里面装的东西对不对。**

2. **「把托管服务说成要装在机器上」**：`Deploy AWS Firewall Manager **on the EC2 instances**`
   同类：`Install AWS WAF on EC2` / `Deploy GuardDuty agents` / `Install CloudFront on origin`
   👉 看到 **install / deploy XX on the EC2 instances**，先问这服务是不是托管的。

### WAF 挂载位置

可挂：**CloudFront、ALB、API Gateway、AppSync、Cognito User Pool、App Runner**
❌ 不能挂 **NLB**（L4 不解析 HTTP）、不能直接挂 EC2
**架构里同时有 CloudFront 和 ALB 时 → 优先挂 CloudFront**（边缘拦截，恶意流量不进 VPC）

---

## Q7 · DocumentDB 认证（D1 Task 1.1）· **真知识缺口**

### 控制平面 vs 数据平面

```
控制平面 Control Plane —— 管「数据库这个资源」
  创建集群 / 改规格 / 删除 / 备份 / 打标签     ✅ IAM 管得了

数据平面 Data Plane —— 管「数据库里的数据」
  连接 / 查询 / 插入 / 删除                   ❌ DocumentDB 只认用户名密码
```

### 各数据库认证方式

| 数据库 | IAM 管控制平面 | **IAM 能直接认证数据库连接吗** |
|---|---|---|
| **DynamoDB** | ✅ | ✅ **纯 IAM**，没有"数据库用户"概念 |
| **RDS MySQL / PostgreSQL** | ✅ | ✅ IAM database authentication |
| **Aurora MySQL / PostgreSQL** | ✅ | ✅ |
| RDS for Oracle / SQL Server | ✅ | ❌ |
| **DocumentDB** | ✅ | ❌ **只能用户名密码** |
| Redshift | ✅ | ✅ 可用 IAM 换临时凭据 |

> **记忆逻辑**：DynamoDB 是 AWS 原生 → 纯 IAM；**兼容开源引擎的**（MongoDB / Oracle / SQL Server）受限于引擎自身认证机制，IAM 插不进去。

### DocumentDB 本身

| 项目 | 内容 |
|---|---|
| 是什么 | 兼容 **MongoDB** 的托管文档数据库，存 JSON |
| 架构 | 类 Aurora：存算分离，存储自动扩容，最多 15 只读副本 |
| **题干关键词** | **MongoDB / document database / JSON documents / 从 MongoDB 迁移** |
| 网络 | 必须在 VPC 内 |

### ⚠️ IAM policy 不会"执行"任何动作

`IAM policy 自动更新密码` 是虚构能力。**IAM policy 只回答"允不允许"。**
想自动轮换凭据 → **Secrets Manager**。

👉 **通用启发**：三个选项都在承诺"消除/自动化/一步到位"，只有一个朴素地说"这部分还得用传统方式"时，**往往朴素的那个对**。

---

## Q8 · 间歇性负载降本（D3 Task 3.2）

**题干三个数字 = Lambda 适配性检查表**：

| 题干 | Lambda 限制 | 结论 |
|---|---|---|
| 最长 **10 分钟** | ≤ 15 分钟 | ✅ |
| 内存 **4 GB** | ≤ 10,240 MB | ✅ |
| 一天随机跑几次 | 事件驱动 | ✅ 主场 |
| 实例**常开** | — | 🔴 为 20 分钟的活付 24 小时钱 |

**成本约算**：EC2 常开 ≈ $70/月 → Lambda ≈ $2.4/月（降 95%+）

### ⚠️ `randomly` 直接杀死"定时启停"
Instance Scheduler 是按**固定时间表**启停。随机请求到来时实例可能是停的 → **应用不可用**。
👉 `randomly` / `unpredictable` / `sporadic` 出现 → 所有"定时启停/预留容量"方案出局。

### AWS Instance Scheduler 的正确场景
**开发/测试环境**在**非工作时间**自动停机。
关键词：`dev/test`、`business hours`、`nights and weekends`、`Mon–Fri 9am–6pm` —— 共同点是**时间可预测**。

### 间歇性负载的计算服务阶梯

```
Lambda      事件驱动、≤15分钟、无状态        最省
  ↓ 超时/需长驻
Fargate     容器、无时长限制、免运维
  ↓ 需控制实例
EC2 + ASG   特定机型 / GPU / 稳定长跑
```

### ⚠️ 纠正：EC2 **按秒计费**（最少 60 秒），不是按小时
官方解析写的 "hourly basis" 是旧说法。结论不变（随机触发下无法精准启停）。

### 🔴 行为模式：打补丁 vs 换底座

| 题 | 我选 | 正确 |
|---|---|---|
| Q1 | EKS 节点组（继续管 EC2） | ECS on **Fargate** |
| Q8 | Instance Scheduler（给 EC2 加开关） | 重构为 **Lambda** |

**两次都选了"在现有架构上打补丁"，答案都是"换一个更托管的服务"。**
👉 题目问 `reduce costs the MOST` / `most cost-effective` / `least operational overhead` 时，
**"refactor / migrate to 更托管的服务" 优先级高于 "给现有方案加管理工具"。**
⚠️ 但若题目问 `least development effort`，判断反转 —— **题目问什么就优化什么，别自己加约束。**

---

## Q9 · DR 策略（D2 Task 2.2）

### 区分 Pilot Light 和 Warm Standby 的唯一标准：**资源是关着还是开着**

```
Pilot Light   EC2 已创建但 ⏹ STOPPED     → 故障时启动 → 切 100% 流量
Warm Standby  EC2 ▶️ RUNNING（规模缩小）  → 故障时扩容 → 切 100% 流量
```

### 四种 DR 策略完整表

| 策略 | RTO / RPO | 成本 | 云上状态 | 关键词 |
|---|---|---|---|---|
| **Backup & Restore** | **小时~天级** | 💰 最低 | 只有备份数据 | `lowest cost`、`hours acceptable` |
| **Pilot Light** | **十分钟级** | 💰💰 | 核心资源**已建但关机** | `stopped`、`turn off`、`core infra in place` |
| **Warm Standby** | **分钟级** | 💰💰💰 | **缩小版全套在跑** | `scaled-down but functional`、`always running` |
| **Multi-Site Active/Active** | **秒级/近零** | 💰💰💰💰 | **两地全量运行** | `zero downtime`、`active-active` |

```
Backup & Restore  →  只有数据    （什么都没建）
Pilot Light       →  建好了，关着 （火种留着）
Warm Standby      →  开着，但小   （小火慢炖）
Multi-Site        →  开着，全量   （两个都是主力）
```

**判据**：给 RTO/RPO 数字 → 对号入座；强调 `lowest cost` → Backup & Restore；强调 `zero downtime` → Multi-Site；中间两个靠**跑没跑**区分。

⚠️ **DR 是 SAA-C03 Domain 2 的正经考点，不是 SAP 专属**（README 里曾误归类到 SAP）。

---

## Q10 · ASG / ALB 四个"等待时间"（D2 Task 2.1）

**题干 15 分钟 = 900 秒 → 选项写 "greater than 900 seconds"，显式配对。**

### ✅ Deregistration Delay（注销延迟 / 连接排空）

**归属：ALB/NLB 的目标组**

```
ASG 决定缩容，实例 X 开始注销
  ├─ ALB 立刻停止发送【新请求】
  └─ X 上【正在处理的请求】继续跑 ⏳ 最长 = deregistration delay
        → 时间到 或 请求完成 → 实例真正终止
```
**默认 300 秒，范围 0–3600 秒。**

### ❌ Cooldown Period（我选的）

**归属：ASG。** 一次伸缩后暂停一段时间再做下次伸缩决策，**防止震荡**。
**与"正在处理的请求"毫无关系。**

### 四个时间设置对照

| 名称 | 设在哪 | 解决什么 | 默认 |
|---|---|---|---|
| **Deregistration delay** | **目标组** | 缩容时不掐断进行中的请求 | 300 秒 |
| **Cooldown period** | ASG | 防止伸缩震荡 | 300 秒 |
| **Health check grace period** | ASG | 新实例启动慢，别刚起来就判 unhealthy | 300 秒 |
| **Lifecycle hook** | ASG | 终止前/启动后插入自定义操作 | 需配置 |

| 题干症状 | 答案 |
|---|---|
| 缩容时用户收到 **5xx / 请求被中断** | **Deregistration delay** 调大 |
| 实例**反复扩了又缩** | **Cooldown period** 调大 |
| 新实例**刚启动就被判 unhealthy 替换** | **Health check grace period** 调大 |
| 终止前要**保存日志 / 优雅下线** | **Lifecycle hook** |

### 干扰项
- **sticky sessions**：保证"路由亲和性"，不保证"请求不被中断"。实例没了照样转走。
- **增大实例规格**：官方原话 *"does not **directly ensure**"*。
  👉 **"减少发生概率" 永远输给 "从机制上保证"。**

---

## Q14 · Instance Store（D4 Task 4.1）

### 三步排除

| 题干 | 作用 |
|---|---|
| **40,000 random IOPS** | ❌ st1（HDD 吞吐型，随机 IOPS ~500）❌ S3（对象存储，非块存储） |
| temporary data that is **discarded** | ✅ 解锁 instance store |
| **MOST cost-effective** | ✅ instance store 不额外收费 → 击败 io1/io2 |

**成本约算（us-east-1）**：
```
io2：400 GB × $0.125                    ≈    $50
     32,000 IOPS × $0.065
   +  8,000 IOPS × $0.046               ≈ $2,448
   ────────────────────────────────────────────
                                        ≈ $2,500 / 月 💸
instance store                               $0     ✅
```

👉 **`gp3/gp2/st1/sc1` 只按 GB 收费；`io1/io2` 按 GB + 按预置 IOPS 双重收费。**
**题目说 cost-effective 又要很高 IOPS → io1/io2 基本是陷阱。**

### Instance Store 完整属性（原笔记漏了"免费"这条）

| 属性 | 内容 |
|---|---|
| 是什么 | 物理直连宿主机的本地 NVMe SSD / HDD，不走网络 |
| 性能 | 🔥 最高档，数十万随机 IOPS、微秒级延迟 |
| **💰 费用** | **$0 额外费用**，已含在实例小时价里 ← **决胜点** |
| **⚠️ 持久性** | **临时**。**stop / terminate / 宿主机故障 → 数据全丢**（仅 reboot 不丢） |
| 限制 | 不能做快照、不能分离挂到别的实例、容量由实例类型固定 |
| 适用 | 缓存、临时文件、scratch、buffer、可重建数据 |

> **核心权衡：用持久性换性能和成本。**

### 块存储决策树

```
数据丢了会怎样？
  ├─ 无所谓（临时/可重建） ─→ Instance Store  ⚡最快 💰免费
  └─ 必须保住 ─→ EBS
                 ├─ 高随机 IOPS（数据库） ─→ io2 / io1
                 ├─ 通用（默认）          ─→ gp3
                 ├─ 大文件顺序吞吐         ─→ st1
                 └─ 冷数据最便宜           ─→ sc1
```

### EBS 类型参数

| 类型 | 介质 | 最大 IOPS | 最大吞吐 | 计费 | 启动盘 |
|---|---|---|---|---|---|
| **gp3** | SSD | 16,000 | 1,000 MB/s | 按 GB（基线 3,000 IOPS 免费） | ✅ |
| gp2 | SSD | 16,000 | 250 MB/s | 按 GB | ✅ |
| **io2 / io1** | SSD | 64,000（io2 Block Express 达 256,000） | 1,000–4,000 MB/s | **按 GB + 按预置 IOPS** 💸 | ✅ |
| **st1** | HDD | ~500 | 500 MB/s | 按 GB | ❌ |
| sc1 | HDD | ~250 | 250 MB/s | 按 GB（最便宜） | ❌ |

👉 **`random IOPS` → SSD；`sequential throughput / MB/s / 大文件` → HDD 可选。**

### 实例族字母

| 字母 | 含义 | 场景 |
|---|---|---|
| T | 突增型（CPU 积分） | 开发测试、小流量 Web |
| M | 通用平衡 | 应用服务器 |
| C | **C**ompute 计算优化 | 批处理、HPC |
| R | **R**AM 内存优化 | 内存数据库、缓存 |
| **I** | **I**OPS 存储优化（NVMe SSD） | **高随机 IOPS + 本地临时盘** |
| D / H | 存储优化（大容量 HDD） | 数据仓库、分布式文件系统 |
| G / P / Inf / Trn | GPU / ML 加速 | 训练、推理、渲染 |

> **"storage optimized" = I / D / H 族**，卖点就是自带高性能本地盘。

---

## Q15 · Serverless 全家桶（✅ 答对，2:34）

**三个约束各指一处**：`OLTP` → RDS/Aurora ｜ `available at all times` → 不能停 ｜ `unpredictable` + `minimum cost during idle` → Serverless
→ **ECS on Fargate + Aurora Serverless**

### OLTP vs OLAP

| | **OLTP** | **OLAP** |
|---|---|---|
| 全称 | Online **Transaction** Processing | Online **Analytical** Processing |
| 干什么 | **交易**：下单、转账、改资料 | **分析**：报表、BI、趋势 |
| 查询 | 大量**小**事务，操作几行 | 少量**大**查询，扫描聚合百万行 |
| 存储 | **行式** | **列式** |
| AWS | **RDS / Aurora**、DynamoDB | **Redshift**、Athena |

| 关键词 | 选 |
|---|---|
| **OLTP**、transaction、订单、库存 | RDS / Aurora |
| **OLAP**、data warehouse、analytics、BI、reporting、PB 级 | **Redshift** |
| 直接查 S3 文件、不想建数仓、按查询量付费 | **Athena** |

⚠️ 把 **Redshift 塞进 OLTP 场景**是架构方向性错误，比"贵一点"严重得多。

### Serverless 对照表

| 传统 | Serverless 版本 | 空闲时费用 |
|---|---|---|
| EC2（事件驱动、≤15分钟） | **Lambda** | $0 |
| ECS on EC2（容器常驻） | **Fargate** | 任务缩到 0 即 $0 |
| RDS / Aurora 预置 | **Aurora Serverless** | 按 ACU 用量 |
| DynamoDB 预置容量 | **DynamoDB On-Demand** | 按请求数 |
| Redshift 预置集群 | **Redshift Serverless** | 按 RPU |
| EMR 集群 | **Athena / Glue** | 按扫描量 / DPU |

**触发词**：`unpredictable` / `intermittent` / `spiky` / `idle periods` / `pay only for what you use`

### Lambda vs Fargate 判据

| | Q8（选 Lambda） | Q15（选 Fargate） |
|---|---|---|
| 负载 | 一天跑两次，各 10 分钟 | 持续可用，流量不可预测 |
| 要求 | — | **available at all times** |
| 形态 | 一段处理逻辑 | 完整 application layer |

> **事件驱动、跑完就结束、≤15 分钟 → Lambda**
> **需长期在线的服务、容器化、负载弹性 → Fargate**

### ⚠️ 看到 delete / terminate the database 先问"数据去哪了"
除非题干明确授权数据可丢弃，否则导致数据丢失的选项一律排除。

---

## Q17 · DAX（D3 Task 3.3）

> 📌 **这题笔记里被自己标注为「送分题」**（`notes/saa/05-dynamodb.md:143`），仍然做错 —— 纯粹的提取失败。

### 四个信号全指向 DAX

| 题干 | 作用 |
|---|---|
| 数据库是 **DynamoDB** | DAX 是 DynamoDB 专属加速器 |
| slow data load times / improve **latency** | 毫秒 → **微秒** |
| **does not require consistent reads** | 🔑 **授权句** |
| **LEAST operational overhead** | 只需换 client endpoint |

### 授权句为什么是钥匙

```
最终一致读（默认）→ DAX 缓存命中 → ⚡ 微秒级
强一致读           → DAX【不缓存】，穿透到 DynamoDB → DAX 白装
```
题干说"不需要一致性读"，就是在**解除 DAX 的固有限制**。

### DAX 完整属性

| 属性 | 内容 |
|---|---|
| 全称 | **D**ynamoDB **A**ccelerator |
| 服务对象 | **仅 DynamoDB** |
| 性能 | 个位数毫秒 → **微秒**级 |
| 形态 | 托管集群，跑在你的 **VPC** 内 |
| 写模式 | **Write-through**（写经 DAX 透传到 DynamoDB） |
| 两种缓存 | **Item cache**（GetItem/BatchGetItem）+ **Query cache**（Query/Scan） |
| 代码改动 | ⭐ 极小，API 完全兼容，换 client + endpoint |
| **⚠️ 限制** | **强一致读不走缓存** |

### 缓存服务选择

| 场景 | 选 |
|---|---|
| 给 **DynamoDB** 加缓存，最少改动 | **DAX** ⭐ |
| 给 **RDS / Aurora** 加缓存 | **ElastiCache** |
| 缓存非数据库数据（会话、排行榜、API 响应） | **ElastiCache** |
| 加速静态内容 / 全球就近访问 | **CloudFront** |
| 需要**强一致读** | ❌ 缓存都用不了，直接读源 |

> **DynamoDB 的亲儿子是 DAX**，别人家的缓存是 ElastiCache。

### ElastiCache 两引擎

| | **Redis** | **Memcached** |
|---|---|---|
| 数据结构 | 丰富（list/set/sorted set/hash） | 简单 K-V |
| **持久化** | ✅ | ❌ |
| **高可用 / 故障转移** | ✅ 副本 + Multi-AZ | ❌ |
| 备份快照 | ✅ | ❌ |
| 多线程 | ❌ 单线程 | ✅ 吃满多核 |

> 要**持久化/高可用/复杂数据结构/排行榜** → **Redis**
> 只要**最简单缓存 + 多核吞吐 + 能随便丢** → **Memcached**

### 🔴 失败模式：服务映射粒度太粗

```
我的映射：  「要加缓存」        → ElastiCache        ❌ 少一个限定维度
正确映射：  「给 DynamoDB 加缓存」 → DAX
            「给 RDS 加缓存」      → ElastiCache
```
**和 Q14 同病**：那题是「要高 IOPS → io2」，正确的是「高 IOPS **且数据可丢** → instance store」。

---

## Q18 · VPC 三层安全（D1 Task 1.2）· Multi-Select

**两个要求 = 两个答案**：

| 题干要求 | 措施 |
|---|---|
| **cannot be accessed from the internet** | **B：DB subnet group 只含私有子网** |
| accessible **from the application tier** over a **specified port only** | **C：DB 安全组入站 源=应用层安全组 + 数据库端口，删除其他规则** |

```
Internet → [Public Subnet: ALB/NAT]  有 IGW 路由
              ↓
           [Private Subnet: App  SG-App]
              ↓ 仅 3306
           [Private Subnet: RDS  SG-DB]   ← B: DB Subnet Group 只放私有子网
                                          ← C: 入站 源=SG-App, 端口=3306
```

### ⭐ 安全组**引用安全组**（核心模式，必考）

```
SG-DB 入站规则
┌────────┬──────┬──────────────────────┐
│ MySQL  │ 3306 │ sg-0abc123 (SG-App)  │  ← 引用 SG，不是 CIDR
└────────┴──────┴──────────────────────┘
```

| 写法 | 应用层扩容/换 IP 时 |
|---|---|
| 源 = CIDR `10.0.1.0/24` | 网段变了要改规则 |
| 源 = **安全组 ID** | ✅ **自动生效** |

👉 题干出现 **"only from the application tier / web tier"** → **答案几乎必然是"安全组引用安全组"**。
👉 `Remove other inbound rules` 体现最小权限 —— **多选题里带"清理多余权限"的选项通常是对的。**

### 私有子网的定义

**私有子网 = 路由表里没有指向 Internet Gateway 的路由。** 不看名字，只看路由表。

| 类型 | 路由表关键条目 | 效果 |
|---|---|---|
| **公有子网** | `0.0.0.0/0 → igw-xxx` | 双向访问互联网 |
| **私有子网** | 无 IGW 路由（可有 `0.0.0.0/0 → nat-xxx`） | 互联网**进不来**；经 NAT 可单向出去 |

**RDS 落在私有子网靠的是 DB Subnet Group。**

### SG vs NACL

| | **Security Group** | **Network ACL** |
|---|---|---|
| 层级 | **实例 / ENI** | **子网** |
| 状态 | **有状态**（回程自动放行） | **无状态**（出入站分别配） |
| 规则 | **只有 Allow** | Allow **和 Deny** |
| 匹配 | 所有规则一起看，任一允许即通过 | **按编号从小到大**，首条匹配生效 |
| 引用安全组 | ✅ | ❌ 只能 CIDR |
| 默认 | 入站全拒，出站全放 | 默认全放行 |

👉 **要"拒绝特定 IP"只能用 NACL**（SG 没有 Deny）。

### 两个错选

- **Shield + 改路由表**：Shield 只抗 DDoS，且 **Shield 不是网络设备，不能作为路由表 target**。
- **Direct Connect**：DX / Site-to-Site VPN 的**前提是一端为 on-premises**。
  👉 题干没有 `on-premises / 混合云 / 本地数据中心` → DX / VPN 一律排除。

### 多选题应试技巧
1. **先数题干有几个独立要求** —— 答案数量通常与之匹配（本题 2 个要求 → 选 2 个）
2. **先排除明显不可能的**（Shield、DX 可秒杀），剩下的再按"要求↔动作"配对
3. 多选必须**全对才得分**，选对一半 = 0 分

---

# 后续行动

## 立即执行的三个动作

1. **读到 `does not require / does not need / discarded / can be lost`** → 停一秒问「这放宽了什么限制？」
2. **读到时间/容量数字** → 先换算单位，再找它该和谁比
3. **看到 Shield** → 除非题干是 DDoS，否则直接排除

## 复习顺序（按遗忘程度反向）

| 优先级 | 笔记 | 理由 |
|---|---|---|
| 🔴 最高 | `01-aws-basics-ec2` ~ `04-database-rds-aurora` | **4 月学的，忘得最多**（Q14 instance store 即出自此） |
| 🟡 中 | `05-dynamodb` ~ `09-vpc-connectivity` | 9 月初学，部分遗忘（Q17 DAX、Q18 VPC） |
| 🟢 低 | `10` ~ `17` | 9 月中旬学，记忆较新 |

## 域优先级（按 权重 × 差距）

| 顺序 | Domain | 权重 | 得分 | 差距 × 权重 |
|---|---|---|---|---|
| 1️⃣ | **D1 安全** | 30% | 40% | **18.0** |
| 2️⃣ | **D2 弹性** | 26% | 40% | **15.6** |
| 3️⃣ | D3 高性能 | 24% | 60% | 9.6 |
| ⏭️ | D4 成本 | 20% | 80% | 4.0（**略过**） |
