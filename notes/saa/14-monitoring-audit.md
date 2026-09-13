# 监控与审计四件套（CloudTrail / CloudWatch / Config / Trusted Advisor）

> P4 安全 · 2026-09-13
> 这四个服务名字很像，但**回答的问题完全不同**

---

## 0. ⭐ 先用一张表钉死

| 服务 | 回答的问题 | 一句话 |
|---|---|---|
| **CloudTrail** | 🔑 **「谁做了什么？」** | **API 调用审计日志** |
| **CloudWatch** | 🔑 **「系统现在怎么样？」** | 指标、日志、告警 |
| **AWS Config** | 🔑 **「资源配置合规吗？变过吗？」** | 配置快照 + 合规规则 |
| **Trusted Advisor** | 🔑 **「有什么可以优化的？」** | 五大支柱最佳实践检查 |

---

## 1. CloudTrail —— 谁做了什么

记录账户内**所有 API 调用**（控制台点击、CLI、SDK 都算）。

| 特性 | 说明 |
|---|---|
| 默认状态 | 🚨 **默认开启**，但只在控制台保留 **90 天** |
| 长期留存 | 创建 Trail → 投递到 **S3**（可无限期保留） |
| **组织级** | 🔑 **Organization Trail** —— 一次配置覆盖所有成员账户 |
| 完整性校验 | **Log File Validation**（防篡改，合规必备） |
| 实时告警 | Trail → **CloudWatch Logs** → Metric Filter → Alarm |

### 两类事件

| 类型 | 默认 | 内容 |
|---|---|---|
| **Management Events** | ✅ 记录 | 创建/删除资源、改配置等**控制平面**操作 |
| **Data Events** | 🚨 **不记录**（量大） | **S3 对象级读写**、Lambda 函数调用 |

> 🚨 **考点**：「需要审计**谁删除了 S3 里的某个对象**」
> → 必须**手动开启 Data Events**，默认不记录。

---

## 2. CloudWatch —— 系统现在怎么样

### ① Metrics（指标）

- 🚨 **默认指标不包含内存和磁盘使用率！**
  EC2 的内存/磁盘需装 **CloudWatch Agent** 上报自定义指标 ← **超高频考点**
- 基础监控 **5 分钟**；**详细监控 1 分钟**（收费）

### ② Logs（日志）

- 日志组保留期：🚨 **默认永久**（不设置就一直计费）
- 来源：CloudWatch Agent、Lambda、ECS、API Gateway、VPC Flow Logs…
- **Logs Insights** —— 用查询语言分析日志
- **Metric Filter** —— 从日志提取模式生成指标 → 再挂 Alarm
  （例：从日志里数 ERROR 出现次数，超阈值告警）

### ③ Alarms（告警）

- 三种状态：`OK` / `ALARM` / `INSUFFICIENT_DATA`
- 动作：SNS 通知、ASG 扩缩容、EC2 停止/终止/恢复
- 🔑 **EC2 自动恢复**：`StatusCheckFailed_System` 告警 → **Recover** 动作
  （保留同一实例 ID / 私有 IP / EIP）

### ④ EventBridge

事件驱动自动化（详见 `11-sqs-sns-eventbridge-stepfunctions.md`）

### VPC Flow Logs（一起记）

记录 VPC / 子网 / ENI 的**网络流量元数据**（源/目的 IP、端口、ACCEPT 还是 REJECT）。

> 🔑 **排错信号**：「查明为什么 EC2 之间连不通 / 是谁在扫描端口」→ **VPC Flow Logs**
> ⚠️ **不记录数据包内容**，只记录元数据。

---

## 3. AWS Config —— 资源配置合规吗

| 能力 | 说明 |
|---|---|
| **配置历史** | 🔑 记录每个资源的**配置变更时间线**<br>（「这个 SG 什么时候被改成 0.0.0.0/0 的？」） |
| **Config Rules** | 托管规则 + 自定义规则，**持续评估**合规性 |
| **自动修复** | 配合 **SSM Automation** 自动纠正不合规资源 |
| **Conformance Packs** | 一组规则打包部署（PCI-DSS、HIPAA 等合规包） |
| 范围 | Region 级（可用 **Aggregator** 聚合多账户多 Region） |

**典型 Config Rules**：
S3 桶不能公开、EBS 必须加密、SG 不能对 0.0.0.0/0 开 22 端口、资源必须打特定标签

---

## 4. Trusted Advisor —— 有什么可以优化的

按**五大支柱**给建议：**成本优化、性能、安全、容错、服务限额**

| 支持计划 | 可用检查 |
|---|---|
| Basic / Developer | 🚨 只有 **7 项核心检查** |
| 🔑 **Business / Enterprise** | **全部检查** + API 访问 |

