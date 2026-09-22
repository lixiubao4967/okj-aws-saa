# SAA-C03 同名词辨析速查（Gateway / Endpoint / Policy）

> **核心认知：AWS 大量复用通用词做服务名。**
> `Gateway`（门）、`Endpoint`（端点）、`Policy`（策略）**本身不表示任何技术含义**。
> 👉 **永远看修饰词，别看主词。** 主词一样、修饰词不同 = 完全不同的东西。
>
> ⚠️ 这也是 AWS 出干扰项的标准手法：把正确服务的"外壳"保留、把"内容"换掉
> （例：`Define an **Shield Advanced** policy in **AWS Firewall Manager**` —— 外壳对、内容错）。

---

# 一、Gateway 全家族

## 🌐 A. VPC 网络连通类（考得最多）

| Gateway | 解决什么 | 方向 | 题干关键词 |
|---|---|---|---|
| **Internet Gateway (IGW)** | VPC 连互联网 | **双向** | 公有子网必需、`0.0.0.0/0 → igw` |
| **NAT Gateway** | 私有子网**出站**上网（打补丁、调外部 API） | **只出不进** | 私有子网、💰 **按小时 + 按 GB** |
| **Egress-Only Internet Gateway** | 同 NAT，但**专用 IPv6** | 只出不进 | **IPv6** + 只出站 |
| **Virtual Private Gateway (VGW)** | VPN 的 **AWS 侧**端点 | — | Site-to-Site VPN |
| **Customer Gateway (CGW)** | VPN 的 **本地侧**端点 | — | 你机房的路由器 |
| **Transit Gateway (TGW)** | 多 VPC + 本地网络的**中心路由器** | — | `大量 VPC 互联`、`简化拓扑`、中心辐射 |
| **Direct Connect Gateway** | 一条 DX 专线连**多 Region 多 VPC** | — | DX + 跨 Region |

## 🔌 B. VPC Endpoint 类

| 类型 | 支持服务 | 费用 | 实现 |
|---|---|---|---|
| **Gateway Endpoint** | ⚠️ **只有 S3 和 DynamoDB** | ⭐ **完全免费** | 加一条**路由表**条目 |
| **Interface Endpoint**（PrivateLink） | **几乎所有** AWS 服务 + 第三方 | 💰 按小时 + 按 GB | 子网里创建 **ENI**（私有 IP） |

## ⚖️ C. 负载均衡类

| | ALB | NLB | **GWLB** |
|---|---|---|---|
| 层级 | L7（HTTP） | L4（TCP/UDP） | **L3（IP 包）** |
| 用途 | Web 路由 | 超低延迟、静态 IP | ⭐ **部署第三方虚拟网络设备** |
| 关键词 | 路径/host 路由 | 百万 TPS、固定 IP | **防火墙 / IDS / IPS / 深度包检测** |

## 💾 D. Storage Gateway（混合云存储，**与网络无关**）

装在**你本地机房**的存储代理，不是网络设备。

| 子类型 | 协议 | 云端落到哪 | 决定性关键词 |
|---|---|---|---|
| **S3 File Gateway** | **NFS / SMB** | S3 对象 | 文件协议访问 S3、**本地缓存** |
| **FSx File Gateway** | **SMB** | FSx for Windows | Windows **AD 集成 / DFS** |
| **Volume Gateway** | **iSCSI**（块） | S3（EBS 快照形式） | 本地**块存储**备份 |
| **Tape Gateway** | **iSCSI VTL** | S3 Glacier | **替代物理磁带库**（Veeam/NetBackup） |

**Volume Gateway 两模式**：
- **Cached**：主数据在 S3，本地只缓存热数据 → `minimize on-premises storage`
- **Stored**：主数据在本地，异步备份到 S3 → `low-latency access to entire dataset`

## 🚪 E. 其他

