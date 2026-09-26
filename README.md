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

### ✅ 已报名（Status: Scheduled）🔄 **2026-09-22 免费改期**

| 项目 | 内容 |
|------|------|
| **日期时间** | **2026-10-04（日）09:45 JST** ⬅️ 原 09-26（土）11:30 |
| 考场 | Shinbashi Kokukaikan Test Center 2（東京都港区新橋 1-18-1 国際会館 2F, 105-0004） |
| 语言 | English |
| 时长 | **170 分钟**（答题 **160** = 130 + ESL 30，其余为流程时间） |
| 便利 | ESL Extra Time 30 Minutes（Approved，永久有效，SAP-C02 也自动生效） |
| 费用 | JPY 22,000（¥20,000 + 税），考过后公司报销；**改期 ¥0** |

> **改期原因**：9/19 官方练习题 55%（加权 52.8%）、9/20 TD Test 1 **38%**，
> 且出现「前天补的笔记今天仍答错」——诊断为**知识在进来但缺少沉淀时间**，需要间隔复习轮次。

> ⚠️ **开始时间提前了 1h45m**：需 **09:15 到场** → **08:00 前出门**。
> **10/1 起每天提早 30 分钟起床**，10/4 当天不要是第一次早起。

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
| **P3 无服务器 + 集成** ✅ | 9/9(水) – 9/12(土) | ✅ Lambda + API Gateway — **10/10** 🎯<br>✅ SQS/SNS/EventBridge/Step Functions — **12/12** 🎯<br>✅ Kinesis 三兄弟 — **6/8**（错题 12-13）<br>**P3 合计 28/30（93%）**，比原计划**提前 2 天完成** | Week 7 |
| **P4 安全 + 监控** ✅ | 9/13(日) 单日完成 | ✅ KMS + 密钥凭据管理 — **9/10**（错题 14）<br>✅ 监控审计四件套 — **9/10**（错题 15）<br>✅ 防护五件套 + Security Hub — **10/10** 🎯<br>**P4 合计 28/30（93%）**，比原计划**提前 4 天完成** | Week 8 |
| **P5 容器 + IaC + 成本** ✅ | 9/13(日) – 9/14(月) | ✅ 容器三兄弟 + ECR + App Runner — **10/10** 🎯<br>✅ CloudFormation / Beanstalk / 成本优化 — **12/12** 🎯<br>**P5 合计 22/22（100%）**，比原计划**提前 4 天完成** | Week 9 |
| **P6 模拟题冲刺** ⭐ | **9/15(火) – 9/23(水)**<br>*提前 4 天，可加到 4-5 套* | 9/19 模拟卷① + 错题精读<br>9/20 错题知识点回炉 + 写 cheatsheets<br>9/21 模拟卷② + 复盘<br>9/22 薄弱域专项（按①②的域得分定）<br>9/23 模拟卷③ + 复盘 | Week 10 |
| **P7 收尾** | 9/24(木) – 9/25(金) | 9/24 通读 cheatsheets + 官方样题<br>9/25 只看错题本和速查表，早睡 | — |
| **考试** | **9/26(土)** | SAA-C03 | — |

**决策关卡：9/20 —— 若模拟卷① 正确率 < 70%，免费改期到 10/3（周六）。**

> ### 📊 混合模拟卷①（Claude 出题，65 题）：**47/65 = 72.3%**
>
> | 批次 | 得分 | 机制 |
> |---|---|---|
> | 第 1 批 | 14/22（63.6%） | 无 |
> | 第 2 批 | 14/22（63.6%） | 无 |
> | **第 3 批** | **19/21（90.5%）** 🎯 | **回读验证** |
>
> ✅ **改期决策：不改期，按原计划 9/26**（第 3 批 90.5% 超过 75% 阈值）
> ⚠️ 但这是 Claude 出的题不是真题，**必须用 Tutorials Dojo 校准**
>
> 原始记录 —— 第 1 批 14/22（63.6%）｜第 2 批 14/22（63.6%）—— 两批一致 = 趋势
> 🔁 **错题在重复**：Q35 = 错题 1 重现、Q37 = 错题 6 重现 → 错题本只读不练不会转化
> ✅ **新机制（第 3 批起强制）**：选出答案后**回到题干，逐句核对「我选的这个满足这句吗」**
> 　（把「注意力」改成「验证」——机械动作，不依赖状态。第 2 批用它能救回 4 题 → 82%）
> 　原第 1 批记录： —— 分模块 93-100% → 混合长题干 64%，落差 29 个点。
> **知识没退步**（8 道错题中 5 道知识点在模块测验已答对），失分集中在**长题干审题**：
> **5/8 的错题，答案就明写在题干里**（「中断后可恢复」「使用模式非常明确」「与数据库查询耗时无关」…）。
> **当前判断：先不改期**，真正的判断点是第 3 批后能否回到 75%+。详见 `practice/saa/08-mock-exam-01-review.md`

