# 混合网络：TGW 进阶 / Direct Connect 进阶 / Route 53 Resolver / PrivateLink + VPC Lattice

> SAP-C03 W6 · 2026-11-09 – 11-16
> **Domain 1（Architecture Design）25% / Domain 4（Resilience, Migration）15%**
> SAA 基础见 `notes/saa/09-vpc-connectivity.md`

---

## 0. 四个层次

| 层次 | 服务 | 管什么 |
|---|---|---|
| 物理链路 | **Direct Connect** / VPN | 本地 ↔ AWS 的线 |
| 三层路由 | **Transit Gateway** | 网络之间怎么走 |
| 名字解析 | **Route 53 Resolver** | 两边互相解析域名 |
| 服务暴露 | **PrivateLink** / **VPC Lattice** | 只开放某个服务 |

> 🎯 混合云场景题常要**几层同时答对**：「本地私有访问 S3」= DX + S3 Interface Endpoint + Resolver Inbound

---

## 1. Transit Gateway 进阶

### ⭐ Association vs Propagation

| 动作 | 含义 | 一句话 |
|---|---|---|
| **Association** | 挂载点发出的流量**查哪张**表 | 我能去哪 |
| **Propagation** | 挂载点的 CIDR **写进哪张**表 | 谁能找到我 |

- 每个挂载点**只能关联一张**表；可以**传播到多张**表
- A → B 要通：A 关联的表里有 B 的 CIDR，**且** B 关联的表里有 A 的 CIDR（回程）

### ⭐ 隔离 + 共享服务

```
Prod-RT   ← Prod 关联   内容：Shared（无 Dev）
Dev-RT    ← Dev 关联    内容：Shared（无 Prod）
Shared-RT ← Shared 关联 内容：Prod + Dev

Prod 只传播到 Shared-RT；Shared 传播到 Prod-RT 和 Dev-RT
```

> 🎯 「多环境互相隔离，但都访问共享服务」→ **多张 TGW 路由表**
> ⚠️ 安全组 / NACL 能做但不是最佳答案，SAP 要**路由层**隔离

### ⭐ 集中出口（Centralized Egress）

```
各 VPC：0.0.0.0/0 → TGW
TGW：0.0.0.0/0 → Egress VPC（NAT GW + IGW）
```

- 省掉 N 个 VPC 的 NAT GW + 集中管控
- Egress VPC 的 NAT GW 仍要**每个 AZ 一个**
- ⚠️ 代价：**TGW 数据处理费**；流量特别大时可能比省下的还多

### ⭐ 集中检查 + Appliance Mode

Inspection VPC 放 **Network Firewall** 或第三方防火墙（经 **Gateway Load Balancer**）。

🚨 TGW 默认按 AZ 就近转发 → 去程、回程可能走不同 AZ 的防火墙 → 有状态防火墙只看到半个连接 → 丢包
→ 在 **Inspection VPC 的 TGW 挂载点**开 **Appliance Mode**（对称路由）

> 🎯 「经检查 VPC 后**间歇性丢包**」「有状态」「非对称路由」→ Appliance Mode

### 其他

| 考点 | 内容 |
|---|---|
| 挂载子网 | 每个 AZ 一个专用 /28 子网放 TGW 挂载点 |
| **TGW Peering** | 跨区域，🚨 **只支持静态路由** |
| Blackhole 路由 | 丢弃去往某 CIDR 的流量 |
| **TGW Connect** | GRE + BGP 接 **SD-WAN** |
| **Cloud WAN** | 多区域、**一份全局策略**管网络分段 |
| 跨账户 | **RAM** 共享 TGW |

---

## 2. Direct Connect 进阶

### 连接方式

| | Dedicated | Hosted |
|---|---|---|
| 来源 | AWS 直接分配物理端口 | **合作伙伴**分带宽 |
| 带宽 | **1 / 10 / 100 / 400 Gbps** | 50 Mbps – 25 Gbps |
| 适用 | 大流量、LAG、MACsec | 小带宽、快开通 |

### LAG

同一站点、相同带宽的多条专用连接用 LACP 捆绑。

