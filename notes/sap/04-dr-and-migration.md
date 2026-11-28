# 弹性与迁移：DR 四档 / 跨区域数据层 / 迁移 7R / 迁移工具

> SAP-C03 W7 · 2026-11-17 起（🚧 进行中：W7-1/W7-2/W7-3 ✅，下一步 W7-4）
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

> 一句话定位：7R 回答**「每个应用怎么迁」**，评估工具回答**「有什么、花多少钱、谁依赖谁、先迁谁」**。

### 3.1 迁移三阶段（AWS 官方框架）

| 阶段 | 要做的事 | 典型工具 |
|---|---|---|
| **Assess（评估）** | 摸清现状、算 TCO、做商业论证 | Migration Evaluator、MRA（迁移就绪评估） |
| **Mobilize（准备）** | 搭 Landing Zone、详细发现与依赖分析、按应用定 7R、**分批（wave）规划** | Application Discovery Service、Control Tower |
| **Migrate & Modernize（迁移与现代化）** | 按 wave 执行迁移，迁完再优化 | MGN、DMS、DataSync…（W7-4） |

> 资金支持：**MAP（Migration Acceleration Program）** 按这三阶段给补贴和专家支持。

### 3.2 7R 对照表（从「改动最少」到「改动最多」）

| 策略 | 中文 | 做什么 | 典型例子 / AWS 工具 |
|---|---|---|---|
| **Retire** | 下线 | 没人用了，直接关掉 | 发现阶段查到长期零访问的系统 |
| **Retain** | 保留 | 暂时不迁（合规、刚升级完、依赖没解决） | 留在本地，以后再说 |
| **Relocate** | 平移 | **整个虚拟化平台**搬上云，VM 不改、运维工具不变 | VMware → **Amazon EVS**（Elastic VMware Service）/ VMware Cloud on AWS |
| **Rehost** | 直接迁移（lift-and-shift） | 服务器原样搬到 EC2 | **MGN** |
| **Replatform** | 平台优化（lift-tinker-shift） | **不改核心代码**，换成托管平台 | 自建 MySQL → RDS；应用 → Beanstalk / 容器 |
| **Repurchase** | 换购（drop-and-shop） | 放弃自建，改用 SaaS | 自建 CRM → Salesforce；Exchange → Microsoft 365 |
| **Refactor / Re-architect** | 重构 | 改代码、改架构，做成云原生 | 单体 → 微服务 / Lambda |

**容易混的三对**：
- **Relocate vs Rehost**：Relocate 搬的是**整个 vSphere 平台**（几天搬几百台，不碰每台 VM）；Rehost 是**逐台服务器**复制到 EC2
- **Rehost vs Replatform**：数据库搬到 EC2 上自己装 = Rehost；搬到 RDS = Replatform。看「有没有换成托管服务」
- **Replatform vs Refactor**：看**改不改应用代码**。只改配置/连接串 = Replatform；拆服务、改架构 = Refactor

**做题信号**：

| 题干关键词 | 选 |
|---|---|
| 「最快」「数据中心合同快到期」「不改应用」 | Rehost（MGN）；VMware 且要保留现有工具 → Relocate |
| 「减少运维负担」「但不能改代码」 | Replatform |
| 「弹性伸缩、按需付费、敏捷」「愿意投入开发」 | Refactor |
| 「许可证太贵」「功能有现成 SaaS」 | Repurchase |

### 3.3 评估工具

| 工具 | 解决什么问题 | 关键点 |
|---|---|---|
| **Migration Evaluator**（原 TSO Logic） | **商业论证 / TCO**：搬上云要花多少钱 | 无 agent 采集器，按**实际利用率 right-size**，考虑**许可证**（BYOL vs 包含许可证）。「给 CFO 看成本」「几周出报告」 → 它 |
| **Application Discovery Service（ADS）** | **详细清单 + 依赖关系**，用于 wave 规划 | 两种采集方式，见下表 |
| **Migration Hub** | **统一追踪**：所有迁移工具的进度汇总到一个看板 | 只追踪，不执行迁移 |
| Migration Hub **Strategy Recommendations** | 分析应用（含二进制/源码），**推荐 7R 策略** | — |
| Migration Hub **Refactor Spaces** | 为**绞杀者模式（strangler fig）**自动搭好路由基础设施 | 自动建 API Gateway / NLB / TGW，逐步把功能切到新服务 |
| **AWS Transform** | **AI agent 驱动的迁移与现代化**（VMware 迁移、大型机、.NET 移植） | 2025 年推出的新服务，逐步接替 Migration Hub 系列的部分功能 |

