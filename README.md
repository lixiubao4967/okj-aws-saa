# AWS 认证学习计划 | OKJ AWS SAA → SAP

> 目标：先通过 **AWS Certified Solutions Architect - Associate (SAA-C03)**，再进阶 **AWS Certified Solutions Architect - Professional (SAP-C02)**

---

## 学习路线总览

```
SAA-C03 (约 8-12 周)  →  SAP-C02 (约 8-12 周)
```

---

## 第一阶段：SAA-C03 Associate（约 10 周）

### 考试信息

| 项目 | 内容 |
|------|------|
| 考试代码 | SAA-C03（2026-09 核实：仍是当前版本，无 SAA-C04） |
| 题目数量 | 65 道（**50 题计分 + 15 题不计分**，不标识） |
| 考试时长 | 130 分钟（英文卷可申请 ESL 延时 → 160 分钟） |
| 通过分数 | 720 / 1000（补偿计分，单域不及格不影响） |
| 考试费用 | $150 USD（日本约 ¥20,000 + 税） |
| 有效期 | 3 年 |
| 考试方式 | Pearson VUE 考场 / OnVUE 在线监考 |

### ✅ 已报名（Status: Scheduled）

| 项目 | 内容 |
|------|------|
| **日期时间** | **2026-09-26（土）11:30 – 14:20 JST** |
| 考场 | Shinbashi Kokukaikan Test Center 2（東京都港区新橋 1-18-1 国際会館 2F, 105-0004） |
| 语言 | English |
| 时长 | **170 分钟**（130 + ESL 延时 30 + 流程时间） |
| 便利 | ESL Extra Time 30 Minutes（Approved，永久有效，SAP-C02 也自动生效） |
| 费用 | JPY 22,000（¥20,000 + 税），考过后公司报销 |

> Order # / Registration ID 见确认邮件，不记录在此仓库。

### 报名要点（备忘，供 SAP-C02 复用）

1. https://aws.amazon.com/certification/ → **AWS Certification Account**（AWS Builder ID 登录）→ Schedule an Exam → Pearson VUE
2. **先申请 ESL +30 分钟延时，再订考位** —— 英文卷且英语非母语可免费申请（130 → 160 分钟），必须在预约前于 Exam accommodations 里提交。已申请过，SAP 时不用再申请
3. 改期政策：考前 24h 以上免费改期，**每个考位最多改 2 次**；24h 内不可改期不可退款
4. CertMetrics 里的 First/Last Name 必须和证件罗马字一致（大小写不影响，姓名归属要正确）

### SAA-C03 考试域分布

| 领域 | 占比 |
|------|------|
| Domain 1: Design Secure Architectures | 30% |
| Domain 2: Design Resilient Architectures | 26% |
| Domain 3: Design High-Performing Architectures | 24% |
| Domain 4: Design Cost-Optimized Architectures | 20% |

### 🔥 9 月冲刺日历（2026-09-03 → 09-26）

> 原 10 周计划已完成 Week 1-4 + Week 5 一半。剩余 5.5 周内容压缩进 23 天。
> 关键窗口：**9/19(土) – 9/23(水) 五连休**（敬老日 + 秋分日 + 国民の休日），全部用于模拟题。

