# 弹性与迁移：DR 四档 / 跨区域数据层 / 迁移 7R / 迁移工具

> SAP-C03 W7 · 2026-11-17 起（🚧 进行中：W7-1 ✅ 已完成，下一步 W7-2）
> **Domain 4（Resilience, Migration）**
> SAA 基础见 `notes/saa/` 中 DR 相关章节

---

## 1. W7-1 灾难恢复（DR）四档

### 1.0 RPO 和 RTO：先分清各由哪一层决定

| 指标 | 含义 | 由哪一层决定 |
|---|---|---|
| **RPO** | 最多能丢多久的数据 | **数据层**：数据怎么复制、多久复制一次 |
| **RTO** | 最多能停机多久 | **计算层 + 切换层**：备用环境有没有在跑、流量怎么切过去 |

> SAP 常把两个指标拆开考：比如「数据不能丢，但停机 2 小时可以接受」。这种情况数据层要持续复制，计算层却可以不启动。
> 「四档」只是常见组合的打包，做题时要分别看数据层和计算层各自需要什么。

### 1.1 四档对照表

| 档位 | 灾备区域里有什么 | RPO | RTO | 成本 |
|---|---|---|---|---|
| **Backup & Restore** | 只有备份（快照 / AMI / S3 副本）+ IaC 模板 | 小时级（取决于备份频率） | 数小时～24h | 💲 |
| **Pilot Light** | **数据层持续复制并在线**；计算层**没在跑**（只有 AMI，或者 ASG 期望容量 = 0） | 秒～分钟 | 数十分钟～小时 | 💲💲 |
| **Warm Standby** | **整套环境都在跑，只是规模缩小**，可以马上接流量，再扩容 | 秒～分钟 | 分钟级 | 💲💲💲 |
| **Multi-site Active/Active** | 两边都满配，都在接流量 | ≈0 | ≈0 | 💲💲💲💲 |

**Pilot Light 和 Warm Standby 怎么区分**：只问「灾备区域现在能不能直接处理一个请求？」
- 不能，要先启动或部署计算资源 → Pilot Light
- 能，只是容量小 → Warm Standby

> 类比：Pilot Light 像燃气热水器的长明火（火种是数据，要点起大火才能用）；Warm Standby 是小火一直开着。

### 1.2 每一档在 AWS 上用什么实现

**Backup & Restore**
- AWS Backup 配**跨区域 + 跨账户**复制（复制到独立的灾备账户，防止主账户被攻破后备份也被删）
- **Backup Vault Lock（合规模式）**：开启后连 root 都删不掉备份。看到「勒索软件 / 不可篡改备份」就想到它
- 用 CloudFormation / CDK 在灾备区域重建；AMI 跨区域复制；S3 CRR
- RPO ≈ 备份间隔 + 跨区域复制延迟

**Pilot Light**
- 数据层：Aurora Global Database / 跨区域只读副本 / DynamoDB Global Tables / S3 CRR
- 计算层：AMI 提前复制好；ASG 已建好但容量设为 0，或只保留 IaC 模板
- 切换：提升（promote）只读副本为主库 → 扩容 ASG → 切 DNS

**Warm Standby**
- 和 Pilot Light 一样有数据复制，另外有一套小规模但完整的 ALB + ASG 在跑
- 切换：提升数据库 + ASG 扩容 + 切流量

**Multi-site**
- Route 53 延迟路由 / 加权路由，或者 Global Accelerator
- 数据层要能多区域写：DynamoDB Global Tables（冲突时最后写入者胜出）；Aurora Global 默认只有一个写入点（开了 write forwarding，写入实际上仍转发到主区域）

### 1.3 SAP 加分点：切换机制要「静态稳定」

- **控制平面**（创建、修改资源的 API，比如改 DNS 记录、新建实例）：大规模故障时可能不可用或被限流
- **数据平面**（已经在运行的功能，比如健康检查判定、按现有配置解析 DNS）：可用性更高
- 最佳实践：**故障切换只依赖数据平面**，资源提前准备好，不在出事时临时创建