> ### 🚨 AWS 官方练习题集（Skill Builder 免费 20 题）：**11/20 = 55%**｜加权 **52.8%**
>
> **2026-09-19 完成（exam mode）。这是出题方自己写的题，代表性高于任何第三方题库。**
>
> | Domain | 考试权重 | 得分 | 加权贡献 |
> |---|---|---|---|
> | D1 安全 | **30%** | 40%（2/5） | 12.0 |
> | D2 弹性 | **26%** | 40%（2/5） | 10.4 |
> | D3 高性能 | 24% | 60%（3/5） | 14.4 |
> | D4 成本 | 20% | **80%**（4/5） | 16.0 |
>
> ⚠️ **加权分低于原始分**：强的 D4 权重最小，弱的 D1+D2（合计占考试 **56%**）都只有 40%。
> ⏱️ **平均 3 分 01 秒/题**（真考需 2 分/题）→ 按此速度 65 题会有 20+ 题做不完。
> 🔴 **未达 9/17 自定的「真题 80%+ 就稳」阈值，差 25 个点。**
>
> **核心发现：模块满分 → 官方题不及格**
> 容器 10/10、应用集成 12/12、防护五件套 10/10、VPC 19/20 —— 对应的官方题全错。
> **9 道错题中 8 道知识点笔记里都有**，Q17（DAX）甚至被自己标注为「送分题」。
> → **不是知识缺口，是 4 个月的遗忘 + 提取失败。**
>
> **三条规律覆盖 7/9 错题**（详见 `practice/saa/09-official-practice-set-review.md`）：
> 1. **「授权句」模式**（Q9/Q14/Q17）：`does not require / discarded` → 在解除某服务的固有限制
> 2. **「数字比对」模式**（Q8/Q10）：题干时间数字要换算后与服务参数比对
> 3. 🔴 **Shield 边界错 2 次**（Q4/Q18）= 失分 22%：**Shield 只抗 DDoS**，其余安全需求一律排除

### 🔥 改期后冲刺计划（2026-09-23 → 10-04，11 天）

> **本次计划的核心补丁：加入「间隔复习」。**
> 前一轮的失败机制不是学得少，是**补进去的知识没有第二次接触**——
> 9/20 补的 IAM DB Auth、刚读完的 Gateway Endpoint 边界，都在一两天内再次答错。

| 阶段 | 日期 | 内容 | 产出 |
|---|---|---|---|
| **P1 清账** ✅ | **9/23（三）– 9/26（六）** | ✅ **TD Test 1 全部 65 题过完**（40 道错题 + 重点对题）<br>✅ 补进 13 处真知识缺口 + 新建 1 份速查表 | ✅ `practice/saa/10-td-test01-review.md` |
| **P2 弱域专项 + 第一轮系统复习** | **9/26（六）– 9/27（日）** | **D1 安全（30%）+ D2 弹性（26%）** 专项<br>**通读三份 cheatsheets**（services / scenarios / naming-traps） | 薄弱点清单更新 |
| **P3 校准 ①** | **9/28（一）– 9/30（三）** | 9/28 **TD Test 2 — 限时 160 分钟，exam mode**<br>9/29–9/30 复盘 + 补洞 | 域得分趋势 |
| **P4 校准 ② + 第二轮复习** | **10/1（四）– 10/2（五）** | 10/1 **TD Test 3 — 限时 160 分钟**<br>10/2 复盘 + **重扫 P1–P3 的全部补充**<br>⏰ **作息开始前移** | 最终判据 |
| **P5 收尾** | **10/3（六）** | ❌ **不做新题**<br>只读错题本 + cheatsheets，**早睡** | — |
| **考试** | **10/4（日）09:45** | 08:00 出门 · 09:15 到场 | |