> ⚠️ 服务变化：Migration Hub 系列 2025 年起逐步对新客户关闭，AWS 推 AWS Transform。SAP-C03 题目可能出现新名字，也可能还是老名字，两边都要认得。

**ADS 两种采集方式**（必考对比）：

| | Agentless Collector | Discovery Agent |
|---|---|---|
| 部署 | 在 **VMware vCenter** 里部署一个 OVA | **每台服务器**装 agent |
| 适用 | 只限 VMware 虚拟机 | 物理机、VM、任何环境 |
| 采集内容 | 配置、CPU/内存/磁盘利用率 | 以上 + **运行中进程** + **网络连接** |
| **依赖关系** | ❌ 做不了 | ✅ 能画出服务器之间谁连谁 |

> 记忆：**要依赖关系 → 必须装 agent**（只有 agent 能在系统里看到网络连接和进程）。
> 数据可以导出到 S3，再用 **Athena + QuickSight** 分析。

### 3.4 绞杀者模式（Strangler Fig）—— Refactor 的标准做法

- 不一次性重写。在单体前面放一个**路由层**（API Gateway / ALB），**按 URL 路径**把已经拆出来的功能转发给新服务，其余路径继续给单体
- 逐个功能迁移，每次只切一条路径，出问题**只回滚这一条路由**
- 单体越来越小，最后下线
- 和「一次性重写 + Route 53 加权路由」的区别：加权路由是**按请求比例**切整个应用，不是**按功能**切；而且要等全部重写完才能开始

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

**W7-2 批改（2026-11-26）**：W7-2 2.5/4，复习 1.8/2（R1 ✅ 已掌握）

| # | 我的答案 | 问题 | 正确理解 |
|---|---|---|---|
| Q1 | Switchover；不用 Failover / detach | 选对，但**没说 Failover 为什么不行** | Failover 用于主区域故障，**不等未复制数据同步完**就切，RPO = 复制延迟，可能丢订单。Switchover 先同步再切，RPO = 0。detach + promote 会脱离全局库，复制要重建 |
| Q2 | Global Tables 最终一致 → 改强一致，代价是性能 | 术语不准 | 默认模式是**异步复制 + LWW 冲突解决**；改为 **MRSC（多区域强一致）**，需要 **3 个区域或 2 区域 + 见证区域**，代价具体是**写入延迟升高**（每次写要跨区域同步确认） |
| Q3 | B（MGN） | ❌ **C 选项就是 DRS，没认出来** | 答案 C。A：RPO 1 小时不满足秒级；B：MGN 是**一次性迁移**，迁完就切换，不是持续 DR（自己也意识到了，但还是选了）；D：300 台常驻 EC2 成本高，而且 rsync 是文件级同步，做不到 Windows 系统级的一致恢复 |
| Q4 | FIS + CloudWatch 告警 + Resilience Hub | ✅ 基本完整 | 补充：用 FIS 场景库的 **AZ Availability: Power Interruption**；告警配置为实验的**停止条件** |
| R2 | GW 只作用于 VPC 内；Interface ENI 经 DX 可达；Resolver 入站端点 | DNS 少一步 | 本地 DNS 要配**条件转发**：把 `s3.<region>.amazonaws.com` 转发到 Resolver 入站端点的 IP。或者不改 DNS，直接用 endpoint 专属域名 |

> 🔁 新模式：**排除法排到最后，宁可选「自己知道有问题」的熟悉选项，也不选没认出的选项**（Q3）。考场规则：如果自己都说出了某个选项的硬伤（「这是一次性的」），那它就一定不是答案，回头重读剩下的选项。

**W7-3 批改（2026-11-28）**：W7-3 2.2/4，复习 2.3/3