| 需求关键词 | 选择 |
|---|---|
| 自动切换、简单 | Route 53 failover 路由 + 健康检查 |
| 人工可控、高可靠的切换开关 + 就绪检查 | **Route 53 ARC**（路由控制走数据平面） |
| 切换要快，不想受客户端 DNS 缓存（TTL）影响 | **Global Accelerator**（固定 Anycast IP） |
| Pilot Light 切换时担心 EC2 容量不够 | **按需容量预留（ODCR）** |

> 有些题的 RTO 从数字上看 Pilot Light 也能满足，但题干强调「切换过程不能依赖在故障区域调用 API」，这时就要往 Warm Standby 或 ARC 的方向选。

### 1.4 做题算法

1. 把 RPO 和 RTO 的数字圈出来，同时找「most cost-effective」这类字眼
2. 从便宜的一档往贵的一档试，第一个能满足的通常就是答案（干扰项常是「能满足，但更贵」的那一档）
3. 单独检查数据层：RPO 要求秒级时，只做定时备份一定不够，必须有持续复制

---

## 2. W7-2 Aurora Global / DynamoDB Global Tables / Elastic Disaster Recovery / FIS

> 一句话定位：前两个解决**数据层跨区域**，DRS 解决**改不动的服务器怎么做 DR**，FIS 解决**怎么证明 DR 真的能用**。

### 2.1 Aurora Global Database —— 单写多读

- 1 个主区域（读写）+ 多个从区域（只读），**存储层复制**，延迟通常 < 1 秒 → RPO ≈ 1 秒，RTO ≈ 1 分钟级
- ⚠️ **只有一个写入点**。题干要「多区域同时写关系型数据」→ Aurora Global 不行（看 Aurora DSQL 或改用 DynamoDB Global Tables）
- 两种切换，考点是**会不会丢数据**：

| 操作 | 场景 | 数据 | 拓扑 |
|---|---|---|---|
| **Switchover**（计划内切换） | 演练、区域轮换，主区域健康 | 先同步完再切，**RPO = 0** | 保持全局库，旧主变从 |
| **Failover**（计划外故障转移） | 主区域挂了 | 丢掉未复制的那点数据（RPO = 复制延迟） | 保持全局库，旧主恢复后自动变从 |
| 手动 detach + promote（老办法） | — | 同上 | **脱离全局库**，变成独立集群，要重建复制 |

- **Write forwarding**：从区域的应用直接往本地集群写，Aurora 把写**转发**到主区域执行。省去应用改连接串，但写延迟 = 跨区域往返；**不是多主**
- **全局写入端点（Global writer endpoint）**：一个固定域名始终指向当前主集群，切换后应用不用改连接配置
- **无头从集群（headless secondary）**：从区域只有存储、没有实例 → 便宜，对应 **Pilot Light**；切换时再加实例

### 2.2 DynamoDB Global Tables —— 多区域多活

- 任何副本区域**都能读写**，默认**异步复制**（通常 < 1 秒），冲突用 **最后写入者胜出（LWW）**
- 默认模式 RPO 不是 0（区域挂掉时还没复制出去的写会丢）
- **MRSC（多区域强一致，2025 GA）**：3 个区域（或 2 区域 + 1 见证区域），写入同步确认 → **RPO = 0**、任何区域都能强一致读；代价是**写延迟变高**
- 避免冲突的设计：**写入亲和（write affinity）**——同一个用户/同一条数据固定在一个「主场区域」写，其他区域只读或故障时接管
- 坑：DAX 缓存**不跨区域**，每个区域单独建