**每日固定动作（不可省）：**
```
① 开头 10 分钟 —— 扫前一天补的笔记/速查表（不精读，只过表格）
② 做题时     —— 回读验证 + 授权句扫描 + 「这个功能真的存在吗」
③ 结束前     —— 一句话总结今天最大的一个认知修正
```

**判据**：Test 2（9/28）≥ **55%**、Test 3（10/1）≥ **65%** → 按 10/4 考。
达不到则 10/2 再评估（**10/3 前仍可免费改期，但每个考位最多改 2 次，已用 1 次**）。

---

### 📍 下次从这里开始（更新于 2026-09-22）

🎓 全部知识模块已结束（17 篇笔记）+ cheatsheets 已完成。P6 模拟题冲刺中。

**当前状态**：Claude 混合卷① 72.3% → **官方题 55%（加权 52.8%）**
⚖️ **改期决策：待定**（免费改期窗口到 **9/25**，考前 24h 以上改期不收费）

**剩余 6 天安排（优先级已按 权重 × 差距 重排）：**

| 日期 | 内容 |
|---|---|
| **9/20** | ✅ 官方题 9 道错题逐题精读完成 → `09-official-practice-set-review.md`<br>🔴 **回炉 `notes/01`–`04`（4 月学的，遗忘最重）** |
| **9/21** | **D1 安全 + D2 弹性专项**（占考试 56%，当前均 40%）<br>重点：WAF/Shield 边界、DR 四策略、SG 引用 SG、解耦模式 |
| **9/22** | 限时混合题（**严格 2 分/题**）+ 错题回炉 |
| **9/23** | 完整 65 题限时模拟 → **改期决策的最终依据** |
| **9/24–9/25** | 只读 cheatsheets + `09-official-practice-set-review.md`，早睡 |

> 🚨 **每题必做两个动作**：
> ① **回读验证** —— 选出答案后逐句核对题干要求（已验证 +27 个百分点）
> ② **授权句扫描** —— 读到 `does not require / does not need / discarded` 立刻问「这放宽了什么？」
>
> ⏱️ **同时必须治速度**：超过 2 分半还在纠结 = 大概率不会，**标记跳过**。
> 数据支持：答对的题普遍更快（会 = 识别；不会 = 回忆+猜）。

> 📖 **复习顺序按遗忘程度反向**（越早学的忘得越多）：
> 🔴 `01`–`04`（4 月）→ 🟡 `05`–`09`（9 月初）→ 🟢 `10`–`17`（9 月中）

> 📕 **速查表已就绪**：
> - `cheatsheets/saa-services.md` —— 数字速查、默认值陷阱、核心对比表、排错四步法
> - `cheatsheets/saa-scenarios.md` —— **第零节是个人失分点对策，考前必读**；关键词映射总表、
>   高频架构母题、考试当天操作清单
> - `cheatsheets/saa-naming-traps.md` 🆕 —— **同名词辨析**：Gateway 全家族（网络/端点/LB/存储）、
>   Endpoint 家族、Policy 家族。治"一看到 Gateway 就发懵"

> ⚠️ **重要提醒**：此前所有测验都是 Claude 出的**单模块题**，与真考差异明显：
> 真考是**跨域混合、题干更长、干扰项更精细**，且 65 题/130 分钟的**时间压力**无法用分模块练习模拟。
> **93% 的分模块正确率 ≠ 真考 93%。** 必须用真实题库校准。

> 💡 **富余时间用法**：P5 完成后距 9/19 还有约 5 天，可把模拟卷从 3 套加到 **4-5 套**，
> 或提前开始写 cheatsheets。决策关卡 9/20 不变。

**9/7 完成**：ElastiCache 5/5 ✅ ｜ 分析服务 0/5 → 错题精读 → 重测 4/5 ✅ **P1 收工**
**9/8 完成**：VPC 核心 **9/10** ✅ ｜ VPC 互联 **10/10** 🎯 **P2 收工（19/20 = 95%）**
**9/9 完成**：Lambda + API Gateway — 测验 **10/10** 🎯（连续两轮满分）
**9/10-9/11 完成**：SQS / SNS / EventBridge / Step Functions — 测验 **12/12** 🎯
**9/12 完成**：Kinesis **6/8**（错题 12 partition key 顺序优先；错题 13 Firehose 不存储≠数据丢）→ **P3 收工 28/30（93%）**

