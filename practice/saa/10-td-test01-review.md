# TD Practice Test 1 复盘（Tutorials Dojo, Exam mode）

> 成绩：**25/65 = 38%**（2026-09-20，限时 130 分钟）
> 复盘完成：2026-09-26
> 域得分：高性能 35% ｜ 安全 45% ｜ 弹性 37% ｜ 成本 33%（n=3）｜ 安全应用 0%（n=2）
> **四域全部 33–45%，没有明显强项。**

---

# 🔴 一、重复考点（同一套题考了两次）

| 知识点 | 题号 | 结果 | 一句话判据 |
|---|---|---|---|
| 🔴 **API Gateway Throttling** | Q13、Q50 | **两次都错** | `protect backend from traffic spikes` → **Throttling**（超限返 **429**） |
| 🔴 **S3 Versioning + MFA Delete** | Q21、Q65 | **两次都错** | 防**意外**删除 → **Versioning + MFA Delete**，不是"禁止删除权限" |
| 🔴 **ASG 缩容行为** | Q10、Q61 | **两次都错** | 缩容 5xx → **Deregistration Delay**；先终止谁 → **AZ均衡 → 最老启动模板** |
| 🟡 S3 Object Lock | Q48、Q54 | 一错一对 | **给了时间期限 → Retention Period；没给 → Legal Hold** |
| 🟢 FSx for Windows | Q39、Q51 | 两次都对 | Windows 文件共享 + AD → FSx for Windows |

> **上面三条红色的必须考前重点扫。**

---

# 🔴 二、七个错误模式（按出现频率）

## ① 虚构能力型干扰项（10+ 次，最大失分源）

**结构：真实的服务 + 它没有的功能。**

| 题 | 虚构了什么 |
|---|---|
| 官方 Q7 | IAM policy 自动更新密码 |
| 官方 Q18 | Shield 作为路由表 target |
| TD Q5 | CloudWatch Alarm 直接改 ECS task count |
| TD Q11 | **EFS lifecycle 删除文件**（只能转层） |
| TD Q11 | AWS Transfer 的 retention policy |
| TD Q18 | RDS 控制台现成的进程级 CPU%/MEM% |
| TD Q46 | **DynamoDB Read Replica**、**CloudFront Multi-AZ** |
| TD Q50 | **API Gateway Multi-AZ + read replica** |
| TD Q54 | Network Firewall 限制 S3 访问 |
| TD Q58 | RDS Event Subscription 捕获数据变更 |

### ✅ 防御动作（两问）
1. **这个功能真的存在吗？**
2. **这个操作物理上做得到吗？**（托管服务你进不去那台机器）

> 🔑 **虚构能力常出现在选项的【后半句】**——前半句服务对，后半句编功能。**重点审查最后一个动作。**

## ② 约束句 / 授权句（6+ 次，最高性价比得分点）

**题干用一句话解除或施加限制，直接决定答案。**

| 题 | 句子 | 作用 |
|---|---|---|
| 官方 Q9 | `does not need to access any databases` | 解锁 Pilot Light |
| 官方 Q14 | `data discarded after rendering` | 解锁 instance store |
| 官方 Q17 | `does not require consistent reads` | 解锁 DAX |
| TD Q8 | `without having to change the current URLs` | 排除 Signed URL |
| TD Q42 | `rather than through an Amazon Cognito identity pool` | 排除 Web Identity |
| TD Q45 | `unencrypted data / master keys should **never** be sent to AWS` | **两个 never，各砍一批** |
| TD Q54 | `Backup Vault Lock ... **but** requires object-level protection` | 排除 Vault Lock |
| TD Q59 | `Direct Connect would be too slow` | 排除 DX |

### ✅ 防御动作
读到 **`without` / `does not require` / `rather than` / `but` / `never` / `discarded`** →
**立刻问：这句放宽了什么？排除了什么？**
⭐ 每出现一个，在草稿纸上记一笔，答完确认每笔都用上了。

## ③ 抓住一个要求，漏了另一个（5+ 次）

| 题 | 抓住了 | 漏了 |
|---|---|---|
| TD Q1 | `remains available` → Multi-AZ | ⚠️ `properly migrated` → DMS |
| TD Q16 | 性能问题 | ⚠️ `9 to 5` 时间可预测 → Scheduled Scaling |
| TD Q31 | 跨 AZ | ⚠️ **block storage** → FSx for NetApp ONTAP |
| TD Q40 | 高可用 | ⚠️ **schema 频繁变化** → DynamoDB |
| TD Q44 | RPO/RTO 数字 | ⚠️ **`relational`** → Aurora 而非 DynamoDB |
| TD Q57 | 数据库读压力 | ⚠️ **`using an API`** → API Gateway + Lambda |

### ✅ 防御动作
- **读完题干先说一句："这题要的是 ___ 类型的方案"**（先定类型，再看指标）
- **题干的类型限定词常在第一句，优化目标常在最后一句** —— 两头都要读

## ④ 笔记里有但调不出来（8+ 次）