| 阶段 | 日期 | 内容 | 对应原计划 |
|------|------|------|-----------|
| **P1 补完数据库** ✅ | 9/3(木) – 9/7(月) | ✅ DynamoDB（分区键/GSI/LSI/DAX/TTL/容量模式）测验 5/6<br>✅ ElastiCache Redis vs Memcached 测验 **5/5**<br>✅ Redshift/Athena/Glue 测验 0/5 🚨 → 错题精读 → **重测 4/5** | Week 5 剩余 |
| **P2 网络（最高频）** ✅ | 9/7(月) – 9/8(火) | ✅ VPC 核心：CIDR/子网/路由表/IGW/NAT GW/SG vs NACL — 测验 **9/10**<br>✅ VPC 互联：Peering/TGW/VPN/DX/Endpoints/PrivateLink — 测验 **10/10**<br>**P2 合计 19/20（95%）**，比原计划**提前 3 天完成** | Week 6 |
| **P3 无服务器 + 集成** | 9/9(水) – 9/12(土) | ✅ Lambda + API Gateway — 测验 **10/10** 🎯<br>✅ SQS/SNS/EventBridge/Step Functions — 测验 **12/12** 🎯<br>🔸 Kinesis 三兄弟 + SQS/SNS/Kinesis 终极区分（笔记 `12-kinesis.md`，**8 题测验待做**） | Week 7 |
| **P4 安全 + 监控** | 9/15(火) – 9/17(木) | KMS/CMK/跨账户加密、Secrets Manager vs Parameter Store、CloudTrail、CloudWatch、Config、WAF/Shield/GuardDuty/Inspector/Macie | Week 8 |
| **P5 容器 + IaC + 成本** | 9/18(金) | ECS vs EKS vs Fargate、ECR、CloudFormation、Beanstalk、Cost Explorer/Budgets/Savings Plans | Week 9 |
| **P6 模拟题冲刺** ⭐ | 9/19(土) – 9/23(水) | 9/19 模拟卷① + 错题精读<br>9/20 错题知识点回炉 + 写 cheatsheets<br>9/21 模拟卷② + 复盘<br>9/22 薄弱域专项（按①②的域得分定）<br>9/23 模拟卷③ + 复盘 | Week 10 |
| **P7 收尾** | 9/24(木) – 9/25(金) | 9/24 通读 cheatsheets + 官方样题<br>9/25 只看错题本和速查表，早睡 | — |
| **考试** | **9/26(土)** | SAA-C03 | — |

**决策关卡：9/20 —— 若模拟卷① 正确率 < 70%，免费改期到 10/3（周六）。**

### 📍 下次从这里开始（更新于 2026-09-09）

1. **做 `notes/saa/12-kinesis.md` 末尾的 8 题测验** → P3 收工
2. 进入 **P4 安全 + 监控**（原定 9/15）：KMS、Secrets Manager vs Parameter Store、
   CloudTrail、CloudWatch、Config、WAF/Shield/GuardDuty/Inspector/Macie

**9/7 完成**：ElastiCache 5/5 ✅ ｜ 分析服务 0/5 → 错题精读 → 重测 4/5 ✅ **P1 收工**
**9/8 完成**：VPC 核心 **9/10** ✅ ｜ VPC 互联 **10/10** 🎯 **P2 收工（19/20 = 95%）**
**9/9 完成**：Lambda + API Gateway — 测验 **10/10** 🎯（连续两轮满分）
**9/10-9/11 完成**：SQS / SNS / EventBridge / Step Functions — 测验 **12/12** 🎯
**9/12 完成**：Kinesis 三兄弟内容已讲完 + P3 全部时长数字汇总表（8 题测验待做）

> 🔥 **连续三轮满分**（VPC 互联 10/10、Lambda+APIGW 10/10、应用集成 12/12）

> ➡️ **进度超前原计划约 3 天**，多出的时间全部并入 P6 模拟题冲刺（9/19-9/23 五连休）。

> 📌 **出题纪律（两次踩坑后的规则）**：给答题格式示例时必须用不可能是答案的串（如 `1A 2B 3C`），
> 且**选项里的加粗只能强调题干关键词，绝不标记正确答案**。笔记里的测验一律不写答案。

> ⚠️ 个人薄弱点提醒
> 1. **计算题必须写出三步，不要心算**（错题 4）
> 2. **服务能力边界要记准** —— 已暴露：Athena 联邦查询的用途、Glue Catalog 归谁用、DynamoDB 不能 join、KDS vs Firehose（错题 7-10）
> 3. 解题通法：先圈**频率词**（偶尔/持续/实时/每天）+ **约束词**（成本/运维/无代码）。两选项都「技术可行」时，判据是约束词

### 10 周学习计划（原始版本，供对照）

#### Week 1-2：AWS 基础 + 计算服务 ✅
- [x] AWS 全球基础设施（Region、AZ、Edge Location）
- [x] IAM：用户、组、角色、策略、MFA、STS
- [x] EC2：实例类型、购买选项（On-Demand/Reserved/Spot/Dedicated）
- [x] EC2 存储：EBS 类型、实例存储、EBS 快照
- [x] AMI、Auto Scaling Group、Launch Template

