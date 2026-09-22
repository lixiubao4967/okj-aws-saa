# Week 4：存储服务

## 1. S3 存储类型

| 类型 | 可用区 | 最低存储时间 | 适用场景 |
|------|--------|-------------|---------|
| Standard | ≥3 AZ | 无 | 频繁访问，通用 |
| Standard-IA | ≥3 AZ | 30 天 | 低频访问，需快速取回 |
| One Zone-IA | 1 AZ | 30 天 | 低频 + 可重建的数据（非合规场景） |
| Glacier Instant | ≥3 AZ | 90 天 | 归档，毫秒级取回 |
| Glacier Flexible | ≥3 AZ | 90 天 | 归档，分钟~小时取回 |
| Glacier Deep Archive | ≥3 AZ | 180 天 | 最便宜，12~48 小时取回 |
| Intelligent-Tiering | ≥3 AZ | 无 | 访问模式不确定时自动分层 |

**考试要点：**
- 合规/审计场景不能用 One Zone-IA（单 AZ 有数据丢失风险），应选 Standard-IA
- Intelligent-Tiering 有月度监控费用，但无取回费用

> 🔑 **IA ≠ Archive —— 题干用词直接决定选哪个（高频陷阱）**
>
> | 题干英文 | 定位 | 选 |
> |---|---|---|
> | **`infrequent access`** 不常访问但**随时要秒读** | 少读，快取 | **Standard-IA / One Zone-IA** |
> | **`archive` / `archival` / `long-term retention`** | 基本不读，**取回可以等** | **Glacier 系列** |
>
> 看到 **archive / archival / long-term retention** → **一律 Glacier**，IA 是干扰项。
> 再细分：毫秒取回 → Glacier Instant；小时级可接受 → Glacier Flexible；
> 最低成本/极少取 → Deep Archive。

### 生命周期「瀑布模型」（转换只能往下走）

```
Standard → Intelligent-Tiering → Standard-IA → One Zone-IA
        → Glacier Instant → Glacier Flexible → Deep Archive
```

- ❌ **不能用生命周期策略反向升级**（Glacier → Standard 做不到），要恢复必须先 Restore 再复制为新对象
- ⚠️ 转 Standard-IA / One Zone-IA 要求对象**至少存在 30 天**
- ⚠️ **小于 128 KB** 的对象转 IA 不划算，AWS 不会自动转

## 2. 生命周期策略

- 自动在存储类型之间转换对象（如 30 天后转 IA，90 天后转 Glacier）
- 可按前缀或标签过滤
- 支持自动删除过期对象

## 3. 跨区域复制（CRR）与同区域复制（SRR）

| | CRR | SRR |
|--|-----|-----|
| 用途 | 灾备、合规、低延迟跨区访问 | 日志聚合、同区域不同账户间复制 |
| 前提 | 源和目标桶都要开启版本控制 | 同上 |

---

## 4. S3 访问控制

### Bucket Policy vs ACL

| 维度 | Bucket Policy | ACL |
|------|--------------|-----|
| 格式 | JSON，功能强大 | XML，功能有限 |
| 粒度 | 细粒度（可按 IP、VPC、加密方式等条件控制） | 粗粒度（READ/WRITE/FULL_CONTROL） |
| 推荐 | ✅ 首选 | ❌ 仅遗留场景 |

### 访问判定逻辑

IAM Policy（用户侧）+ Bucket Policy（资源侧）+ ACL 综合评估：
- 有任何 Deny → 拒绝
- 没有 Deny，有 Allow → 允许
- 都没有 → 默认拒绝

**跨账户访问：资源方 + 请求方双方都要 Allow**
**同账户访问：IAM Policy 或 Bucket Policy 任一方 Allow 即可**

## 5. Pre-signed URL（预签名 URL）

- 拥有权限的 IAM 用户/角色生成带签名的 URL
- 拿到 URL 的人无需 AWS 凭证即可访问
- 有效期：默认 1 小时，最长 7 天（IAM 用户）/ 12 小时（IAM 角色）
- 支持 GET（下载）和 PUT（上传）
- 权限 = 生成者的权限，生成者权限被撤销则 URL 立即失效

**典型场景：** 付费内容下载、临时分享私有文件、允许未认证用户上传

## 6. 服务端加密（SSE）

| 方式 | 密钥管理 | 特点 |
|------|---------|------|
| SSE-S3 | AWS 全权管理 | 默认加密，零配置，AES-256 |
| SSE-KMS | AWS KMS 管理 | 可审计密钥使用（CloudTrail），受 KMS 配额限制 |
| SSE-C | 客户自己管理 | 每次请求提供密钥，必须用 HTTPS |

