# CloudFormation / Beanstalk + 成本优化

> P5 收官 · 2026-09-14（原定 9/18，提前 4 天）
> **Domain 4「Design Cost-Optimized Architectures」占 20%**

---

## 1. CloudFormation —— 基础设施即代码

用 YAML/JSON 模板声明资源，AWS 负责创建、更新、删除。
**核心价值：可重复、可版本控制、可回滚。**

### 模板的主要部分

| 段 | 用途 |
|---|---|
| **Parameters** | 输入变量（实例类型、环境名等） |
| **Resources** | 🚨 **唯一必填** —— 要创建的资源 |
| **Mappings** | 静态查找表（如各 Region 的 AMI ID） |
| **Outputs** | 输出值，🔑 **可被其他 Stack 引用（Cross-Stack Reference）** |
| **Conditions** | 条件创建（如仅 prod 环境才建备用实例） |

### ⭐ 四个必考概念

| 概念 | 说明 |
|---|---|
| **Change Sets** | 🔑 **更新前预览**「哪些资源会被改/删/替换」，确认后再执行 |
| **StackSets** | 🔑 **一次把同一模板部署到多个账户 + 多个 Region**（配合 Organizations） |
| **Drift Detection** | 🔑 检测有人**手动改了**资源，导致实际状态偏离模板 |
| **Nested Stacks** | 把公共组件（如标准 VPC）抽成子模板，供多个父栈复用 |

### 删除保护

- `DeletionPolicy: Retain` —— 删栈时**保留**该资源
- `DeletionPolicy: Snapshot` —— 删前建快照（用于 RDS / EBS）

> 🔑 判据
> `多账户多 Region 统一部署` → **StackSets**
> `更新前想知道影响` → **Change Sets**
> `有人手动改了配置，想发现` → **Drift Detection**

### 🚨 Drift Detection vs AWS Config

两者都能发现「配置被改了」，但**目的不同**：

```
AWS Config       问：改成的样子【合不合规】
Drift Detection  问：改得【和模板一致吗】
```

> 题干说「**CloudFormation 管理的**资源被手动修改」→ **Drift Detection**

---

## 2. Elastic Beanstalk —— 平台即服务

你只上传**代码**，Beanstalk 自动搞定 EC2、ALB、ASG、容量、监控、部署。

- 支持 Java / Python / Node / .NET / PHP / Ruby / Go / Docker
- 🔑 **底层资源你仍然拥有并可控**（不像 App Runner 那样完全黑盒）
- **Beanstalk 本身免费**，只付底层资源的钱

### ⭐ 部署策略（考点）

| 策略 | 停机 | 特点 |
|---|---|---|
| **All at once** | 🚨 **有停机** | 最快、最便宜 |
| **Rolling** | 无 | 分批更新，**期间容量下降** |
| **Rolling with additional batch** | 无 | 先加新实例再替换，**容量不下降**，多花钱 |
| **Immutable** | 无 | 🔑 **全新 ASG 起全量新实例**，最安全、**回滚最快** |
| **Blue/Green** | 无 | 起一套全新环境，🔑 **切换 CNAME（Swap URL）** |

> 🔑 判据
> `零停机 + 最快回滚` → **Immutable**
> `完全隔离的新环境再切流量` → **Blue/Green**

---

## 3. ⭐ 成本优化：EC2 四种购买方式

| 方式 | 折扣 | 适合 |
|---|---|---|
| **On-Demand** | 0% | 短期、不可预测、测试 |
| **Reserved Instance（RI）** | 最多 **72%** | 🔑 **稳定、可预测的长期负载**（1 或 3 年） |
| **Savings Plans** | 最多 **72%** | 🔑 **承诺每小时消费额**，比 RI **更灵活** |
| **Spot** | 最多 **90%** | 🚨 **可被 2 分钟通知中断** → 容错型、无状态、批处理 |
| **Dedicated Host** | — | 合规要求**物理隔离**、自带许可（BYOL） |

### RI vs Savings Plans（高频对比）

```
RI：           承诺买【特定实例类型 + Region】
               → 折扣高，但换实例类型就不划算
               → Standard（折扣高、不可改）/ Convertible（可换类型、折扣略低）

Savings Plans： 承诺【每小时花多少钱】（如 $10/hr）
               → 实例类型、大小、OS、Region 随便换，自动抵扣
               → Compute SP 最灵活（🔑 还覆盖 Lambda 和 Fargate！）
               → EC2 Instance SP 折扣更高但锁定实例族
```

> 🔑 判据
> `稳定负载 + 最大折扣` → **RI 或 EC2 Instance SP**
> `想要折扣但保留灵活性 / 还想覆盖 Lambda 和 Fargate` → **Compute Savings Plans**
> `可中断、容错、批处理、大幅降本` → **Spot**

---

## 4. 成本管理工具四件套