| # | 我的答案 | 问题 | 正确理解 |
|---|---|---|---|
| Q1 | Migration Evaluator；ADS 是查依赖不是做商业论证 | ✅ 方向对，理由少一半 | 题干还有一个硬条件「不在每台服务器上装软件」：ADS 要拿全数据就得装 Discovery Agent，Agentless Collector 又只能采集 VMware；而且 ADS **不出 TCO/许可证成本报告**。Migration Evaluator 不用装 agent，几周就能出 right-size 和许可证成本 |
| Q2 | Discovery Agent；Agentless 探测不了物理机的进程 | 理由不准 | Agentless 有**两处**不行：① 只能通过 vCenter 采集，**物理机根本采集不到**；② 就算是 VMware VM，也**拿不到网络连接** → 画不出依赖关系。要依赖关系就必须装 agent |
| Q3 | a Replatform / b Refactor / c Retain / d Retire ✅ / e Rehost | ❌ **5 个错 4 个** | a **Repurchase**（改用 SaaS，判断点是自建 → 买现成服务）；b **Replatform**（换成托管服务、代码不改；Refactor 必须改代码）；c **Relocate**（整个 VMware 平台搬上云，保留 vCenter → Amazon EVS）；e **Retain**（不能动 = 暂时不迁。而且大型机没法 rehost 到 EC2，要走 AWS Transform 重构） |
| Q4 | B | 选对，但**题目要求说出另外三个错在哪，没写** | A：一次性重写，加权路由按**请求比例**切换整个应用，不能按功能切，回滚也只能整体回滚；C：Rehost 根本没有拆分，和需求无关；D：容器化 + 蓝绿部署还是**整个单体**一起发布，不能按功能切换 |
| R3 | 禁止删除权限边界策略，否则会越界 | 第 4 次说不准 | 要禁止的是 `iam:DeleteRolePermissionsBoundary`（**从角色上摘掉边界**，不是删除策略本身）。完整做法：① `iam:CreateRole` 加条件 `iam:PermissionsBoundary` = 指定边界；② Deny `DeleteRolePermissionsBoundary`；③ Deny 修改边界策略本身（`CreatePolicyVersion` / `DeletePolicy` 等）。不禁止 ① 和 ② 的后果：开发者把自己建的角色的边界摘掉，角色就拿到身份策略的全部权限，实现**权限提升** |

> R1（MGN = 一次性迁移，切换后停止复制；DRS = 长期持续复制 + 演练 + 回切）✅、R2（Failover 不等复制完就切，RPO = 复制延迟；Switchover 同步完再切，RPO = 0）✅，从轮换中移除。
> 🔁 **7R 判断顺序**（治 Q3）：① 还用不用？不用 → Retire ② 现在能不能动？不能 → Retain ③ 换成别人的 SaaS？→ Repurchase ④ 整个 VMware 平台一起搬？→ Relocate ⑤ 改不改代码？改 → Refactor ⑥ 换成托管服务？换 → Replatform，不换 → Rehost
> 🔁 Q4 再次出现「只选不排除」。题目明确要求说出其他选项错在哪时，没写就按半分算。

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

**W7-2 检查题（2026-11-25 出，2026-11-26 已作答，批改见第 5 节）**

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

**W7-3 检查题（2026-11-28 出，同日已作答，批改见第 5 节）**

1. 公司有 800 台本地服务器，计划 18 个月内迁到 AWS。CFO 要求先看一份按实际利用率 right-size 后的 TCO 商业论证，几周内交付，不希望在每台服务器上装软件。用什么工具？为什么不用 Application Discovery Service？
2. 进入 Mobilize 阶段，需要知道哪些服务器之间有网络依赖，以便把相互依赖的服务器放进同一批迁移。环境里有 VMware VM，也有物理机。ADS 用哪种采集方式？另一种为什么不行？
3. 下面五个应用分别属于 7R 中的哪一种？
   - a. 自建 Exchange 邮件服务器，改用 Microsoft 365
   - b. VM 上的 Oracle 数据库，迁到 Amazon RDS for Oracle，应用代码不改
   - c. 200 台 VMware VM，数据中心合同 3 个月后到期，运维团队要继续用 vCenter 和现有脚本
   - d. 一套报表系统，近 6 个月访问日志几乎为零
   - e. 一台大型机，因监管审批要到明年才能动
4. 一个 Java 单体电商应用要逐步拆成微服务。要求迁移期间对外 URL 不变，每次只切一个功能，切出问题能只回滚那个功能。
   - A. 在新环境把整个应用重写成微服务，完成后用 Route 53 加权路由从 10% 逐步切到 100%
   - B. 在单体前面放 API Gateway，按 URL 路径把已拆出的功能路由到新的 Lambda/容器服务，其余路径继续转发到单体
   - C. 用 MGN 把单体 rehost 到 EC2，配置 Auto Scaling 应对流量
   - D. 把单体容器化部署到 ECS，用蓝绿部署发布新版本
   选哪个？其他三个分别错在哪？

**同时出的复习题**

- R1.（W7-2 Q3 错）MGN 和 DRS 用同一套复制技术，区别在哪？题干出现什么词时选 DRS？
- R2.（W7-2 Q1 没说清）Aurora Global Database 的 Failover 为什么可能丢数据？Switchover 为什么不会？
- R3.（W5 错 3 次）给开发者授予创建角色的权限、并用权限边界限制他们时，还必须禁止哪个操作？不禁止会怎样？
