# TD Practice Test 3 复盘

> 成绩：**≈32/65 ≈ 49%**（2026-10-02 20:54，Exam mode）
> 域得分：高性能 48%（21 题）｜安全 47%（17 题）｜弹性 50%（18 题）｜成本 56%（9 题）
> 复盘完成：2026-10-03（考前一天，按判据快速过，不逐题深挖）

---

# 一、趋势：平台期

| Domain | Test 1 | Test 2 | **Test 3** |
|---|---|---|---|
| 高性能 | 35% | 40% | **48%** ↑ |
| 安全 | 45% | 67% | **47%** ↓ |
| 弹性 | 37% | 47% | **50%** ↑ |
| 成本 | 33% | 40% | **56%** ↑ |
| **总分** | **38%** | **50%** | **≈49%** → |

## 🔬 诊断：三层原因叠加

```
分模块测验（Claude 出，学完即测）  93–100%   ← 虚高（blocked practice）
混合模拟卷（Claude 出）            72%
官方 20 题                          55%
TD 三套                             38% → 50% → 49%
```

1. **长尾知识覆盖不全**——每套 TD 暴露 10–25 个新点；笔记里一行带过 ≈ 没学过
2. **补过的笔记没记住**——本轮一半错题是 Test 1/2 已补过的知识点
3. **英文阅读**——占比小（约 2/17）

## 决策（10/3）

- **按期 10/4 考，不再改期**
- **保持英文（160 分钟）**：英文导致的错只有约 2/17，改中文少 30 分钟不划算
- 🔁 **若这次没过，重考时改中文**

---

# 二、🚨 重复错题（补过笔记还是错）

| 题 | 知识点 | 上次补在哪 | 一句话判据 |
|---|---|---|---|
| Q2 | Aurora 端点 | Test 1 → `notes/saa/04` 3.5 | ⭐ **Cluster = 只连 Primary（写）**；**Reader = 读负载均衡**（名字是坑） |
| Q14 | CORS vs 跨账户 | Test 1 → `notes/saa/03` 7.5 | 看到 `account` → **IAM 跨账户权限**；`origin`/`domain` 才是 CORS |
| Q17 | Storage Gateway | Test 1「调不出来」 | **本地服务器还在 + 低延迟/本地缓存** → **File Gateway**（不是 FSx） |

> 🔑 **补笔记 ≠ 记住。只有主动提取（我问你答）才转化。**

---

# 三、🔴 错误模式（本轮）

## ① 回声词陷阱（新模式，2 次）

**选项照抄题干里的词 → 往往是干扰项。**

| 题 | 题干 | 被钓的选项 |
|---|---|---|
| Q3 | `in a single Availability Zone` | 配额 `per Availability Zone`（实际**按 Region**） |
| Q28 | cluster `spans multiple AZs`（**这是问题根源**） | spread `across multiple AZs` |

✅ **防御**：题干描述的**现状**往往就是要**改变**的东西，不是要保留的。

## ② 虚构能力（老模式，持续）

| 题 | 虚构了什么 |
|---|---|
| Q14 | bucket policy 里放 `AmazonS3FullAccess`（那是 IAM 托管策略） |
| Q27 | **Origin Shield 防未授权访问**（它只是多一层缓存） |
| Q28 | placement group **跨 Region** |
| Q37 | **CloudFront 用 DynamoDB 做源站** |

## ③ 同族名字搞混

| 题 | 混淆 | 区分 |
|---|---|---|
| Q2 | Cluster vs Reader endpoint | 写 vs 读 |
| Q26 | CreationPolicy vs UpdatePolicy | 题干 `stack creation` → Creation |
| Q27 | S3 presigned URL vs CloudFront signed URL | `via CloudFront only` → CloudFront 的 |
| Q36 | CGW vs IGW | CGW = VPN 本地端；上公网 = IGW |

## ④ 默认值 / 数字比对

| 题 | 判据 |
|---|---|
| Q13 | Kinesis 默认保留 **24 小时**；`every other day` = **每 48 小时** → 过期 |
| Q31 | VPC/子网 CIDR **/16 ~ /28**；子网**只在一个 AZ**；新子网默认关联主路由表 |
| Q37 | ⭐ **CLI 建的 DynamoDB 表默认不开 Auto Scaling** |

