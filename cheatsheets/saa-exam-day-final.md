# 🎯 考前最后一页（2026-10-04 考场前看）

> 来源：`practice/saa/02`–`12` 全部错题 + TD Test 1–4
> 用法：**先盖住右边，自己说答案，再看**（主动提取 94% vs 通读 43%）

---

## 一、每题必做 3 个动作

```
① 约束句扫描  without / does not require / rather than / but / never / must / discarded
               → 这句放宽了什么？排除了什么？（草稿纸记一笔，最后确认都用上了）
② 虚构能力检查 选项【后半句】那个功能真的存在吗？托管服务你进得去吗？
③ 要求计数    题干几个独立要求？类型限定词在第一句，优化目标在最后一句
```

## 二、6 个失分模式 → 防御

| 模式 | 防御 |
|---|---|
| **虚构能力**（最大失分源） | DynamoDB Read Replica ❌ / CloudFront Multi-AZ ❌ / Origin Shield 防未授权 ❌ / placement group 跨 Region ❌ / EFS lifecycle 删文件 ❌ / CloudFront 用 DynamoDB 当源站 ❌ |
| **回声词** | 选项照抄题干的现状描述 → 常是干扰项。**现状往往是要改的东西** |
| **抓一漏一** | 先说「这题要 ___ 类型方案」（relational? block storage? on-premises? API?）再看指标 |
| **选中间项** | `MOST cost-effective` / `LEAST overhead` → 先看**最轻量**的（S3 静态托管 > Beanstalk；Lambda > ECS；Fargate > EKS） |
| **方向读反** | 防火墙/白名单 → 先问「**谁限制谁**」：别人限制我 → 我要固定 IP → **NLB + EIP / Global Accelerator** |
| **创建后忘启用** | WAF 要 associate；DynamoDB Streams 要 enable；IAM DB Auth 要在 RDS 侧开；Cost Tags 要在 Billing 激活；CLI 建 DynamoDB 默认**没** Auto Scaling；ASG 健康检查默认 **EC2** 不是 ELB |

---

## 三、🔴 重复错过的点（错 ≥2 次）

| 题干信号 | 答案 |
|---|---|
| API Gateway 保护后端免受 traffic spikes | **Throttling**（429） |
| 防 S3 **意外**删除 | **Versioning + MFA Delete** |
| ASG 缩容时 5xx | **Deregistration Delay**（默认 300s；请求 15 分钟 → >900s） |
| ASG 先终止谁 | 实例最多的 AZ → 最老启动模板 → 最接近计费小时 |
| Aurora 端点 | **Cluster = 写（Primary）**；**Reader = 读负载均衡** |
| `account` 跨账户访问 S3 | **IAM / bucket policy 跨账户**；`origin`/`domain` 才是 CORS |
| 本地服务器还在 + 低延迟本地缓存 | **Storage Gateway（File Gateway）**，不是 FSx |
| 误删恢复 + 「不影响现有库」 | **PITR 到新实例**（不是原地/Backtrack） |
| Lambda 高并发 `too many connections` | **RDS Proxy** |
| 冷启动不可接受 | **Provisioned Concurrency** |
| Object Lock 给了期限 / 没给期限 | **Retention Period / Legal Hold** |
| DynamoDB **跨账户**备份 | **AWS Backup**（PITR/on-demand 都不行） |

---

## 四、高频判据（一行一个）

**存储**
- Glacier 取回：Flexible **Expedited 1–5 分钟** / Standard 3–5h / Bulk 5–12h；Deep Archive **无 Expedited**，12h / 48h；`guaranteed` → **Provisioned Capacity**
- 模式已知 → **Lifecycle**；模式未知 → **Intelligent-Tiering**
- 高 IOPS **且数据可丢** + cost-effective → **Instance Store**；数据要保住 → io2
- 多实例共享文件 → **EFS**（不是 EBS Multi-Attach）；跨 AZ **block** → FSx for NetApp ONTAP
- 只过滤 → **S3 Select**；要转换/打码 → **S3 Object Lambda**；多文件 SQL → **Athena**
- 上传加速（不缓存）→ **Transfer Acceleration**；分发（缓存）→ **CloudFront**
- 仅通过 CloudFront 访问 → **CloudFront Signed URL/Cookie + OAC**；不改 URL → Signed **Cookie**
- 静态网站 + 自定义域名 → **bucket 名 = 域名**