**选型决策：**
- 无特殊要求 → SSE-S3
- 需要审计密钥 / 控制密钥权限 → SSE-KMS
- 必须自己管密钥 → SSE-C

**SSE-KMS 注意事项：**
- 高吞吐场景可能触及 KMS API 配额限制（ThrottlingException）
- 解决方案：申请提高 KMS 配额，或启用 S3 Bucket Keys（减少最多 99% KMS 调用）

**强制加密：** 用 Bucket Policy Deny 拒绝没带加密头的请求（默认加密 ≠ 强制加密）

## 7. S3 Block Public Access

- 账户/桶级别的安全开关，一键阻止所有公开访问
- 四个开关：BlockPublicAcls / IgnorePublicAcls / BlockPublicPolicy / RestrictPublicBuckets
- 新建桶默认全部开启
- **账户级别启用可覆盖所有桶** — 防止意外公开的最佳方案

## 7.5 ⭐ CORS（跨源资源共享）—— 浏览器机制，不是 AWS 机制

> TD Test 1 Q20 错题补充。**触发词：browser + blocked + JavaScript。**

**问题**：浏览器的**同源策略**规定 JS 只能向"同源"发请求。
**同源 = 协议 + 域名 + 端口 三者完全相同。**

⚠️ **同一个 bucket 也可能算跨源**，因为 S3 有两种端点域名：

| 端点 | 域名 |
|---|---|
| **静态网站托管端点** | `<bucket>.s3-website-<region>.amazonaws.com` |
| **REST API 端点** | `<bucket>.s3.amazonaws.com` |

网页从网站端点加载、JS 去调 API 端点 → **域名不同 → 被浏览器拦截**。

**解法**：在 bucket 上配置 CORS 规则，声明允许哪些来源。

```json
[{
  "AllowedOrigins": ["http://mysite.s3-website-us-east-1.amazonaws.com"],
  "AllowedMethods": ["GET"],
  "AllowedHeaders": ["*"],
  "MaxAgeSeconds": 3000
}]
```

**CORS 在 AWS 里出现的位置**：S3、**API Gateway**（前端调 API 报 CORS 错 → 在资源上启用）、
CloudFront（需转发 `Origin` 头并加入缓存键）、AppSync。

🔴 **别和这三个混**（它们都以 `Cross-` 开头但毫无关系）：

| | 领域 | 作用 |
|---|---|---|
| **CORS** Cross-**Origin** | 浏览器安全 | 允许跨域名的 JS 请求 |
| **CRR** Cross-**Region** Replication | S3 复制 | 复制对象到另一 Region |
| **Cross-account access** | IAM | 跨 AWS 账号授权 |
| **Cross-Zone Load Balancing** | ELB | 流量分到所有 AZ |

---

## 8. S3 Access Points

- 为桶创建多个独立入口，每个入口有独立的 DNS 名称和访问策略
- 解决多团队访问同一桶时 Bucket Policy 过于复杂的问题
- 可限制只允许从特定 VPC 访问（网络隔离）
- 桶 Policy 可设为"只允许通过 Access Point 访问"，防止绕过

**选型：** 2-3 个简单规则用 Bucket Policy；团队多/策略复杂用 Access Points

---

## 9. EFS vs EBS vs S3 对比

| 特性 | EBS | EFS | S3 |
|------|-----|-----|-----|
| 类型 | 块存储 (Block) | 文件存储 (File) | 对象存储 (Object) |
| 访问方式 | 挂载到单个 EC2（同 AZ） | 挂载到多个 EC2（跨 AZ） | HTTP API |
| 协议 | 设备级别 | NFS v4.1 | REST API |
| 容量 | 创建时指定，手动扩容 | 自动伸缩，按用量付费 | 无限 |
| 性能 | 最高（io2 可达 64,000 IOPS） | 中等 | 取决于请求模式 |
| 持久性 | 单 AZ（快照可跨 AZ） | 跨多 AZ 冗余 | 11 个 9 |
| 典型场景 | 数据库、OS 启动盘 | 共享文件系统、CMS | 静态资源、备份、数据湖 |

**考试关键词映射：**
- "shared storage" / 多实例共享 → **EFS**
- "database" / "boot volume" / 高 IOPS → **EBS**
- "static assets" / "backup" / "data lake" → **S3**
- "Windows 文件共享" → **FSx for Windows**（EFS 只支持 Linux/NFS）

**易混点：**
- EBS Multi-Attach（io1/io2）只支持同一 AZ 内最多 16 个实例，跨 AZ 共享一定选 EFS
- EFS 也有存储分层：Standard / Standard-IA / One Zone / One Zone-IA，可配生命周期策略
- S3 不能被挂载为文件系统，不能替代 EFS 的共享文件场景