## ⑤ 抓住一个要求，漏了另一个

| 题 | 抓住了 | 漏了 |
|---|---|---|
| Q17 | SMB → FSx | `on-premises` + `same low-latency as local` → Storage Gateway |
| Q22 | 150 MB/s → Bulk？ | ⚠️ `under 15 minutes` 是硬约束 → Expedited |

## ⑥ 授权句

| 题 | 句子 | 解锁 |
|---|---|---|
| Q34 | `If interrupted, transcoded by another instance` + `only needed until backlog reduced` | ⭐ **Spot** |

---

# 四、本轮真知识缺口（新补）

## Glacier 取回三档（笔记里原本完全没有）

| 存储类 | Expedited | Standard | Bulk |
|---|---|---|---|
| **Instant Retrieval** | — | **毫秒级** | — |
| **Flexible Retrieval** | ⭐ **1–5 分钟** | 3–5 小时 | 5–12 小时 |
| **Deep Archive** | ❌ **没有** | 12 小时 | 48 小时 |

- 口诀：**分钟 / 小时 / 半天**
- ⭐ **Provisioned Capacity** = 给 Expedited 买保险；`under all circumstances` / `guaranteed` → 选它；**只配合 Expedited**

## Placement Group 三种

| 类型 | 比喻 | 题干触发词 |
|---|---|---|
| ⭐ **Cluster** | 挤在一起（单 AZ） | `HPC`、`tightly-coupled`、`low latency`、`node-to-node` |
| **Spread** | 每台单独放（每 AZ 最多 7 台） | `small number of critical instances`、防同时宕机 |
| **Partition** | 分小区 | `Hadoop`、`Cassandra`、`Kafka`、`HDFS` |

⚠️ 都**不能跨 Region**。

## NACL 临时端口（Q30）

```
安全组 = 有状态：入站 443 一条就够
NACL   = 无状态：入站 443 + ⭐ 出站临时端口（1024–65535）
```
比喻：打总机（443），对方回电打的是你的手机号（随机临时端口）。

## 其他

| 知识点 | 判据 |
|---|---|
| **EC2 配额**（Q3） | 按 **Region**、On-Demand 按 **vCPU**；解决 = **申请提额**，不是换地方绕过 |
| **CloudFormation**（Q26） | 装好再继续 = ⭐ **CreationPolicy + cfn-signal**；`cfn-init` 是安装；DependsOn 只管顺序 |
| **CloudFront 私有内容**（Q27） | **Signed URL/Cookie + OAC**（阻止直连 S3） |
| **敏感数据存储**（Q29） | **EBS 加密 + S3 SSE/CSE**；快照是备份不是保护 |
| **RI 不用了**（Q32） | **RI Marketplace** 卖掉（TD 说 terminate 能省钱不严谨：RI 是计费承诺） |
| **S3 取部分数据**（Q35） | 只过滤 → **S3 Select**；要转换/打码 → **S3 Object Lambda**；多文件 SQL → **Athena**<br>⭐ **唯一定位对象 = bucket + key** |
| **公网访问不了 EC2**（Q36） | 两查：**公网 IP/EIP** + **路由表到 IGW** |
| **DynamoDB 性能**（Q37） | **DAX** + 开 Auto Scaling；**API Gateway 缓存** |

---

# 五、考前提取题（不写答案）

1. 审计要求 12 小时内取回 + 最便宜 → Glacier 哪档？
2. Deep Archive 能 1 小时内取回吗？
3. Kafka 集群，一组 broker 坏了不影响其他组 → 哪种 placement group？
4. 3 台关键数据库主机不能同时宕机 → 哪种？
5. NACL 全拒绝，EC2 要**主动**访问外网 443 → 开哪两条？
6. Aurora 有 2 个 Replica，要均衡读流量 → 哪个端点？
7. 本地应用用 NFS，想把数据放 AWS 但保持本地低延迟 → 什么服务？
8. CloudFormation 要等 EC2 上软件装好才继续 → 什么属性 + 什么脚本？
