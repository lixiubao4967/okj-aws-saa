# Week 1-2：AWS 基础 + 计算服务

## 1. AWS 全球基础设施

### Region（区域）
- 全球划分的地理区域，如 `us-east-1`、`ap-northeast-1`
- 每个 Region 完全独立，数据默认不跨 Region
- 选择优先级：**合规 > 延迟 > 服务可用性 > 价格**

### Availability Zone（可用区）
- 每个 Region 包含 2-6 个 AZ（通常 3 个）
- 每个 AZ 是独立数据中心，有独立供电、网络、冷却
- **高可用的关键：跨 AZ 部署**

### Edge Location（边缘节点）
- 400+ 个，用于 CloudFront（CDN）和 Route 53（DNS）缓存
- 只做缓存，不能跑 EC2（Lambda@Edge 除外）

---

## 2. IAM（Identity and Access Management）

- **全局服务**，不分 Region

### 核心概念
| 概念 | 说明 |
|------|------|
| User | 代表一个人或程序 |
| Group | 用户集合，不能嵌套 |
| Role | 可被"扮演"的身份，通过 STS 临时凭证工作 |
| Policy | JSON 权限文档，定义 Allow/Deny |

### 关键规则
- Root 账户只用于初始设置
- 权限默认 Deny，显式 Deny > 显式 Allow > 默认 Deny
- EC2 访问其他服务 → **绑 IAM Role**，不要用 Access Key

### Policy 结构
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
```

### MFA & STS
- MFA：建议为 Root 和所有 User 开启
- STS：`AssumeRole` 提供临时凭证（15 分钟 ~ 12 小时）

---

## 3. EC2（Elastic Compute Cloud）

### 实例命名规则
`m5.xlarge` → m（族）5（代）xlarge（大小）

### 常考实例族
| 族 | 特点 | 场景 |
|----|------|------|
| T | 可突增，有 CPU 积分 | 开发测试、小型 Web |
| M | 通用平衡 | 应用服务器 |
| C | 计算优化 | 批处理、科学计算 |
| R | 内存优化 | 内存数据库、大数据 |
| I | 存储优化，高 IOPS | NoSQL、数据仓库 |
| G/P | GPU 加速 | 机器学习、视频编码 |

### 购买选项
| 选项 | 适用场景 | 折扣 |
|------|----------|------|
| On-Demand | 短期、不可预测 | 无 |
| Reserved (RI) | 稳定运行 1-3 年 | 最高 72% |
| Savings Plans | 灵活长期使用 | 最高 72% |
| Spot | 可中断的批处理 | 最高 90% |
| Dedicated Host | 合规、自带许可证（BYOL） | — |

> 口诀：省钱稳定 → RI；省钱可中断 → Spot；合规/许可证 → Dedicated Host

---

## 4. EC2 存储

### EBS（Elastic Block Store）
- 网络附加块存储，绑定单个 AZ
- 通常一个 EBS 挂一个 EC2（io1/io2 支持 Multi-Attach）

| 类型 | 特点 | 场景 |
|------|------|------|
| gp3/gp2 | 通用 SSD，性价比高 | 启动盘、普通应用 |
| io2/io1 | 最高 IOPS，Multi-Attach | 高性能数据库 |
| st1 | 吞吐优化 HDD | 大数据、日志 |
| sc1 | 最便宜 HDD | 归档（HDD 不能做启动盘） |

### Instance Store
- 物理磁盘，极高 IOPS，但 **EC2 停止后数据丢失**
- 适合缓存、临时文件

### EBS 快照
- 增量备份，可跨 AZ / 跨 Region 复制
- 跨 AZ 迁移：快照 → 目标 AZ 恢复

---

## 5. AMI / Launch Template / Auto Scaling Group

### AMI
- EC2 镜像模板，绑定 Region，跨 Region 需复制

### Launch Template
- EC2 启动参数模板，支持版本管理

### Auto Scaling Group (ASG)
- 自动增减实例，核心参数：Min / Desired / Max
- 搭配 ELB + 多 AZ = 高可用三件套

| 扩缩策略 | 工作方式 |
|----------|----------|
| Target Tracking | 维持指标在目标值（如 CPU 50%） |
| Step Scaling | 按告警阈值分段调整 |
| Scheduled | 按时间计划扩缩 |
| Predictive | 基于历史模式预测 |

---

## 学习进度

- [x] AWS 全球基础设施 — 已答题通过
- [x] IAM — 已答题通过
- [x] EC2 实例类型 + 购买选项 — 已答题通过
- [x] EC2 存储 + AMI + ASG — 已讲解
- [ ] **待完成：Week 1-2 综合题（4 道）**
- [ ] 进入 Week 3：ELB / Route 53 / CloudFront