- 作用：**加带宽** + 单条链路故障自动切换
- 「最少活动链路数」：低于该数整个 LAG 下线，让流量切到备用路径
- 🚨 **所有连接接在同一个站点的同一台 AWS 设备上** → **只防单条线路 / 端口故障，防不了设备故障和站点故障**

> ⚠️ LAG 是高可用题最常见的陷阱：看起来在加冗余，但全在一台设备上

### ⭐ 三档弹性

| 模型 | 设计 | SLA | 防什么 |
|---|---|---|---|
| **Maximum Resiliency** | 2 站点 × 每站点 2 条（不同设备） | **99.99%** | 设备 + 站点 |
| **High Resiliency** | 2 站点 × 每站点 1 条 | 99.9% | 站点 |
| **Development and Test** | 1 站点 × 2 条（不同设备） | — | 设备，不用于关键业务生产负载 |

> 🎯 「**关键业务**」「**最高可用**」→ Maximum
> 🎯 「要冗余但**控制成本**」→ **DX + Site-to-Site VPN 备份**
> 同前缀 DX 和 VPN 都有 → AWS **优先走 DX**

**解法**：先问要防哪一级故障 —— 链路 → LAG；设备 → 同站点不同设备；站点 → 2 个站点；DX 整体 → VPN 备份。

### ⭐ DX 加密（默认不加密）

| 方案 | 层 | 特点 | 识别词 |
|---|---|---|---|
| **MACsec** | 二层 | **线速**加密；仅**专用连接**（10/100/400G）+ 支持的站点 | 「高带宽」+「加密」+「不影响吞吐」 |
| **IPsec VPN over DX** | 三层 | 灵活；**每条隧道约 1.25 Gbps** | 「已有 DX 加一层加密」「带宽不大」 |

- **Private IP VPN**：DX 上用**私有 IP** 建 VPN（Transit VIF → TGW），流量完全不经公网

> 🎯 10 Gbps DX 要加密且不降吞吐 → **MACsec**（VPN 被 1.25 Gbps/隧道卡住）

### DX Gateway

一条 DX → DX Gateway（全球）→ 多个区域的 VGW / TGW。
🚨 **DX Gateway 不做 VPC 之间的转发**，只管「本地 ↔ VPC」。
VPC 之间互通（跨区域）→ 各区域 TGW + **TGW Peering**。

### SiteLink

多个本地站点之间通过 **AWS 骨干网**互通，**不经过任何区域 / VPC**。

> 🎯 「全球**分公司之间**互通」「已有 DX」「不想自建跨国专线」→ SiteLink

---

## 3. Route 53 Resolver（混合云 DNS）

| 方向 | 用什么 | 配置 |
|---|---|---|
| **本地 → AWS** | **Inbound Endpoint** | 本地 DNS 条件转发到入站端点的 IP |
| **AWS → 本地** | **Outbound Endpoint** + **转发规则** | 规则「`corp.local` → 本地 DNS IP」关联到 VPC |

> 🔑 **Inbound = 别人进来问 AWS；Outbound = AWS 出去问别人**
> 🚨 前提：已有 **DX 或 VPN**（Resolver 不提供网络连通）；端点至少 **2 个 AZ**

### ⭐ 多账户集中 DNS

```
网络账户 Shared Services VPC
  ├─ 只建一套 Inbound + Outbound Endpoint
  └─ 转发规则 ──RAM 共享──→ 所有业务账户的 VPC 关联
```

- 私有托管区域**跨账户关联 VPC**：托管区域账户先授权 → VPC 账户发起关联（只能 CLI / API）
- 🆕 **Route 53 Profiles**：把 PHZ 关联 + 转发规则 + DNS Firewall **打包**，RAM 共享给大量 VPC

> 🎯 「**多账户** + 混合 DNS + **最少运维**」→ 集中端点 + RAM 共享规则 / **Route 53 Profiles**

### DNS Firewall

在 VPC 的 DNS 解析层拦截域名。
> 🎯 「阻止解析**恶意域名**」「防 **DNS 外传数据**」→ Resolver DNS Firewall

---

## 4. PrivateLink 进阶 + VPC Lattice

### SaaS 提供方架构