#### Week 3：高可用 + 负载均衡 ✅
- [x] ELB：ALB / NLB / GLB / CLB 区别与使用场景
- [x] Auto Scaling 策略（目标追踪、步进、计划）
- [x] Route 53：路由策略（Simple/Weighted/Latency/Failover/Geolocation/Multivalue）
- [x] CloudFront + S3 静态网站加速

#### Week 4：存储服务 ✅
- [x] S3：存储类型、生命周期策略、跨区复制（CRR/SRR）
- [x] S3 安全：Bucket Policy、ACL、Pre-signed URL、SSE-S3/KMS/C
- [x] EFS vs EBS vs S3 使用场景对比
- [x] Storage Gateway、Snowball/Snowcone/Snowmobile

#### Week 5：数据库服务（进行中）
- [x] RDS：Multi-AZ vs Read Replica、备份与恢复
- [x] Aurora：全局数据库、Serverless、Aurora vs RDS
- [x] DynamoDB：分区键、排序键、索引（GSI/LSI）、DAX、TTL、RCU/WCU 计算
- [x] ElastiCache：Redis vs Memcached、集群模式、缓存策略（测验 5/5）
- [x] Redshift、Athena、Glue 简介（测验 0/5 → 错题精读 → 重测 **4/5**）

#### Week 6：网络服务 ✅
- [x] VPC：子网（公/私）、路由表、Internet Gateway、NAT Gateway（测验 9/10）
- [x] Security Group vs NACL（测验 9/10）
- [x] VPC Peering、Transit Gateway、VPN、Direct Connect（测验 10/10）
- [x] VPC Endpoints（Gateway/Interface）+ PrivateLink（测验 10/10）

#### Week 7：应用集成 + 无服务器（进行中）
- [x] Lambda：触发器、并发、层（Layers）、边缘函数（测验 10/10）
- [x] API Gateway：三种类型、端点类型、授权方式、29 秒超时（测验 10/10）
- [x] SQS：标准 vs FIFO、可见性超时、长轮询、死信队列（DLQ）（测验 12/12）
- [x] SNS：扇出模式（Fan-out）+ 消息过滤（测验 12/12）
- [x] EventBridge（SaaS/cron/Replay）、Step Functions（Standard vs Express）（测验 12/12）
- [~] Kinesis：Data Streams / Firehose / Managed Flink（笔记已写，测验待做）

#### Week 8：安全 + 监控
- [ ] KMS：CMK、密钥轮换、跨账户加密
- [ ] Secrets Manager vs SSM Parameter Store
- [ ] CloudTrail、CloudWatch（Metrics/Logs/Alarms/Events）
- [ ] AWS Config、Trusted Advisor
- [ ] WAF、Shield、GuardDuty、Inspector、Macie

#### Week 9：容器 + 其他服务
- [ ] ECS vs EKS vs Fargate 选择场景
- [ ] ECR、App Runner
- [ ] CloudFormation：模板结构、StackSets、Change Sets
- [ ] Elastic Beanstalk、OpsWorks
- [ ] 成本优化：Cost Explorer、Budgets、Savings Plans

#### Week 10：冲刺 + 刷题
- [ ] 复习错题，重点覆盖薄弱领域
- [ ] 完成至少 3 套完整模拟题（65 题/套）
- [ ] 整理高频考点速查表

---

## 第二阶段：SAP-C02 Professional（约 10 周，SAA 通过后开始）

### 考试信息

| 项目 | 内容 |
|------|------|
| 考试代码 | SAP-C02 |
| 前提条件 | 持有效 AWS Associate 级别认证 |
| 题目数量 | 75 道（单/多选） |
| 考试时长 | 180 分钟 |
| 通过分数 | 750 / 1000 |
| 考试费用 | $300 USD |
| 有效期 | 3 年 |

### SAP-C02 考试域分布

| 领域 | 占比 |
|------|------|
| Domain 1: Design Solutions for Organizational Complexity | 26% |
| Domain 2: Design for New Solutions | 29% |
| Domain 3: Continuous Improvement for Existing Solutions | 25% |
| Domain 4: Accelerate Workload Migration and Modernization | 20% |

### SAP 进阶重点（相比 SAA 的深化方向）