DAX、Pilot Light、Instance Store、Storage Gateway、API Gateway 缓存/限流、
IAM DB Auth（**9/20 刚补，9/23 又错**）、S3 Access Point、Object Lock

**根因：笔记按"服务说明书"组织，考试按"问题场景"提问，索引方向相反。**

### ✅ 对策
- 笔记每个功能后面加一句 **「题干出现 ___ 时选它」**
- **每天开头 10 分钟扫前一天补的内容**（补笔记 ≠ 记住）

## ⑤ 同族选项靠一句话区分（4 次）

Signed URL vs Signed Cookies（Q8）｜Retention vs Legal Hold（Q48/Q54）｜
Regular vs Rate-based rule（Q25）｜CSE-KMS vs CSE 客户端主密钥（Q45）

### ✅ 防御动作
**两个选项属于同一族时，题干一定埋了一句话来区分它们。找到那句话，别凭印象选。**

## ⑥ 选"中间选项"（3 次）

| 题 | 选了 | 正确 |
|---|---|---|
| 官方 Q1 | EKS 托管节点组 | ECS **Fargate** |
| 官方 Q8 | Instance Scheduler | 重构 **Lambda** |
| TD Q57 | ECS + Service Auto Scaling | ⭐ **Lambda** |

### ✅ 防御动作
题干给**极端约束词**（`within seconds`、`MOST cost-effective`、`LEAST overhead`）时，
**答案通常就是最极端的那个选项**，不要往中间找。

## ⑦ 创建后还要启用/关联（3 次）

| 题 | 漏掉的那一步 |
|---|---|
| TD Q25 | 创建 WAF Web ACL 后要 **associate 到 ALB** |
| TD Q32 | 用 Streams 前必须 **enable DynamoDB Streams** |
| TD Q23 | 建 IAM Role 后要在 **RDS 侧启用 IAM DB Auth** |

### ✅ 防御动作
**「这套东西默认是开着的吗？」「创建完还要做什么才生效？」**

### AWS 默认关闭的高频项
DynamoDB Streams ｜ S3 版本控制 ｜ S3 访问日志 ｜ CloudTrail 数据事件 ｜
RDS Enhanced Monitoring / Performance Insights ｜ VPC Flow Logs ｜
EBS 账户级默认加密 ｜ Multi-AZ ｜ CloudWatch 详细监控 ｜
⭐ **ASG 健康检查类型 = EC2（不是 ELB）**

---

# 三、本轮补进笔记的真知识缺口

| 知识点 | 补到哪 |
|---|---|
| Aurora 四种端点（Cluster/Reader/**Custom**/Instance） | `notes/saa/04` 3.5 |
| RDS vs Aurora 完整对比 + RPO/RTO | `cheatsheets/saa-scenarios` |
| RDS 监控三层（CloudWatch / **Enhanced Monitoring** / **Performance Insights**） | `notes/saa/14` 2.5 |
| S3 **客户端加密 CSE**（原笔记只有服务端三种） | `notes/saa/03` 6.5 |
| S3 数据保护（Versioning / MFA Delete / **Object Lock**） | `notes/saa/03` 7.2 |
| **CORS** + `Cross-` 家族辨析 | `notes/saa/03` 7.5 + `saa-naming-traps` |
| **S3 Transfer Acceleration + Multipart Upload** | `notes/saa/03` 10.5 |
| **S3 Access Point**（VPC 类型）触发词 | `notes/saa/03` 8 |
| **AWS Transfer Family** | `notes/saa/03` 9.5 |
| 各服务「自动删除」机制对照 | `notes/saa/03` |
| **Gateway / Endpoint / Policy** 同名词辨析 | `cheatsheets/saa-naming-traps.md` 🆕 |
| **多选题两种类型**判别法 | `cheatsheets/saa-scenarios` 零节 |
| 专用数据库地图（DocumentDB/Keyspaces/Neptune/Timestream/QLDB） | `cheatsheets/saa-scenarios` |

---

# 四、考前必扫清单（按优先级）

## 🔴 P0：重复错过的三条
```
API Gateway + traffic spikes        → Throttling（429）
防 S3 意外删除                       → Versioning + MFA Delete
ASG 缩容：5xx → Deregistration Delay ／ 终止谁 → AZ均衡+最老启动模板
```

## 🔴 P0：三个动作（每题必做）
```
① 约束句扫描  —— without / does not require / rather than / never
② 虚构能力检查 —— 这个功能真的存在吗？这个操作做得到吗？
③ 要求计数    —— 题干几个独立要求？多选题选几个？
```

## 🟡 P1：三份速查表
```
saa-scenarios.md   —— 打法 + 关键词→服务（按"问题→答案"方向，最对症）
saa-naming-traps.md —— Gateway/Endpoint/Policy/Cross- 同名词辨析
saa-services.md     —— 数字、默认值陷阱、核心对比
```

## 🟢 P2：速度
```
目标 2 分 27 秒/题（160 分钟 ÷ 65 题）
当前 3 分 01 秒/题 → 需砍约 20%
超过 2 分半还在纠结 → 标记跳过，最后回来
```