```
提供方：应用 ← NLB ← Endpoint Service（接受请求 / 白名单）
消费方：Interface Endpoint（几百个客户）
```

- **单向**：消费方 → 提供方；提供方不能访问消费方网络
- 不需要 Peering / TGW，**CIDR 冲突无所谓**，可扩展到成千上万消费方
- 🆕 **跨区域 PrivateLink**：Interface Endpoint 可直接连其他区域的 Endpoint Service

> 🎯 「私有地提供给**很多**客户」「客户 **CIDR 可能重叠**」「只暴露**一个服务**」→ PrivateLink

### ⭐ 本地通过 DX 私有访问 S3

| 方案 | 结果 | 原因 |
|---|---|---|
| S3 **Gateway** Endpoint | ❌ | 是**路由表里的一条路由**，只对 VPC 内部流量生效 |
| S3 **Interface** Endpoint | ✅ | 是**有私有 IP 的网卡**，本地经 DX 能直接访问 |

DNS：配 **Resolver Inbound Endpoint**，本地才能解析到 Interface Endpoint 的私有 IP。

> 🔑 判断「X 能不能从 Y 访问」→ 先想清楚这个服务的**物理形态**

### VPC Lattice

**服务与服务之间的应用层网络**：跨 VPC / 账户连微服务，自带 **IAM 授权策略**，CIDR 重叠无所谓。

| | TGW | PrivateLink | VPC Lattice |
|---|---|---|---|
| 连什么 | **整个网络**（三层） | **单个服务**（单向） | **很多服务之间**（应用层） |
| CIDR 重叠 | ❌ | ✅ | ✅ |
| 访问控制 | 路由表 + SG | 白名单 + SG | **IAM 授权策略** |
| 场景 | 全通、集中出口 / 检查 | SaaS 对外 | 跨账户**微服务**互调 |

> 🎯 「多账户**微服务**互调」「**不想管**底层网络」「按 **IAM** 控服务间访问」→ VPC Lattice

---

## 5. 我的失分点（W6）

| # | 错误 | 正确理解 |
|---|---|---|
| 1 | RAM 共享的子网路由表以为归**管理账户 / 委托管理员**管 | 归 **VPC 所有者（网络账户）**；RAM 共享**所有权不变** |
| 2 | （讲解更正）曾说 LAG 能防设备故障 | LAG 全部连接在**同一台设备** → 只防单链路故障 |
| 3 | DX 加密理由只说「硬件 vs 软件」 | 关键是数字：VPN **1.25 Gbps / 隧道** |
| 4 | W5 遗留：权限边界「拆锁」路径错 3 次，第 4 次答对 | `iam:DeleteRolePermissionsBoundary` 对新建角色调用 → 边界消失 → Admin。另有 `iam:PassRole` 路径：限制只能 pass `app-*` 且带边界的角色 |

---

## 6. 自测题（不写答案）

1. Association 和 Propagation 分别决定什么？A 访问 B 需要哪两个条件？
2. Prod / Dev 隔离、都能访问 Shared：几张表？Prod 传播到哪？
3. 有状态防火墙集中检查后间歇性丢包，原因和修法？
4. 集中出口省了什么、多了什么？Egress VPC 的 NAT GW 要几个？
5. TGW Peering 能用动态路由传播吗？
6. LAG 能防哪一级故障？为什么？
7. 关键业务 DX 最高可用怎么设计？只想控制成本又要冗余呢？
8. 10 Gbps DX 要加密且不降吞吐，选什么？为什么不选 VPN？
9. DX Gateway 下挂的两个 VPC 能通过它互通吗？要互通用什么？
10. 全球分公司之间想走 AWS 骨干网互通，用什么？
11. 本地解析 AWS 私有域名、AWS 解析本地域名，各用什么端点？前提？
12. 50 个账户都要混合 DNS，怎么最省运维？
13. 本地经 DX 访问 S3，为什么 Gateway Endpoint 不行？DNS 怎么办？
14. SaaS 给 300 个 CIDR 冲突的客户私有提供 API，用什么？
15. 跨账户微服务互调、按 IAM 控制访问，用什么？
