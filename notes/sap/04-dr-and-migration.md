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

（待学）

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
