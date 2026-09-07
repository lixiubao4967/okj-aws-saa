# VPC 互联（Peering / TGW / VPN / Direct Connect / Endpoints）

> P2 网络 · 2026-09-08
> SAA 场景题重灾区，也是 SAP-C02 的核心。学扎实一次赚两回。

---

## 1. VPC Peering —— 两两拉专线

两个 VPC 之间建立私网直连，流量走 AWS 骨干网。**支持跨账户、跨 Region。**

### 🚨 三条必考限制

```
① CIDR 不能重叠             ← 重叠了就建不了，无解
② 不可传递（non-transitive） ← A↔B 通、B↔C 通，A↔C 依然不通！
③ 双方路由表都要手动加路由    ← 只加一边不通
```

### ② 不可传递 = 它的死穴

要让 N 个 VPC 全互通，需 **N(N−1)/2** 条连接：

| VPC 数 | Peering 连接数 |
|---|---|
| 3 | 3 |
| 5 | 10 |
| 10 | **45** |
| 20 | **190** 😱 |

### 其他要点

- **不支持边缘到边缘路由**：不能借道对方 VPC 的 IGW / NAT GW / VPN 出网
- ✅ 无单点故障、无带宽瓶颈、无额外费用（只付跨 AZ/跨 Region 流量费）
- ✅ 适用：**VPC 数量少（2–3 个）且拓扑简单**

---

## 2. Transit Gateway（TGW）—— 网络路由器

**专门解决 Peering 的 N² 爆炸问题。** 区域级网络中转枢纽（hub-and-spoke）。

```
        VPC-A    VPC-B    VPC-C
          \        |        /
           \       |       /
            +--- TGW ---+          ← 只需 N 条连接
            /      |      \
       本地DC    VPN     其他Region
      (via DX)          的 TGW
```

| 特性 | 说明 |
|---|---|
| **传递路由** | ✅ **支持**（vs Peering 的最大区别） |
| 可连接 | VPC、Site-to-Site VPN、Direct Connect Gateway、**其他 Region 的 TGW**（TGW Peering） |
| 网络分段 | 支持**多张路由表** → 可做隔离（Prod / Dev 互不可见） |
| **IP 组播** | ✅ **AWS 唯一支持 multicast 的服务** ← 考点 |
| 跨账户共享 | 靠 **AWS RAM**（Resource Access Manager） |
| CIDR 重叠 | ❌ 依然不允许 |
| 费用 | 按连接数（attachment）+ 流量，比 Peering 贵 |

> 🔑 **判据**：`几十个 VPC`、`简化网络管理`、`hub-and-spoke`、`集中式`、`传递路由`、`组播` → **TGW**

---

## 3. Site-to-Site VPN —— 走公网的加密隧道

连接本地数据中心 ↔ AWS，**走公共互联网**，IPsec 加密。

### 两端组件（名字必须记准）

| 位置 | 组件 |
|---|---|
| **客户侧**（你的机房） | **Customer Gateway（CGW）** |
| **AWS 侧** | **Virtual Private Gateway（VGW）** 或挂到 TGW |

### 要点

- 每个 VPN 连接自动创建 **2 条隧道**（终结在不同 AZ）→ 天生冗余
- **每条隧道上限 1.25 Gbps**
- ⚡ **分钟级搭建完成**、便宜
- ❌ 带宽和延迟**不稳定**（受公网影响）

---

## 4. Direct Connect（DX）—— 物理专线

从机房经运营商拉物理线路直连 AWS，**完全不走公网**。

| 特性 | 说明 |
|---|---|
| 带宽 | Dedicated：1 / 10 / 100 Gbps；Hosted：50 Mbps – 10 Gbps |
| **搭建周期** | 🚨 **1 个月以上**（要拉物理线） |
| 延迟 | 一致、可预测（最大价值） |
| 成本 | 月租贵，但**出网流量单价明显低于公网** → 大流量反而省钱 |
| **加密** | 🚨 **默认不加密！** |
| 高可用 | 🚨 **单条 DX 不高可用** |

### 🚨 三个必考结论

1. **题干说「最快 / 立刻 / 几天内」→ 绝不能选 DX**（要一个月）
   → 选 **VPN 先顶上**，DX 后续再上
2. **DX 需要加密** → **在 DX 上再跑一层 IPsec VPN**（DX + VPN）
3. **DX 高可用方案**（成本从高到低）：
   - 最佳：**两条 DX，接在不同的 DX Location**
   - 经济：**一条 DX + 一条 Site-to-Site VPN 作备份** ← 考试最常见答案

### Direct Connect Gateway

让**一条** DX 连接**多个 Region 的多个 VPC**（否则一条 DX 只能连一个 VPC）。

### 虚拟接口（VIF）

| VIF 类型 | 用途 |
|---|---|
| **Private VIF** | 访问 VPC 内私有资源 |
| **Public VIF** | 访问 AWS **公共**服务（S3、DynamoDB 公网端点） |
| **Transit VIF** | 连接 **Transit Gateway** |

### VPN vs DX 决策表

| 需求 | 答案 |
|---|---|
| 快速上线 / 临时 / 预算有限 | **VPN** |
| 一致的低延迟 / 高带宽 / 大流量降本 | **DX** |
| 需要加密 + 专线 | **DX + VPN** |
| DX 的备份链路 | **VPN** |