**9/13 完成**：KMS + 密钥凭据管理 — 测验 **9/10**（错题 14：加密 EBS 快照跨 Region 复制）
　　　　　　　监控审计四件套 — 测验 **9/10**（错题 15：EC2 Recover vs ASG 替换）
　　　　　　　防护五件套 + Security Hub — **10/10** 🎯 → **P4 收工 28/30（93%）**
　　　　　　　容器三兄弟 + ECR + App Runner — 测验 **10/10** 🎯
**9/14 完成**：CloudFormation / Beanstalk / 成本优化 — **12/12** 🎯 → **P5 收工 22/22（100%）**
　　　　　　　🎓 **全部知识模块结束**
　　　　　　　✍️ **cheatsheets 两份完成**（原计划 9/20，提前 6 天）
**9/19 完成**：🚨 **AWS 官方练习题 20 题 — 11/20（55%），加权 52.8%**，未达 80% 阈值
**9/20 完成**：9 道错题逐题精读 + 知识巩固 → `practice/saa/09-official-practice-set-review.md`
　　　　　　　📝 补齐笔记缺口：`01` Instance Store 完整属性（原缺「免费」这条决胜点）
　　　　　　　📝 新增 `04` 第 5 节：控制平面 vs 数据平面 + DocumentDB（原完全未覆盖）
　　　　　　　🚨 **TD Practice Test 1（Exam mode）：25/65 = 38%**，四个域 33-45% 全面偏低
**9/21 进行中**：TD Test 1 错题逐题精读（Q1/Q3/Q5/Q8/Q9/Q10…）
　　　　　　　📝 `01` 补 Instance Store「只能启动时指定」+ EBS 快照 ≠ S3 对象 对照表
　　　　　　　📝 `03` 补「IA ≠ Archive」关键词判据 + 生命周期瀑布模型
　　　　　　　✍️ **新增 `cheatsheets/saa-naming-traps.md`** —— Gateway/Endpoint/Policy 同名词辨析

> 🔥 连续三轮满分后在 Kinesis 轮回落 —— 暴露两个新毛病，见下方提醒

> ➡️ **进度超前原计划约 3 天**，多出的时间全部并入 P6 模拟题冲刺（9/19-9/23 五连休）。

> 📌 **出题纪律（两次踩坑后的规则）**：给答题格式示例时必须用不可能是答案的串（如 `1A 2B 3C`），
> 且**选项里的加粗只能强调题干关键词，绝不标记正确答案**。笔记里的测验一律不写答案。

> ⚠️ 个人薄弱点提醒（累计 5 条，详见 `practice/saa/04-serverless-errors.md` 末尾）
> 1. ✅ 过度设计 —— 已改正
> 2. ✅ 计算题跳步 —— 已改正（写三步不心算）
> 3. ✅ 服务能力边界记错 —— 已改正
> 4. 🚨🚨 **漏读后半句的「限定条件」—— 已栽 3 次（错题 1 / 12 / 15），最顽固失分点**
>    读到「且」「同时」「并保持」「不影响」「最低成本」「不能」时，**停一秒把后面那句单独圈出来**。
>    主需求往往多个方案都满足，**限定条件才是唯一的筛子**。考前最后一天必重读 `05-security-errors.md` 末节
> 5. ⚠️ **把服务属性套到整个架构** —— 架构题先画数据路径，再问「数据/状态现在在哪」
> 6. ⚠️ **警惕「XX 不支持 YY」类选项** —— AWS 极少用「功能不存在」当正确答案。
>    选项越**具体、像操作步骤** → 越可能是答案；越**笼统否定** → 越可能是干扰项（错题 14）
>
> 解题通法：先圈**频率词**（偶尔/持续/实时/每天）+ **约束词**（成本/运维/无代码/顺序）。
> 两选项都「技术可行」时，判据是约束词。

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

#### Week 7：应用集成 + 无服务器 ✅
- [x] Lambda：触发器、并发、层（Layers）、边缘函数（测验 10/10）
- [x] API Gateway：三种类型、端点类型、授权方式、29 秒超时（测验 10/10）
- [x] SQS：标准 vs FIFO、可见性超时、长轮询、死信队列（DLQ）（测验 12/12）
- [x] SNS：扇出模式（Fan-out）+ 消息过滤（测验 12/12）
- [x] EventBridge（SaaS/cron/Replay）、Step Functions（Standard vs Express）（测验 12/12）
- [x] Kinesis：Data Streams / Firehose / Managed Flink（测验 6/8）

