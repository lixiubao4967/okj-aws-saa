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
| 考试代码 | SAA-C03 |
| 题目数量 | 65 道（单/多选） |
| 考试时长 | 130 分钟 |
| 通过分数 | 720 / 1000 |
| 考试费用 | $150 USD |
| 有效期 | 3 年 |

### SAA-C03 考试域分布

| 领域 | 占比 |
|------|------|
| Domain 1: Design Secure Architectures | 30% |
| Domain 2: Design Resilient Architectures | 26% |
| Domain 3: Design High-Performing Architectures | 24% |
| Domain 4: Design Cost-Optimized Architectures | 20% |

### 10 周学习计划

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

#### Week 5：数据库服务
- [ ] RDS：Multi-AZ vs Read Replica、备份与恢复
- [ ] Aurora：全局数据库、Serverless、Aurora vs RDS
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
│   │   └── 03-s3-storage-security.md
│   └── sap/               # SAP-C02 进阶笔记
│       ├── 01-organizations.md
│       ├── 02-hybrid-network.md
│       └── 03-migration.md
├── practice/
│   ├── saa/               # SAA 错题本 + 模拟记录
│   │   └── 01-week1-2-errors.md
│   └── sap/               # SAP 错题本 + 模拟记录
└── cheatsheets/           # 考前速查表
    ├── saa-services.md    # 常见服务对比速查
    └── saa-scenarios.md   # 场景题解题思路
```

---

## 进度追踪

- [ ] SAA-C03 学习中
- [ ] SAA-C03 模拟题通过率 > 80%
- [ ] SAA-C03 报名
- [ ] SAA-C03 通过
- [ ] SAP-C02 学习中
- [ ] SAP-C02 模拟题通过率 > 80%
- [ ] SAP-C02 报名
- [ ] SAP-C02 通过
</content>
</invoke>