---

## 5. ⭐ VPC Endpoints（最高频）

**核心价值：VPC 内资源私有访问 AWS 服务，流量全程留在 AWS 网络内，不经互联网。**

默认情况下私有子网 EC2 访问 S3 要走 `NAT GW → IGW → S3 公网端点`（绕出去再绕回来，慢 + 付 NAT 流量费）。Endpoint 抄近道。

| | **Gateway Endpoint** | **Interface Endpoint（PrivateLink）** |
|---|---|---|
| 支持的服务 | 🚨 **只有 S3 和 DynamoDB** | **几乎所有其他 AWS 服务** |
| **费用** | 🚨 **完全免费** | 按小时 + 流量收费 |
| 实现方式 | 路由表加一条**前缀列表**路由 | 子网内创建 **ENI + 私有 IP** |
| 受 SG 控制 | ❌ 不能（用 Endpoint Policy / NACL） | ✅ **可以挂 SG** |
| **能否从本地 DC 访问** | 🚨 **不能** | 🚨 **能**（经 DX / VPN） |
| 跨 VPC / 跨 Region | ❌ 不能 | 部分支持 |

### 两个高频判据

```
私有子网访问 S3，想省掉 NAT GW 的费用
    → Gateway Endpoint（免费！）

本地数据中心经 DX/VPN 私有访问 S3 或其他服务
    → Interface Endpoint（Gateway Endpoint 做不到）
```

### AWS PrivateLink

把**你自己的服务**暴露给成千上万个其他 VPC，**无需 Peering、无需 CIDR 不重叠**。

```
服务提供方：NLB + 创建 Endpoint Service
消费方：    创建 Interface Endpoint
```

> 🔑 题干「向数百个客户 VPC 提供我们的服务」+「不想管 Peering / IP 冲突」→ **PrivateLink + NLB**

---

## 6. ⭐ 排除法速查（本节解题主武器）

每个服务都有一个「一票否决」的硬限制。**先排除，再从剩下的挑最简单的。**

| 看到这个约束 | 立刻排除 |
|---|---|
| **CIDR 重叠** | Peering、TGW（都不允许）→ 只能 **PrivateLink** |
| 「需要传递路由」 | Peering |
| 「几天内上线」 | Direct Connect（要一个月） |
| 「S3/DynamoDB 之外的服务」 | Gateway Endpoint |
| 「本地 DC 要私有访问」 | Gateway Endpoint |
| 「要挂 Security Group」 | Gateway Endpoint、NAT Gateway |
| 「要免费」 | Interface Endpoint |
| 「要加密」 | 裸 Direct Connect |
| 「组播 multicast」 | 除 TGW 外全部 |

---

## 7. 测验（10 题）

**Q1.** 公司有 12 个 VPC 需要全互通，且未来还会增加。要求**简化网络管理**。
A) 建 66 条 VPC Peering　B) Transit Gateway　C) 每个 VPC 装 NAT GW　D) VPC Endpoints

**Q2.** VPC-A 与 VPC-B 已建 Peering，VPC-B 与 VPC-C 已建 Peering。VPC-A 能否访问 VPC-C？
A) 能，Peering 支持传递　B) 不能，Peering 不可传递　C) 能，但要开启传递选项　D) 取决于路由表

**Q3.** 私有子网的 EC2 需要频繁读写 S3，目前走 NAT Gateway，**账单里 NAT 流量费很高**。最佳优化？
A) 换更大的 NAT GW　B) 创建 S3 **Gateway Endpoint**　C) 创建 S3 Interface Endpoint　D) 把 EC2 移到公有子网

**Q4.** 本地数据中心已有 Direct Connect，现在需要**从本地私有访问 Kinesis**（不经互联网）。
A) Gateway Endpoint　B) Interface Endpoint　C) VPC Peering　D) Public VIF + NAT GW

**Q5.** 公司需要连接本地机房到 AWS，要求**两周内上线**，预算有限，可接受带宽波动。
A) Direct Connect　B) Site-to-Site VPN　C) Direct Connect + VPN　D) Transit Gateway

**Q6.** 金融公司要求本地到 AWS 的链路**既是专线（低延迟稳定）又必须加密**。
A) 只用 Direct Connect　B) 只用 VPN　C) **Direct Connect 上跑 IPsec VPN**　D) Peering

**Q7.** 已有一条 Direct Connect，架构师要**以最低成本消除单点故障**。
A) 再买一条 DX 接在同一 Location　B) 再买一条 DX 接在不同 Location　C) 配置 Site-to-Site VPN 作为备份　D) 无需操作，DX 自带高可用

**Q8.** SaaS 公司要把自己的服务提供给**数百个客户 VPC**，客户的 VPC CIDR **可能与自己重叠**，且不想维护大量 Peering。
A) VPC Peering　B) Transit Gateway　C) **PrivateLink + NLB**　D) 公网 ALB + 白名单

**Q9.** 以下哪项**只能**用 Transit Gateway 实现？
A) 跨账户连接　B) 跨 Region 连接　C) **IP 组播（multicast）**　D) 私网互通

**Q10.** 关于 Gateway Endpoint，下列**错误**的是？
A) 只支持 S3 和 DynamoDB　B) 完全免费　C) 通过路由表前缀列表工作　D) **可以从本地数据中心经 DX 访问**