> 考点：「想用完整的 Trusted Advisor 检查」→ 需升级到 **Business 或 Enterprise Support**

---

## 5. ⭐ 四服务辨析（考试直接套）

```
「谁在 3 月 5 日删除了那个 RDS 实例？」
    → CloudTrail（API 调用审计）

「这台 EC2 的内存使用率多少？」
    → CloudWatch + 装 Agent（默认没有内存指标！）

「我们有多少个 S3 桶是公开的？什么时候变公开的？」
    → AWS Config（配置合规 + 历史）

「哪些预留实例利用率低，可以省钱？」
    → Trusted Advisor（成本优化建议）

「为什么这两台 EC2 网络不通？」
    → VPC Flow Logs
```

---

## 6. ⭐ CloudTrail vs Config —— 最易混的一对

都「记录变化」，区别在于记录的是**动作**还是**状态**：

```
CloudTrail：记录【动作】
    "用户 Alice 在 10:23 调用了 ModifySecurityGroup"
    关注【谁、什么时候、做了什么】

Config：    记录【状态】
    "这个 SG 在 10:23 前是 A 配置，之后是 B 配置，B 不合规"
    关注【资源现在长什么样、合不合规】
```

> **判据**：题干问「**谁**」→ CloudTrail；问「**配置对不对 / 变成什么样了**」→ Config
> 实务中两者常一起用：Config 发现不合规 → 去 CloudTrail 查是谁改的。

---

## 7. 🚨 默认值陷阱汇总（考前必扫）

| 服务 | 默认行为 | 陷阱 |
|---|---|---|
| CloudTrail | 默认开，但只留 **90 天** | 长期保留要建 Trail 投递 S3 |
| CloudTrail Data Events | **默认不记录** | 审计 S3 对象操作必须手动开 |
| CloudWatch EC2 指标 | **无内存、无磁盘** | 必须装 CloudWatch Agent |
| CloudWatch Logs | **默认永久保留** | 不设过期会一直计费 |
| Trusted Advisor | 只有 **7 项**检查 | 全量需 Business/Enterprise |
| VPC Flow Logs | **默认不开启** | 需手动启用 |

---

## 8. 测验（10 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 安全团队需要查明**是谁在上周删除了一个 RDS 实例**。应该查？
A) CloudWatch Logs　B) CloudTrail　C) AWS Config　D) VPC Flow Logs

**Q2.** 运维要监控 EC2 的**内存使用率**并在超过 80% 时告警。发现 CloudWatch 控制台里找不到内存指标。原因与做法？
A) 需要开启详细监控　B) 内存指标需安装 CloudWatch Agent 上报自定义指标　C) 需要升级支持计划　D) 内存指标在 CloudTrail 里

**Q3.** 合规审计要求：**证明过去一年中哪些 S3 桶曾被设为公开，以及变更时间**。
A) CloudTrail Data Events　B) S3 访问日志　C) AWS Config 配置历史 + Config Rules　D) Trusted Advisor

**Q4.** 需要审计**谁读取/删除了 S3 桶里的具体对象**，但发现 CloudTrail 里查不到。原因？
A) CloudTrail 没开启　B) Data Events 默认不记录，需手动开启　C) S3 不支持审计　D) 需要 Organization Trail

**Q5.** 公司有 30 个 AWS 账户，希望**一次配置就审计所有账户的 API 调用**。
A) 每个账户单独建 Trail　B) Organization Trail　C) CloudWatch 跨账户　D) Config Aggregator

**Q6.** 想使用 Trusted Advisor 的**全部检查项**（含成本优化完整建议）。需要？
A) 开启 CloudTrail　B) 升级到 Business 或 Enterprise Support　C) 启用 AWS Config　D) 免费即可全部使用

**Q7.** 两台 EC2 之间网络不通，SG 和 NACL 看起来都正确。要确认流量到底被谁拒绝了，应启用？
A) CloudTrail　B) VPC Flow Logs　C) CloudWatch Agent　D) AWS Config

**Q8.** 要求：**EC2 因底层硬件故障时自动恢复，且保持原有私有 IP 和实例 ID**。
A) ASG 自动替换实例　B) CloudWatch Alarm + EC2 Recover 动作　C) 多可用区部署　D) Route 53 故障转移

**Q9.** 应用日志写入 CloudWatch Logs，需要在日志中**出现 ERROR 超过 10 次/分钟时告警**。
A) Logs Insights 定时查询　B) Metric Filter 提取指标 + CloudWatch Alarm　C) CloudTrail Alarm　D) Config Rule

**Q10.** 关于 CloudTrail 和 AWS Config，下列**错误**的是？
A) CloudTrail 关注「谁做了什么」　B) Config 关注「资源配置是否合规」　C) Config 可配合 SSM Automation 自动修复　D) CloudTrail 日志在控制台永久保留
