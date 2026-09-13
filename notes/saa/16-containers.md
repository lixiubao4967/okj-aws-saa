# 容器三兄弟（ECS / EKS / Fargate）+ ECR + App Runner

> P5 容器 · 2026-09-13（原定 9/18，提前 5 天）

---

## 1. 🚨 先分清「谁管什么」

容器服务最容易乱，因为 **ECS/EKS 是「编排」，Fargate 是「算力模式」**
—— 它们**不是并列关系**：

```
编排层（谁来调度容器）：  ECS  或  EKS
                              ↓
算力层（容器跑在哪）：    EC2 模式  或  Fargate 模式
```

组合是 **2×2 四种**：
ECS on EC2 / ECS on Fargate / EKS on EC2 / EKS on Fargate

---

## 2. ECS vs EKS（编排层）

| | **ECS** | **EKS** |
|---|---|---|
| 本质 | 🔑 **AWS 自研**编排 | 🔑 **托管 Kubernetes** |
| 学习成本 | 低，与 AWS 深度集成 | 高，但是行业标准 |
| 费用 | 编排本身**免费** | 🚨 **$0.10/小时/集群**（约 $73/月） |
| 适合 | 纯 AWS 环境、想简单 | 🔑 **已有 K8s 经验 / 多云可移植 / 混合云** |

> 🔑 判据：`Kubernetes`、`kubectl`、`Helm`、`多云`、`混合云`、`现有 K8s 工作负载迁移` → **EKS**
> 只说「运行容器、最简单、最省事」→ **ECS**

### 🚨 ECS vs EKS 的选择与「规模」无关

很多人以为「大规模就用 K8s」，但 **ECS 一样能跑大规模**。
**判据只有一个：题干提没提 Kubernetes 生态。**
提了 → EKS；没提 → ECS（而且 EKS 每月多花 $73，成本敏感题更倾向 ECS）

---

## 3. ⭐ EC2 模式 vs Fargate 模式（算力层，更高频）

| | **EC2 启动类型** | **Fargate** |
|---|---|---|
| 谁管服务器 | 🚨 **你自己**（打补丁、扩容、优化利用率） | 🔑 **AWS 全管，无服务器** |
| 计费 | 按 EC2 实例 | 按**任务的 vCPU + 内存**用量 |
| 控制力 | 高（可选实例类型、用 Spot、GPU） | 低 |
| 适合 | 需要 GPU / 特殊实例 / 大规模稳定负载省钱 | 🔑 **不想管基础设施 / 负载波动大 / 运维开销最小** |

> 🔑 **判据极其干净**
> `无服务器`、`不想管理服务器/EC2`、`最小运维开销` → **Fargate**
> `GPU`、`需要特定实例类型`、`用 Spot 省钱` → **EC2 启动类型**

### 🚨 SAA 里绝大多数容器题答案是 Fargate

因为 SAA 考「架构选型」，题干十有八九会写「最小运维开销」「不想管理基础设施」。

**真正需要选 EC2 模式的只有三种情况**：
1. 需要 **GPU**
2. 需要**特定实例类型**（如超大内存）
3. 要用 **Spot 实例**大幅降本

**看不到这三个信号，就选 Fargate。**

---

## 4. ECS 的两个必考细节

### ① 任务的 IAM 权限有两层（容易混）

| 角色 | 给谁用 |
|---|---|
| **Task Role** | 🔑 **容器内的应用**访问 AWS 服务（如读 S3、写 DynamoDB） |
| **Task Execution Role** | 🔑 **ECS 代理**拉 ECR 镜像、往 CloudWatch 写日志 |

> 考点：「容器内应用需要读 S3」→ **Task Role**（不是 Execution Role）

### ② 与 ALB 集成 —— 动态端口映射

同一台 EC2 上跑多个同一服务的任务时会**端口冲突**。
🔑 **ALB 支持动态端口映射**（容器端口随机分配，ALB 自动注册目标），
**NLB / CLB 不行**。

---

## 5. ECR —— 容器镜像仓库

- 私有 / 公有镜像仓库，与 IAM 集成
- 🔑 **镜像扫描**：基础扫描 / **增强扫描（由 Inspector 驱动）**
- **生命周期策略**：自动清理旧镜像省存储费
- 支持跨 Region / 跨账户复制

---

## 6. App Runner —— 最简单的容器托管

从**源码或镜像**直接部署 Web 应用，自动完成构建、部署、负载均衡、自动扩缩、HTTPS。
**完全不用碰 ECS / VPC / ALB。**

> 🔑 判据：`最快上线一个 Web 应用/API`、`完全不想碰基础设施` → **App Runner**

---

## 7. ⭐ 容器选型决策树

```
需要 Kubernetes / 多云可移植 / 已有 K8s 技能
    → EKS

纯 AWS 环境，只想简单地跑容器
    → ECS

    ├─ 不想管服务器、运维最小、负载波动
    │     → Fargate 模式
    │
    └─ 需要 GPU / 特殊实例 / 用 Spot 大幅省钱
          → EC2 模式

只想把一个 Web 应用/API 跑起来，什么都不想管
    → App Runner

批处理作业（跑完就退出，可能几小时）
    → AWS Batch

长时间运行但超过 Lambda 15 分钟
    → ECS / Fargate
```

---

## 8. 测验（10 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 公司要把现有的 **Kubernetes 工作负载**迁到 AWS，团队已熟悉 kubectl，未来还要保持**多云可移植性**。
A) ECS on Fargate　B) EKS　C) App Runner　D) Elastic Beanstalk

**Q2.** 团队要跑一批容器化微服务，明确要求**不管理任何服务器、运维开销最小**。
A) ECS on EC2　B) ECS on Fargate　C) EC2 + Docker　D) EKS on EC2

**Q3.** 机器学习推理服务需要 **GPU 实例**跑容器。应选？
A) Fargate　B) App Runner　C) ECS/EKS on EC2 启动类型　D) Lambda

**Q4.** ECS 任务里的应用需要**读取 S3 存储桶**。应该配置哪个角色？
A) Task Execution Role　B) Task Role　C) EC2 实例角色　D) Service-Linked Role

**Q5.** 同一台 EC2 上要运行同一服务的多个 ECS 任务，出现**端口冲突**。解决方案？
A) 用 NLB　B) 用 ALB 的动态端口映射　C) 每个任务一台 EC2　D) 改用 Classic LB

**Q6.** 初创团队只想**尽快把一个 Web API 上线**，不想碰 VPC、ALB、集群配置。
A) EKS　B) ECS on EC2　C) App Runner　D) CloudFormation 部署 ECS

**Q7.** 关于 EKS 的成本，正确的是？
A) 编排完全免费　B) 每集群约 $0.10/小时的控制平面费用　C) 只按容器用量计费　D) 比 ECS 更便宜

**Q8.** 需要**自动清理 ECR 中的旧镜像**以节省存储费用。使用？
A) S3 生命周期策略　B) ECR 生命周期策略　C) Lambda 定时删除　D) Config Rule

**Q9.** 数据处理作业跑完即退出，单次约 **3 小时**，需要托管的批处理调度。最合适？
A) Lambda　B) AWS Batch　C) App Runner　D) API Gateway

**Q10.** 关于 ECS 和 EKS 的选择，下列**错误**的是？
A) 提到 kubectl / Helm 倾向 EKS　B) ECS 编排本身不额外收费　C) 大规模负载必须用 EKS，ECS 撑不住　D) EKS 适合多云可移植场景