| 需求关键词 | 选择 |
|---|---|
| 关系型、单写、秒级 RPO、读本地化 | Aurora Global Database |
| 多区域都要写、键值访问 | DynamoDB Global Tables（默认） |
| 多区域都要写 + **RPO = 0 / 强一致** | DynamoDB Global Tables **MRSC** |
| 计划内切区域、不能丢一条数据 | Aurora **Switchover** |

### 2.3 AWS Elastic Disaster Recovery（DRS）—— 服务器级 DR

- 在源服务器装 Agent，**块级持续复制**到目标区域的**暂存区（staging area）**：低配复制服务器 + 便宜的 EBS 卷
- 平时不跑真正的业务实例 → 成本接近 Pilot Light；RPO **秒级**、RTO **分钟级**
- 演练（drill）/ 故障时一键启动恢复实例；支持**时间点恢复**（被勒索软件加密后回到加密之前）和**回切（failback）**
- 源可以是：本地机房、其他云、**另一个 AWS 区域的 EC2**
- 识别信号：「几百台本地服务器 / 应用不能改造 / 要秒级 RPO / 成本低」→ DRS
- 和 **MGN** 是同一套复制技术：MGN 用于**一次性迁移**（迁完就切），DRS 用于**长期 DR**（一直复制、随时能拉起）
- 和 **AWS Backup** 区别：Backup 是定时快照（RPO 小时级），DRS 是持续复制（RPO 秒级）

### 2.4 AWS Fault Injection Service（FIS）—— 混沌工程

- 用**实验模板**有控制地注入故障：停止/终止 EC2、RDS/Aurora 故障转移、ECS/EKS 任务停止、网络中断、API 限流/报错、EBS I/O 暂停等
- **停止条件**：绑定 CloudWatch 告警，指标恶化就**自动中止**实验 → 可以在生产环境安全地做
- **场景库**：预置的复合场景，比如「AZ 断电（AZ Availability: Power Interruption）」「跨区域连接中断」
- 识别信号：「验证系统在故障时是否按预期恢复」「定期演练」「出问题自动停止」→ FIS
- 搭配：**Resilience Hub** 负责**评估**（按你定的 RTO/RPO 目标给应用打分、给建议），FIS 负责**实际打一下**验证



## 3. W7-3 迁移 7R + 评估工具

（待学）

## 4. W7-4 MGN / DMS·SCT / DataSync / Snow / Transfer Family

（待学）

---

## 5. 我的失分点（W7）

**W7-1 批改（2026-11-24）**：W7-1 4.5/5，W6 复习 2/3

| # | 我的答案 | 问题 | 正确理解 |
|---|---|---|---|
| Q1 | Pilot Light + 跨区域只读副本 | 能满足要求，但**不是最便宜** | RTO 4h 已经够 Backup & Restore 用。RPO 15 分钟靠 **RDS 跨区域自动备份复制**（快照 + 事务日志一起复制到灾备区域，可以在那边做 PITR）就能满足，不用常驻副本。按做题算法「从便宜的一档往上试」，先试的应该是 Backup & Restore |
| Q5 | 换成 ARC 或 Global Accelerator | **ARC 解决不了** | ARC 的路由控制最终还是改 Route 53 的应答，**本质上仍是 DNS**，客户端缓存问题照样存在。只有 **Global Accelerator** 不依赖 DNS（固定 Anycast IP，在网络层切换）。ARC 解决的是「切换开关可靠、走数据平面」，不是缓存 |
| R2 | MACsec，因为 VPN 封装耗资源 | 理由不够硬 | 关键数字：**单条 IPsec 隧道上限约 1.25 Gbps**，要跑满 10G 就得多隧道 + TGW ECMP。MACsec 在二层加密，线速运行。⚠️ 只有**专用连接（Dedicated）**才支持 MACsec，托管连接（Hosted）不支持 |
| R3 | GW Endpoint 是 VPC 内用的；DNS 用 Interface Endpoint | 没说清原理，DNS 部分答非所问 | GW Endpoint 是**路由表目标**（prefix list），只对 VPC 内发起的流量生效，**不能传递**给从 DX/VGW/TGW 进来的流量。要用 **S3 Interface Endpoint**（有私有 IP 的 ENI，从 DX 能到达）。DNS：用 endpoint 专属域名（`bucket.vpce-xxx.s3.<region>.vpce.amazonaws.com`），或者开启 Private DNS + **Route 53 Resolver 入站端点**，让本地 DNS 转发过来 |