**计算 / 扩展**
- 可中断（`if interrupted…`、`backlog`）→ **Spot**
- `9 to 5` 可预测 → **Scheduled Scaling**；`randomly/unpredictable` → 定时方案全出局
- Placement：HPC/低延迟 → **Cluster**；少量关键机不同时宕 → **Spread**（每 AZ 7 台）；Hadoop/Kafka/Cassandra → **Partition**
- EC2 配额按 **Region、vCPU** → 申请提额
- CloudFormation 等软件装完 → **CreationPolicy + cfn-signal**
- 超时三数字：API GW **29 秒** / Lambda **15 分钟** / ALB 默认 60 秒
- 容器里的应用访问 S3 → **Task Role**；拉镜像写日志 → Execution Role
- EC2/Lambda/ECS 访问 AWS 服务 → **永远 IAM Role**

**网络**
- NACL 无状态 → 必须放**出站临时端口 1024–65535**；SG 有状态
- 公网连不上 EC2 → **公网 IP/EIP** + **路由表 → IGW**
- 只允许来自 web 层 → **SG 引用 SG**
- VPC Peering **不传递、不 edge-to-edge** → 多 VPC + 本地 → **Transit Gateway**
- gRPC → **只有 ALB**；固定 IP → **NLB**（ALB 不能有 EIP）
- WAF 挂不上 NLB → 前面加 **CloudFront**；有 CloudFront 优先挂 CloudFront
- 没提 on-premises → DX / VPN 全排除；VPN 的 **CGW** 是本地端，上公网是 **IGW**
- Failover 要秒级（DNS TTL 不够）→ **Global Accelerator**
- Geolocation = 按国家（合规）；Geoproximity = 距离 + **bias**；Latency = 最快

**数据库 / 缓存 / 消息**
- 给 **DynamoDB** 加缓存 → **DAX**（强一致读不走 DAX）；给 **RDS** 加缓存 → ElastiCache
- 跨 Region 多活 DynamoDB → **Global Tables**；Region 故障 Aurora → **Global Database**
- 故障转移要 <30s + 读分流 → **迁 Aurora**
- 实时多消费者/重放 → **KDS**（默认保留 **24h**）；落 S3/Redshift 免代码 → **Firehose**
- SNS = 每个订阅者收全部（有 **filter policy**）；SQS = 一条只一个消费者；扇出 = **SNS → 多 SQS**
- 多步骤 + 重试/分支 → **Step Functions**；JMS/AMQP 迁移不改代码 → **Amazon MQ**
- Redshift 只是夜间周末空转 → **暂停/恢复**；偶尔查 → Athena

**安全 / 成本 / 管理**
- 必须**阻止** → SCP / WAF / bucket policy；GuardDuty/Inspector/Macie/Config **只看不拦**
- 即使有完整权限也要拒绝 → **bucket policy Deny + `aws:SourceVpce`**
- Shield 只在题干是 DDoS 时才选
- 自动轮换凭据 → **Secrets Manager**
- 多账户落地 → **Control Tower**；资源共享 → **RAM**
- EBS 快照/AMI 免费自动化 → **DLM**；跨服务 + Vault Lock → **AWS Backup**
- RI 不用了 → **RI Marketplace**
- DR：Backup&Restore（最便宜）→ Pilot Light（建好关着）→ Warm Standby（开着但小）→ Multi-Site；「第二区域不能长期跑计算」→ Backup & Restore

---

## 五、节奏

```
160 分钟 / 65 题 ≈ 2 分 27 秒；留 15 分钟检查
第 40 题看表：已用 >100 分钟 → 加速
超 2 分半 → Mark for review，跳
后半段疲劳最容易错【简单题】→ 深呼吸 10 秒，题干读完
TD Test 4 涨 14 分的唯一变量：仔细读英文题干
```

---

## 六、30 秒自测（盖住答案）

<details><summary>1. 12 小时内取回 + 最便宜 → ?</summary>Deep Archive（Standard 取回 ≤12h，最便宜）；要 1 小时内 → 只有 Flexible Expedited</details>
<details><summary>2. Kafka，一组 broker 坏不影响其他组 → ?</summary>Partition placement group</details>
<details><summary>3. NACL 全拒绝，EC2 主动访问外网 443 → 开哪两条？</summary>出站 443 + 入站 1024–65535</details>
<details><summary>4. Aurora 2 个 Replica 均衡读 → ?</summary>Reader endpoint</details>
<details><summary>5. 本地 NFS 应用，数据放 AWS 保持本地低延迟 → ?</summary>S3 File Gateway</details>
<details><summary>6. 客户防火墙只放行白名单 IP，后端要 L7 路由 → ?</summary>NLB + EIP 在前，ALB 在后（或 Global Accelerator）</details>
<details><summary>7. Lambda 连 RDS 报 too many connections → ?</summary>RDS Proxy</details>
<details><summary>8. API 被流量尖峰打垮后端 → ?</summary>API Gateway Throttling</details>