| Gateway | 用途 |
|---|---|
| **Amazon API Gateway** | 托管 **API 前端**（REST / HTTP / WebSocket） |
| **Local Gateway (LGW)** | **Outposts** 连本地网络专用 |
| **Carrier Gateway** | **Wavelength**（5G 边缘）连运营商网络专用 |

---

# 二、Gateway 最易混的四组

### ① IGW vs NAT Gateway —— 看**方向**
```
IGW：  互联网 ⇄ VPC         双向，公有子网用
NAT：  私有子网 → 互联网     只出不进，私有子网用
```
> NAT 的意义就是「里面的能出去，外面的进不来」。

### ② VGW vs CGW —— 看**哪一侧**
```
[本地机房] ─── CGW ═══ VPN 隧道 ═══ VGW ─── [AWS VPC]
           你的路由器              AWS 的端点
```
> **C**ustomer = **你**的；**V**irtual **P**rivate = AWS 虚拟出来的。

### ③ Transit Gateway vs VPC Peering
| | VPC Peering | **Transit Gateway** |
|---|---|---|
| 拓扑 | 点对点，N 个 VPC 要 **N(N-1)/2** 条 | 中心辐射，N 个 VPC 只要 **N** 条 |
| **传递路由** | ❌ **不支持**（A-B、B-C ≠ A 能到 C） | ✅ 支持 |
| 适用 | 少量 VPC（2-3 个） | **大量 VPC / 也要连本地** |
| 费用 | 只付跨 AZ/Region 流量 | 按小时 + 按 GB |

### ④ Storage Gateway vs DataSync
| | **DataSync** | **Storage Gateway** |
|---|---|---|
| 定位 | **迁移/同步工具**（跑任务） | **持续的混合存储访问层** |
| **本地缓存** | ❌ **没有** | ✅ **有** |
| 本地应用文件协议持续读写 | ❌ | ✅ |
| 题干 | `migrate`、`one-time`、`bulk`、`periodic sync` | `hybrid`、**`local cache`**、`low-latency`、`SMB/NFS/iSCSI` |

---

# 三、一眼分辨：看修饰词属于哪个领域

```
Internet / NAT / Transit / Virtual Private / Customer / Direct Connect
      → 【网络连通】

Endpoint
      → 【私有访问 AWS 服务】

Load Balancer
      → 【流量分发】

Storage / File / Volume / Tape
      → 【混合云存储】

API
      → 【应用接口】
```

---

# 四、Endpoint 家族（第二大混淆源）

| Endpoint | 是什么 |
|---|---|
| **Gateway VPC Endpoint** | 路由表条目，私有访问 **S3 / DynamoDB**，**免费** |
| **Interface VPC Endpoint** | 子网里的 ENI，私有访问几乎所有服务，**收费** |
| **VPC Endpoint Service** | 你**自己发布**服务给别人用 PrivateLink 访问 |
| **RDS Endpoint** | 数据库连接地址（Multi-AZ 故障切换后**不变**） |
| **Aurora Cluster Endpoint** | 指向**写入实例** |
| **Aurora Reader Endpoint** | ⭐ 自动在**所有只读副本间负载均衡** |
| **Aurora Custom Endpoint** | 指定一组实例（如给报表专用） |
| **API Gateway - Edge-optimized** | 默认，走 CloudFront 边缘，**全球用户** |
| **API Gateway - Regional** | 同 Region 客户端，或自己搭 CDN |
| **API Gateway - Private** | ⭐ **只能从 VPC 内访问**（配 Interface Endpoint） |

---

# 五、Policy 家族（第三大混淆源）

## 🔐 IAM 相关

| Policy | 作用 |
|---|---|
| **Identity-based policy** | 挂在**用户/组/角色**上，"这个身份能做什么" |
| **Resource-based policy** | 挂在**资源**上（S3 bucket policy、KMS key policy），"谁能动我" |
| **SCP**（Service Control Policy） | **Organizations** 层面的**权限上限**，不授权只设边界 |
| **Permissions boundary** | 给**单个 IAM 实体**设权限上限 |
| **Session policy** | AssumeRole 时临时收窄权限 |

