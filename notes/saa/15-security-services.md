# 防护五件套（WAF / Shield / GuardDuty / Inspector / Macie）+ Security Hub

> P4 安全收官 · 2026-09-13
> 名字都很像，但**各管一段**

---

## 0. ⭐ 先按「防什么」分组

| 服务 | 管什么 | 一句话 |
|---|---|---|
| **WAF** | 🔑 **应用层（L7）攻击** | 过滤 HTTP 请求：SQL 注入、XSS、按 IP/地理封禁 |
| **Shield** | 🔑 **DDoS（L3/L4/L7）** | 抗流量攻击 |
| **GuardDuty** | 🔑 **威胁检测**（发现坏人在活动） | 分析日志找异常行为 |
| **Inspector** | 🔑 **漏洞扫描**（发现自己有洞） | 扫 EC2 / 容器镜像 / Lambda 的 CVE |
| **Macie** | 🔑 **敏感数据发现** | 用 ML 找出 S3 里的 PII |

---

## 1. WAF —— 应用层防火墙

### 🚨 部署位置（只能挂这五个，是考点）

**CloudFront、ALB、API Gateway、AppSync、Cognito User Pool**

> 🚨 **WAF 不能挂在 NLB 上**（NLB 是 L4，WAF 是 L7）
> 题干是 NLB → 要么改用 ALB，要么在前面加 CloudFront

### 能做的规则

- **托管规则组** —— AWS 和第三方提供的现成规则（OWASP Top 10、已知恶意 IP）
- **SQL 注入 / XSS 过滤**
- **IP 集合**（黑白名单）、**地理封禁**（Geo Match）
- 🔑 **速率限制（Rate-based Rule）** —— 「同一 IP 5 分钟内超过 N 次请求就封」

> 🔑 判据：`SQL 注入`、`XSS`、`按国家封禁`、`按 IP 限流` → **WAF**

---

## 2. Shield —— 抗 DDoS

| | **Shield Standard** | **Shield Advanced** |
|---|---|---|
| 费用 | 🔑 **免费，自动开启** | 🚨 **$3000/月** |
| 防护 | L3/L4 常见攻击（SYN Flood、UDP 反射） | + **L7 攻击**、更大规模 |
| 独有 | — | 🔑 **DDoS 响应团队（DRT）24×7 支持**<br>🔑 **DDoS 导致的扩容费用补偿**<br>+ 免费使用 WAF |

> 🔑 判据
> `DDoS` + `需要专家支持` / `费用补偿` → **Shield Advanced**
> 只说「防常见 DDoS」→ **Shield Standard**（已免费自带，不用做任何事）

---

## 3. GuardDuty —— 威胁检测（找坏人）

**智能威胁检测服务**，用 ML + 威胁情报分析三类日志源
（🔑 **不需要你手动开启这些日志**）：

```
① CloudTrail 事件日志  —— 异常 API 调用（root 登录、禁用 CloudTrail）
② VPC Flow Logs       —— 与已知恶意 IP 通信、端口扫描
③ DNS 日志            —— 查询挖矿域名、C2 域名
```

**可扩展检测**：EKS 审计日志、S3 数据事件、RDS 登录、Lambda 网络活动、EBS 恶意软件扫描

> 🔑 判据：`检测异常/可疑活动`、`加密货币挖矿`、`被入侵的实例`、`凭证泄露` → **GuardDuty**
> 🔑 **一键开启、无需部署代理** —— 这是它的卖点

---

## 4. Inspector —— 漏洞扫描（找自己的洞）

**自动扫描三类目标的已知漏洞（CVE）和配置问题**：

| 目标 | 扫什么 |
|---|---|
| **EC2 实例** | OS 软件包 CVE、网络可达性（需 **SSM Agent**） |
| **ECR 容器镜像** | 镜像层里的软件包 CVE |
| **Lambda 函数** | 代码依赖的 CVE |

- **持续自动扫描**（不是定时任务），有新 CVE 公布就重新评估
- 结果发到 **Security Hub** 和 EventBridge

> 🔑 **GuardDuty vs Inspector 一句话区分**
> **GuardDuty = 有人在攻击我吗？**（行为异常，事中检测）
> **Inspector = 我自己有没有洞？**（已知漏洞，事前预防）

### 具体场景区分

```
你的 EC2 上装了个有漏洞的 Apache 版本
  → Inspector 报："这台机器的 Apache 有 CVE-2024-XXXX"    （洞还没被利用）

黑客利用这个洞进来了，开始向境外 IP 传数据、跑挖矿程序
  → GuardDuty 报："这台实例在与已知恶意 IP 通信 / 疑似挖矿"（正在被利用）
```