#### Week 8：安全 + 监控 ✅
- [x] KMS：三种密钥、信封加密、CMK、密钥轮换、跨账户双重授权（测验 9/10）
- [x] Secrets Manager vs SSM Parameter Store（测验 9/10）
- [x] CloudTrail（90天/Data Events/Organization Trail）、CloudWatch（Metrics/Logs/Alarms）（测验 9/10）
- [x] AWS Config、Trusted Advisor、VPC Flow Logs（测验 9/10）
- [x] WAF、Shield、GuardDuty、Inspector、Macie + Security Hub（测验 10/10）

#### Week 9：容器 + 其他服务 ✅
- [x] ECS vs EKS vs Fargate 选择场景（测验 10/10）
- [x] ECR、App Runner（测验 10/10）
- [x] CloudFormation：模板结构、StackSets、Change Sets、Drift Detection（测验 12/12）
- [x] Elastic Beanstalk：五种部署策略（测验 12/12）
- [x] 成本优化：RI vs Savings Plans vs Spot、Cost Explorer、Budgets（测验 12/12）

#### Week 10：冲刺 + 刷题
- [ ] 复习错题，重点覆盖薄弱领域
- [ ] 完成至少 3 套完整模拟题（65 题/套）
- [x] 整理高频考点速查表（`cheatsheets/saa-services.md` + `saa-scenarios.md`）

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
│   │   ├── 12-kinesis.md      # Kinesis 三兄弟 + SQS/SNS/Kinesis 区分
│   │   ├── 13-kms-secrets.md  # KMS/信封加密 + Secrets Manager vs Parameter Store
│   │   ├── 14-monitoring-audit.md  # CloudTrail/CloudWatch/Config/Trusted Advisor
│   │   ├── 15-security-services.md # WAF/Shield/GuardDuty/Inspector/Macie/SecurityHub
│   │   ├── 16-containers.md   # ECS/EKS/Fargate + ECR + App Runner
│   │   └── 17-iac-cost.md     # CloudFormation/Beanstalk + 成本优化
│   └── sap/               # SAP-C02 进阶笔记
│       ├── 01-organizations.md
│       ├── 02-hybrid-network.md
│       └── 03-migration.md
├── practice/
│   ├── saa/               # SAA 错题本 + 模拟记录
│   │   ├── 01-week1-2-errors.md
│   │   ├── 02-week5-errors.md
│   │   ├── 03-vpc-errors.md
│   │   ├── 04-serverless-errors.md
│   │   ├── 05-security-errors.md
│   │   └── 06-containers-iac-cost-errors.md
│   └── sap/               # SAP 错题本 + 模拟记录
└── cheatsheets/           # 考前速查表 ✅
    ├── saa-services.md    # 数字速查/默认值陷阱/核心对比/排错四步法
    └── saa-scenarios.md   # 个人失分点对策/关键词映射/架构母题/当天清单
```

---

## 进度追踪

- [x] SAA-C03 **全部知识模块完成**（Week 1-9 / P1~P5 全收工，17 篇笔记，超前 4 天）

### 📊 分模块测验成绩汇总

| 模块 | 得分 | 备注 |
|---|---|---|
| P2 网络 | 19/20（95%） | 错题 11 |
| P3 无服务器 + 集成 | 28/30（93%） | 错题 12-13 |
| P4 安全 + 监控 | 28/30（93%） | 错题 14-15 |
| P5 容器 + IaC + 成本 | **22/22（100%）** 🎯 | 无错题 |

> ⚠️ 这些是**单模块**测验成绩，不能直接换算成真考预期分。见下方提醒。
- [x] SAA-C03 报名 ✅ 已预约 **2026-09-26（土）11:30 JST**，新桥国际会馆考场
- [ ] SAA-C03 模拟题通过率 > 80%
- [ ] SAA-C03 通过
- [ ] SAP-C02 学习中
- [ ] SAP-C02 模拟题通过率 > 80%
- [ ] SAP-C02 报名
- [ ] SAP-C02 通过
</content>
</invoke>