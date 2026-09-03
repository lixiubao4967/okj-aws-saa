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
| **考试日期** | **2026-09-26（周六）** |

### 报名要点

1. https://aws.amazon.com/certification/ → **AWS Certification Account**（AWS Builder ID 登录）→ Schedule an Exam → Pearson VUE
2. **先申请 ESL +30 分钟延时，再订考位** —— 英文卷且英语非母语可免费申请（130 → 160 分钟），必须在预约前于 accommodations 里提交
3. 改期政策：考前 24h 以上免费改期，**每个考位最多改 2 次**；24h 内不可改期不可退款

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
| **P1 补完数据库** | 9/3(木) – 9/6(日) | DynamoDB（分区键/GSI/LSI/DAX/TTL/容量模式）、ElastiCache Redis vs Memcached、Redshift/Athena/Glue 速览、回顾 RDS+Aurora 笔记 | Week 5 剩余 |
| **P2 网络（最高频）** | 9/7(月) – 9/11(金) | 9/7-9/9 VPC 核心：子网、路由表、IGW、NAT GW、SG vs NACL<br>9/10-9/11 VPC 互联：Peering、Transit Gateway、VPN、Direct Connect、VPC Endpoints | Week 6 |
| **P3 无服务器 + 集成** | 9/12(土) – 9/14(月) | Lambda（并发/Layers/Edge）、SQS 标准 vs FIFO + DLQ、SNS 扇出、EventBridge、Step Functions、API Gateway、Kinesis 三兄弟 | Week 7 |
| **P4 安全 + 监控** | 9/15(火) – 9/17(木) | KMS/CMK/跨账户加密、Secrets Manager vs Parameter Store、CloudTrail、CloudWatch、Config、WAF/Shield/GuardDuty/Inspector/Macie | Week 8 |
| **P5 容器 + IaC + 成本** | 9/18(金) | ECS vs EKS vs Fargate、ECR、CloudFormation、Beanstalk、Cost Explorer/Budgets/Savings Plans | Week 9 |
| **P6 模拟题冲刺** ⭐ | 9/19(土) – 9/23(水) | 9/19 模拟卷① + 错题精读<br>9/20 错题知识点回炉 + 写 cheatsheets<br>9/21 模拟卷② + 复盘<br>9/22 薄弱域专项（按①②的域得分定）<br>9/23 模拟卷③ + 复盘 | Week 10 |
| **P7 收尾** | 9/24(木) – 9/25(金) | 9/24 通读 cheatsheets + 官方样题<br>9/25 只看错题本和速查表，早睡 | — |
| **考试** | **9/26(土)** | SAA-C03 | — |

**决策关卡：9/20 —— 若模拟卷① 正确率 < 70%，免费改期到 10/3（周六）。**

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
- [ ] DynamoDB：分区键、排序键、索引（GSI/LSI）、DAX、TTL
- [ ] ElastiCache：Redis vs Memcached
- [ ] Redshift、Athena、Glue 简介

#### Week 6：网络服务
- [ ] VPC：子网（公/私）、路由表、Internet Gateway、NAT Gateway
- [ ] Security Group vs NACL
- [ ] VPC Peering、Transit Gateway、VPN、Direct Connect
- [ ] VPC Endpoints（Gateway/Interface）

#### Week 7：应用集成 + 无服务器
- [ ] Lambda：触发器、并发、层（Layers）、边缘函数
- [ ] SQS：标准 vs FIFO、延迟队列、死信队列（DLQ）
- [ ] SNS：扇出模式（Fan-out）
- [ ] EventBridge、Step Functions、API Gateway
- [ ] Kinesis：Data Streams / Firehose / Analytics

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
│   │   └── 04-database-rds-aurora.md
│   └── sap/               # SAP-C02 进阶笔记
│       ├── 01-organizations.md
│       ├── 02-hybrid-network.md
│       └── 03-migration.md
├── practice/
│   ├── saa/               # SAA 错题本 + 模拟记录
│   │   ├── 01-week1-2-errors.md
│   │   └── 02-week5-errors.md
│   └── sap/               # SAP 错题本 + 模拟记录
└── cheatsheets/           # 考前速查表
    ├── saa-services.md    # 常见服务对比速查
    └── saa-scenarios.md   # 场景题解题思路
```

---

## 进度追踪

- [x] SAA-C03 学习中（Week 1-4 完成，Week 5 进行中 → 转入 9 月冲刺日历）
- [ ] SAA-C03 报名（目标考期 **2026-09-26**，待在 Pearson VUE 下单）
- [ ] SAA-C03 模拟题通过率 > 80%
- [ ] SAA-C03 通过
- [ ] SAP-C02 学习中
- [ ] SAP-C02 模拟题通过率 > 80%
- [ ] SAP-C02 报名
- [ ] SAP-C02 通过
</content>
</invoke>