| 工具 | 用途 |
|---|---|
| **Cost Explorer** | 🔑 **可视化分析历史成本 + 预测未来**，给 RI/SP 购买建议 |
| **AWS Budgets** | 🔑 **设预算阈值，超了发告警**（按成本/用量/RI 利用率） |
| **Cost and Usage Report (CUR)** | 最详细的原始账单数据 → S3，可用 Athena/QuickSight 分析 |
| **Cost Anomaly Detection** | ML 检测异常花费突增 |

> 🔑 判据：`分析钱花在哪 / 预测` → **Cost Explorer**；`超支要告警` → **Budgets**

---

## 5. ⭐ 各服务降本手段速查（高频场景题）

```
S3      → 生命周期策略转 IA/Glacier
          Intelligent-Tiering（🔑 访问模式【未知/不规律】时）
EBS     → gp2 换 gp3（同性能便宜 20%）、删未挂载的卷和旧快照
EC2     → RI/SP（稳定）、Spot（容错）、Right Sizing（Compute Optimizer）
数据传输 → VPC Endpoint 省 NAT GW 流量费
          CloudFront 缓存省源站出网费
          同 AZ 通信免费，跨 AZ 收费
NAT GW  → 用 Gateway Endpoint 访问 S3/DynamoDB（🔑 免费）
RDS     → RI、停止非生产实例、Aurora Serverless v2（波动负载）
Lambda  → 调优内存找性价比点（Power Tuning）
日志    → CloudWatch Logs 设保留期（🚨 默认永久！）
```

---

## 6. ⭐ 成本题的三个母题

SAA 的成本题几乎不考「算钱」，而是考「认出哪个方案更便宜」：

### ① 「负载可中断吗？」
能中断 → **Spot**（省 90%）。批处理、渲染、CI/CD、无状态 Web 层都算。

### ② 「访问模式知道吗？」（S3 专属）
```
知道（如 30 天后不再访问） → 生命周期策略
不知道 / 不规律            → Intelligent-Tiering（自动分层，无检索费）
```

### ③ 「流量走公网了吗？」
最容易被忽视的成本项。私有子网经 NAT GW 访问 S3 = 付 NAT 处理费 + 流量费；
换成 **Gateway Endpoint 直接归零**。

---

## 7. 测验（12 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 公司有 50 个 AWS 账户，需要**把同一套安全基线模板部署到所有账户的 3 个 Region**。
A) 每个账户手动建 Stack　B) CloudFormation StackSets　C) Nested Stacks　D) Terraform

**Q2.** 更新生产环境的 CloudFormation 栈前，想**先知道哪些资源会被替换或删除**。
A) Drift Detection　B) Change Sets　C) Stack Policy　D) Rollback Configuration

**Q3.** 有人绕过 CloudFormation **手动修改了栈里的 Security Group**。如何发现这类偏离？
A) CloudTrail　B) AWS Config Rules　C) CloudFormation Drift Detection　D) Trusted Advisor

**Q4.** 删除 CloudFormation 栈时，希望**保留其中的 RDS 数据库不被删除**。
A) 设置 DeletionPolicy: Retain　B) 开启终止保护　C) 先手动导出数据　D) 用 Nested Stack

**Q5.** Beanstalk 部署新版本要求**零停机、且出问题时回滚最快**，成本不是主要考虑。
A) All at once　B) Rolling　C) Immutable　D) Rolling with additional batch

**Q6.** 视频转码作业**可以随时中断后重跑**，希望**最大幅度降低成本**。
A) On-Demand　B) Reserved Instance　C) Spot 实例　D) Dedicated Host

**Q7.** 公司有稳定的基础负载，但**未来一年可能更换实例类型**，同时还想让折扣**覆盖 Lambda 和 Fargate**。
A) Standard RI　B) Convertible RI　C) Compute Savings Plans　D) EC2 Instance Savings Plans

**Q8.** 财务希望**每月账单超过 $10,000 时收到告警**。
A) Cost Explorer　B) AWS Budgets　C) Cost Anomaly Detection　D) CloudWatch Alarm

**Q9.** 想**分析过去 6 个月成本构成并预测下季度支出**，还想获得 RI 购买建议。
A) AWS Budgets　B) Cost Explorer　C) CUR + Athena　D) Trusted Advisor

**Q10.** S3 中的数据**访问模式完全不可预测**，希望自动优化存储成本且**不产生检索费用**。
A) 生命周期策略转 Glacier　B) S3 Intelligent-Tiering　C) 全部放 S3 Standard-IA　D) 全部放 Glacier Deep Archive

**Q11.** 私有子网的 EC2 频繁访问 S3，账单中 **NAT Gateway 流量费很高**。最有效的降本方式？
A) 换更小的 NAT GW　B) 创建 S3 Gateway Endpoint　C) 把 EC2 移到公有子网　D) 改用 Interface Endpoint

**Q12.** 关于 Savings Plans 和 Reserved Instances，下列**错误**的是？
A) Compute Savings Plans 可覆盖 Lambda 和 Fargate　B) Standard RI 折扣高但不可更改实例类型　C) Savings Plans 承诺的是每小时消费额　D) Savings Plans 折扣上限远低于 RI，最多只有 30%