> 🔴 **S3 生命周期 ≠ EFS 生命周期（高频陷阱）**
>
> | | **S3 Lifecycle** | **EFS Lifecycle Management** |
> |---|---|---|
> | Transition 转存储层 | ✅ Standard → IA → Glacier… | ✅ Standard ↔ IA ↔ Archive |
> | **Expiration 删除文件** | ⭐ **✅ 能删** | ❌ **不能删，只能转层** |
> | **转换周期上限** | ⭐ **无上限**（任意天数） | ⚠️ **最长 365 天**（最短 1 天） |
> | 其他 | 删旧版本、中止未完成分段上传 | 无 |
>
> **两条排除 EFS 的硬判据：**
> - 题干要求「N 天后自动**删除**」→ 必须 S3
> - 题干要求转换周期 **超过 1 年**（如"2 年后转冷存储"= 730 天）→ 必须 S3

### 各服务的「自动删除」机制（别把 S3 的概念套到别处）

| 服务 | 机制 | 能删 |
|---|---|---|
| **S3** | Lifecycle rule → **Expiration action** | ✅ |
| **EFS** | 无删除功能 | ❌ |
| **DynamoDB** | **TTL**（按时间戳属性自动删项） | ✅ |
| **CloudWatch Logs** | **Retention setting**（保留天数） | ✅ |
| **EBS 快照** | **DLM**（Data Lifecycle Manager） | ✅ |
| **ECR 镜像** | **ECR Lifecycle Policy** | ✅ |
| **RDS 备份** | **Backup retention period**（1–35 天） | ✅ |
| **Kinesis Data Streams** | **Retention period**（默认 24h，最长 365 天） | ✅ |

---

## 9.5 AWS Transfer Family

| 项目 | 内容 |
|------|------|
| **协议** | **SFTP / FTPS / FTP / AS2** |
| **后端存储** | **S3 或 EFS**（两者都支持） |
| **认证** | 服务托管 / AD / 自定义 IdP（Lambda + API Gateway） |
| **运维** | ⭐ 全托管，**无需管理 SFTP 服务器**；跨多 AZ 高可用 |
| ⚠️ **没有的功能** | **没有 retention policy** —— 数据保留/删除要在**后端存储**上配 |

**关键词**：`SFTP / FTPS / FTP` + `不想管服务器` → **AWS Transfer Family**
⚠️ 看到「在 EC2 上装 SFTP 服务 + cron 清理」→ 运维最重，`least operational overhead` 题里一律排除。

---

## 10. Storage Gateway — 混合云存储桥梁

在本地部署网关（VM/硬件），让本地应用透明访问 AWS 云存储。

| 类型 | 协议 | 数据存储位置 | 典型场景 |
|------|------|-------------|---------|
| S3 File Gateway | NFS / SMB | S3 | 本地应用以文件方式读写，实际存到 S3 |
| FSx File Gateway | SMB | FSx for Windows | Windows 文件共享上云，本地缓存加速 |
| Volume Gateway | iSCSI | S3（EBS 快照） | 本地块存储备份到云端 |
| Tape Gateway | iSCSI VTL | S3 Glacier | 替代物理磁带库，归档到 Glacier |

**Volume Gateway 两种模式：**
- **Cached Mode**：主数据在 S3，本地只缓存热数据 → "minimize on-premises storage"
- **Stored Mode**：主数据在本地，异步备份到 S3 → "low-latency access to full dataset"

---

## 11. Snow 系列 — 大规模离线数据迁移

| 设备 | 容量 | 特点 | 典型场景 |
|------|------|------|---------|
| Snowcone | 8TB HDD / 14TB SSD | 最小，可手提 | 边缘采集、小规模迁移 |
| Snowball Edge Storage | 80TB | 存储优化 | 中大规模数据迁移 |
| Snowball Edge Compute | 42TB | 可跑 EC2 和 Lambda | 边缘计算 + 数据迁移 |
| Snowmobile | 100PB（集装箱卡车） | 超大规模 | 数据中心级别迁移 |

**容量选择逻辑：**
- < 10TB → 走网络（Direct Connect / VPN / DataSync）
- 10TB ~ 数十PB → Snowball Edge（可订多台并行）
- \> 10PB 且时间紧 → Snowmobile

**迁移工具速查：**

| 场景 | 方案 |
|------|------|
| 本地 NFS/SMB 文件上云 | S3 File Gateway |
| Windows 文件共享 | FSx File Gateway |
| 本地块存储备份 | Volume Gateway |
| 替代磁带备份 | Tape Gateway |
| 大量数据物理搬运 | Snowball Edge |
| 无网络边缘计算 | Snowball Edge Compute |
| 持续在线文件同步 | DataSync |