> 🔁 重复模式：**「只知道选哪个，说不出为什么不选另一个」**（Q5、R2、R3）。SAP 的干扰项都是「看起来也行」的那个，答题时要能说出它为什么不行。

## 6. 自测题（不写答案）

**W7-1 检查题（2026-11-17 出，2026-11-24 已作答，批改见第 5 节）**

1. 订单系统要求 RPO 15 分钟、RTO 4 小时，成本最低，数据库是 RDS MySQL。选哪一档？数据层具体怎么做？
2. 支付公司要求 RPO 几乎为 0、RTO 10 分钟；灾备区域平时不接任何生产流量，在这个前提下成本尽量低。
   - A. AWS Backup 每小时跨区域复制，灾备区域用 CloudFormation 重建
   - B. Aurora Global Database；灾备区域只保留 AMI，ASG 容量为 0；切换时提升从集群、扩容 ASG
   - C. Aurora Global Database；灾备区域运行一套小规模但完整的 ALB + ASG；切换时提升从集群、扩容 ASG
   - D. 两个区域都满配，DynamoDB Global Tables 双活
3. 用一句话说明 Pilot Light 和 Warm Standby 的本质区别。
4. 担心勒索软件攻破生产账户后连备份一起删掉，备份怎么设计？至少两个关键措施。
5. 现在用 Route 53 failover 路由，演练时发现部分客户端切换后十几分钟还在访问旧区域。原因是什么？换成什么？

**同时出的 W6 复习题（2026-11-24 已作答）**

- R1. 有状态防火墙集中检查后出现间歇性丢包，原因和修法？
- R2. 10 Gbps DX 要加密且不降吞吐，选什么？为什么不选 VPN？
- R3. 本地经 DX 访问 S3，为什么 Gateway Endpoint 不行？DNS 怎么办？

**W7-2 检查题（2026-11-25 出，待作答）**

1. 电商用 Aurora MySQL Global Database（东京主、新加坡从）。每季度要做一次区域切换演练，要求演练过程中不能丢任何一笔订单，演练后全局复制关系要保留。用哪个操作？为什么不用另外两种方式？
2. 游戏公司玩家分布在美、欧、亚三地，要求每个区域都能低延迟写入玩家状态；业务能接受极少量冲突时以最后一次写入为准。选什么？如果需求改成「任何区域故障都不能丢已确认的写入」，要怎么调整？代价是什么？
3. 制造企业有 300 台本地 Windows/Linux 服务器，应用老旧无法改造，要求 RPO 秒级、RTO 30 分钟，灾备平时成本尽量低。
   - A. AWS Backup 每小时备份到 AWS，灾备时从备份恢复
   - B. AWS Application Migration Service 把服务器复制到 AWS
   - C. AWS Elastic Disaster Recovery，复制到暂存区，灾备时启动恢复实例
   - D. 在 AWS 上提前建好 300 台同规格 EC2，用 rsync 每分钟同步
4. SRE 团队要在生产环境每月验证「单个 AZ 断电时应用能自动恢复」，要求如果错误率超过阈值立刻停止演练。用什么服务、怎么配？如果还要先知道「应用当前设计能不能达到 RTO 15 分钟的目标」，再加什么？

**同时出的复习题**

- R1.（W7-1 错题）为什么 Route 53 ARC 解决不了客户端 DNS 缓存问题？什么能解决？
- R2.（W6 R3 错题）本地经 DX 访问 S3，为什么必须用 Interface Endpoint？本地 DNS 要怎么配？