- [ ] 多账户管理：AWS Organizations、SCP、OU 设计
- [ ] 混合云架构：Direct Connect、VPN、Transit Gateway 复杂拓扑
- [ ] 大规模数据迁移：Database Migration Service、Application Migration Service
- [ ] 高级安全架构：IAM 权限边界、ABAC、跨账户角色链
- [ ] 成本优化高级策略：预留实例组合、Savings Plans 最优化
- [ ] 灾难恢复策略：RTO/RPO 与 Backup/Pilot Light/Warm Standby/Multi-Site
- [ ] Well-Architected Framework 五大支柱深度理解
- [ ] 微服务 + 容器高级架构（Service Mesh、EKS 网络）
- [ ] 机器学习服务集成：SageMaker、Rekognition、Comprehend 架构应用

---

## 题库资源

### 免费资源

| 资源 | 说明 | 链接 |
|------|------|------|
| AWS 官方样题 (SAA) | 官方出品，最权威 | https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Sample-Questions.pdf |
| AWS 官方样题 (SAP) | 官方出品，最权威 | https://d1.awsstatic.com/training-and-certification/docs-sa-pro/AWS-Certified-Solutions-Architect-Professional_Sample-Questions.pdf |
| ExamTopics SAA-C03 | 社区维护，免费查看 | https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-associate-saa-c03/ |
| ExamTopics SAP-C02 | 社区维护，免费查看 | https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-professional-sap-c02/ |
| GitHub 开源题库 (650+ 题) | 含详细解析 | https://github.com/Iamrushabhshahh/AWS-Certified-Solutions-Architect-Associate-SAA-C03-Exam-Dump-With-Solution |

### 付费推荐（性价比高）

| 资源 | 说明 |
|------|------|
| [Tutorials Dojo](https://portal.tutorialsdojo.com/) | 最受推荐的模拟题平台，含详细解析，SAA 约 $15 |
| [Stephane Maarek (Udemy)](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03/) | 最佳视频课程，Udemy 打折时约 $15 |
| [Whizlabs](https://www.whizlabs.com/) | 题目质量高，含免费试题 |

---

## 学习笔记目录

```
okj-aws-saa/
├── README.md              # 本文件：总学习计划
├── notes/
│   ├── saa/               # SAA-C03 知识点笔记
│   │   ├── 01-aws-basics-ec2.md
│   │   ├── 02-high-availability-lb.md
│   │   ├── 03-s3-storage-security.md
│   │   ├── 04-database-rds-aurora.md
│   │   ├── 05-dynamodb.md
│   │   ├── 06-elasticache.md
│   │   ├── 07-analytics.md    # Redshift/Athena/Glue
│   │   ├── 08-vpc-core.md     # VPC/子网/路由表/IGW/NAT/SG vs NACL
│   │   ├── 09-vpc-connectivity.md  # Peering/TGW/VPN/DX/Endpoints
│   │   ├── 10-lambda-apigateway.md # Lambda 并发/VPC/Edge + API Gateway
│   │   ├── 11-sqs-sns-eventbridge-stepfunctions.md  # 应用集成
│   │   └── 12-kinesis.md      # Kinesis 三兄弟 + SQS/SNS/Kinesis 区分
│   └── sap/               # SAP-C02 进阶笔记
│       ├── 01-organizations.md
│       ├── 02-hybrid-network.md
│       └── 03-migration.md
├── practice/
│   ├── saa/               # SAA 错题本 + 模拟记录
│   │   ├── 01-week1-2-errors.md
│   │   ├── 02-week5-errors.md
│   │   ├── 03-vpc-errors.md
│   │   └── 04-serverless-errors.md
│   └── sap/               # SAP 错题本 + 模拟记录
└── cheatsheets/           # 考前速查表
    ├── saa-services.md    # 常见服务对比速查
    └── saa-scenarios.md   # 场景题解题思路
```

---

## 进度追踪

- [x] SAA-C03 学习中（Week 1-6 完成 = **P1 + P2 收工**；P3 无服务器进行中，超前 3 天）
- [x] SAA-C03 报名 ✅ 已预约 **2026-09-26（土）11:30 JST**，新桥国际会馆考场
- [ ] SAA-C03 模拟题通过率 > 80%
- [ ] SAA-C03 通过
- [ ] SAP-C02 学习中
- [ ] SAP-C02 模拟题通过率 > 80%
- [ ] SAP-C02 报名
- [ ] SAP-C02 通过
</content>
</invoke>