**时间线：Inspector 在事前（预防），GuardDuty 在事中（检测）。**

---

## 5. Macie —— 敏感数据发现

用**机器学习**扫描 **S3**，找出里面的**敏感数据（PII）**：
信用卡号、身份证号、护照号、姓名地址、密钥凭证。

> 🔑 判据：`PII`、`个人身份信息`、`敏感数据`、`GDPR/HIPAA 合规发现` + 明确说 **S3** → **Macie**
> ⚠️ Macie **只扫 S3**，不扫别的。

---

## 6. Security Hub —— 汇总台

把 GuardDuty、Inspector、Macie、Config、Firewall Manager 等的发现
**汇总到一个面板**，并按 CIS / PCI-DSS 等标准做合规评分。

> 🔑 判据：`集中查看所有安全告警`、`多账户统一安全视图` → **Security Hub**

---

## 7. ⭐ 五件套决策树（考试直接套）

```
HTTP 请求层面的过滤（SQL注入 / XSS / 限流 / 地理封禁）
    → WAF

流量型 DDoS，需要专家支持和费用补偿
    → Shield Advanced

发现"有人在我账户里搞事"（异常 API、挖矿、恶意 IP 通信）
    → GuardDuty

检查"我的 EC2 / 镜像 / Lambda 有没有已知漏洞"
    → Inspector

找出"S3 里有没有客户的身份证号 / 信用卡号"
    → Macie

把上面所有告警汇总到一个面板
    → Security Hub
```

---

## 8. 🚨 高频易错点

| 易错 | 正解 |
|---|---|
| 「保护 **NLB** 后面的应用不受 SQL 注入」→ 直接选 WAF | ❌ **WAF 挂不上 NLB**。要在 NLB 前加 CloudFront，或换成 ALB |
| Shield Standard 要不要配置 | ❌ 不用，**免费且自动开启** |
| GuardDuty 要不要先开 VPC Flow Logs / DNS 日志 | ❌ 不用，GuardDuty **自己就能读**，一键开启 |
| Macie 能不能扫 EBS / RDS | ❌ **只扫 S3** |
| Inspector 扫 EC2 要不要装东西 | ✅ 需要 **SSM Agent** |

---

## 9. 测验（10 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 电商网站遭受 **SQL 注入和 XSS 攻击**，需要在请求到达 ALB 后端前过滤掉。应使用？
A) Shield Advanced　B) WAF　C) GuardDuty　D) NACL

**Q2.** 安全团队怀疑某台 EC2 已被入侵，正在**与已知恶意 IP 通信并运行挖矿程序**。哪个服务能检测到？
A) Inspector　B) Macie　C) GuardDuty　D) Config

**Q3.** 合规要求定期确认 **EC2 实例和 ECR 镜像中是否存在已知 CVE 漏洞**。
A) GuardDuty　B) Inspector　C) Trusted Advisor　D) WAF

**Q4.** 公司在 S3 中存有大量客户资料，需要**自动识别哪些对象包含身份证号和信用卡号**。
A) Macie　B) GuardDuty　C) Config Rules　D) CloudTrail Data Events

**Q5.** 应用部署在 **NLB** 后面，要防护 SQL 注入。以下哪个方案可行？
A) 直接给 NLB 关联 WAF　B) 在 NLB 前加 CloudFront 并关联 WAF　C) 用 Shield Standard　D) 用 NACL 过滤

**Q6.** 某金融客户遭受大规模 DDoS，希望获得 **AWS 专家 24×7 支持**，并对**攻击期间产生的扩容费用获得补偿**。
A) Shield Standard　B) Shield Advanced　C) WAF 速率限制　D) CloudFront

**Q7.** 想**阻止来自特定国家的访问**，以及**限制单个 IP 每 5 分钟的请求次数**。
A) NACL　B) Security Group　C) WAF（Geo Match + Rate-based Rule）　D) Shield

**Q8.** 公司有 20 个账户，希望**在一个面板集中查看 GuardDuty、Inspector、Macie 的所有发现**并做合规评分。
A) CloudWatch Dashboard　B) Security Hub　C) Config Aggregator　D) Trusted Advisor

**Q9.** 关于 Shield Standard，下列**正确**的是？
A) 需要付费 $3000/月　B) 需要手动启用配置　C) 免费且自动为所有 AWS 客户开启　D) 只保护 CloudFront

**Q10.** 关于 GuardDuty，下列**错误**的是？
A) 分析 CloudTrail、VPC Flow Logs、DNS 日志　B) 一键开启，无需部署代理　C) 需要先手动开启 VPC Flow Logs 才能工作　D) 可检测加密货币挖矿行为