> 🔑 **最终权限 = 身份策略 ∩ 资源策略 ∩ SCP ∩ 权限边界**，任一处 Deny 即拒绝。

## 📦 资源相关

| Policy | 作用 |
|---|---|
| **S3 Bucket Policy** | 资源策略，控制谁能访问 bucket |
| **KMS Key Policy** | ⭐ **KMS 的"第二把锁"**——光有 IAM 权限不够，密钥策略也要允许 |
| **S3 Lifecycle Policy** | 对象自动转存储类 / 过期删除 |
| **ASG Scaling Policy** | 目标追踪 / 步进 / 简单 / 计划 |
| **CloudFront Origin Protocol Policy** | ⚠️ CloudFront **用什么协议回源**（HTTP Only / HTTPS Only / **Match Viewer**）——**与访问控制无关** |
| **CloudFront Viewer Protocol Policy** | 用户用什么协议访问（可强制 HTTPS） |
| **Route 53 Routing Policy** | Simple / Weighted / Latency / Failover / Geolocation / Geoproximity / Multivalue |

---

# 五点五、`Cross-` 家族（四个完全无关的东西）

> **实战踩坑**：TD Test 1 Q20 四个选项全部以 `Cross-` 开头，分属四个领域。

| 名称 | 领域 | 作用 | 触发词 |
|---|---|---|---|
| ⭐ **CORS**<br>Cross-**Origin** Resource Sharing | **浏览器安全** | 允许网页 JS 访问**不同域名**的资源 | ⭐ **browser** + **blocked** + **JavaScript**<br>`No 'Access-Control-Allow-Origin' header` |
| **CRR**<br>Cross-**Region** Replication | **S3 复制** | 对象自动复制到**另一个 Region** 的 bucket | 灾备、合规、跨区低延迟访问 |
| **Cross-account access** | **IAM** | **跨 AWS 账号**授权（用 Role + 资源策略） | 另一个账号要访问我的资源 |
| **Cross-Zone Load Balancing** | **ELB** | 把流量分到**所有 AZ** 的目标 | ELB 各 AZ 负载不均 |

> 🔑 **`Cross-` 只是"跨"，跨什么才是关键。**
> 三词同现 **browser / blocked / JavaScript** → **必定 CORS**。

**CORS 在 AWS 里出现的位置**：S3（bucket CORS 配置）、**API Gateway**（资源上启用 CORS）、
CloudFront（需转发 `Origin` 头并加入缓存键）、AppSync。

**S3 的两种端点域名不同 → 同一个 bucket 也算跨源**：
- 静态网站端点：`<bucket>.s3-website-<region>.amazonaws.com`
- REST API 端点：`<bucket>.s3.amazonaws.com`

---

# 六、其他被复用的主词

| 主词 | 完全不同的东西 |
|---|---|
| **Replication** | S3 **CRR/SRR** ｜ RDS **Read Replica** ｜ Aurora **Replica** ｜ DynamoDB **Global Tables** |
| **Access** | S3 **Access Point** ｜ CloudFront **OAC/OAI** ｜ IAM **Access Analyzer** ｜ **Access Key** |
| **Snapshot** | EBS 快照 ｜ RDS 快照 ｜ Redshift 快照 ｜ FSx 备份（互不相通） |
| **Cache** | CloudFront 边缘缓存 ｜ ElastiCache ｜ **DAX** ｜ Global Accelerator（**不缓存**） |
| **Signed** | CloudFront **Signed URL/Cookies**（走 CDN） ｜ S3 **Pre-signed URL**（直连 S3） |

---

# 📌 考场动作

1. 看到 `Gateway` / `Endpoint` / `Policy` → **先读修饰词，确定领域**
2. 选项"外壳正确、内容可疑"时 → **单独验证内容那部分**
3. 两个选项主词相同、修饰词不同 → **题干一定埋了一句话来区分它们**，回去找那句
