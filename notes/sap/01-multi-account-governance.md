# 多账户治理：Organizations / SCP / RCP / Control Tower / Identity Center / RAM / 跨账户角色

> SAP-C03 W4 · 2026-10-26 – 11-03
> **Domain 2「Security, Compliance, and Governance」占 25%**

---

## 0. 一张图看分工

| 服务 | 管什么 | 一句话 |
|---|---|---|
| **Organizations** | 结构 | Root → OU → 账户，统一账单 |
| **SCP** | 身份侧上限 | 限制**我的人**能做什么 |
| **RCP** | 资源侧上限 | 限制**我的资源**能被谁访问 |
| **Control Tower** | 编排 | 按最佳实践一键搭多账户环境 + 持续合规 |
| **IAM Identity Center** | 人 → 账户 | 员工一套账号登录所有账户 |
| **RAM** | 资源 → 账户 | 子网 / TGW 等资源跨账户共享 |
| **跨账户角色** | 程序 → 账户 | `sts:AssumeRole` 临时凭证 |

> 🎯 SAP 多账户题常把 2–3 个组合考，错误选项多是**某一块用错服务**（RAM 共享 S3、用跨账户角色给员工日常登录…）。

---

## 1. SCP（Service Control Policy）

### 求值逻辑

```
最终权限 = IAM 身份策略 ∩ SCP 上限（每一层 OU 都要 Allow）
```

- **只设上限，不授予权限**：IAM 有 AdministratorAccess、SCP 只 Allow `s3:*` → 最终只能用 S3
- 🚨 **管理账户不受任何 SCP 约束** → 管理账户里不放业务负载
- Service-linked role 不受 SCP 约束

### ⭐ 区域限制的陷阱

```json
{
  "Effect": "Deny",
  "NotAction": ["iam:*", "cloudfront:*", "route53:*", "organizations:*", "sts:*", "support:*"],
  "Resource": "*",
  "Condition": { "StringNotEquals": { "aws:RequestedRegion": ["ap-northeast-1"] } }
}
```

- 全局服务（IAM / CloudFront / Route 53…）的请求落到 **us-east-1**
- 不用 `NotAction` 排除它们 → 全局服务全部被拒

---

## 2. RCP（Resource Control Policy）

### 为什么需要它

🚨 **SCP 只约束本组织内的 principal**。外部账户通过 bucket policy 来访问时，你的 SCP 根本不参与判断 → 要拦外部访问者，只能在资源侧设限。

### 五个特性

1. 和 SCP 一样**只设上限，不授予权限**
2. 默认挂 `RCPFullAWSAccess`，不能摘 → 实际都写 **Deny**
3. **管理账户的资源豁免**
4. **只支持部分服务**：首批 S3 / STS / KMS / SQS / Secrets Manager（后续陆续增加）→ 选项用 RCP 保护不支持的服务 = 错
5. Service-linked role 不受影响

### ⭐ 数据边界 RCP

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "StringNotEqualsIfExists": { "aws:PrincipalOrgID": "o-abc123" },
    "BoolIfExists": { "aws:PrincipalIsAWSService": "false" }
  }
}
```

🚨 **必须排除 AWS 服务主体**：`cloudtrail.amazonaws.com` 等服务主体**没有 OrgID**，只检查 PrincipalOrgID 会把 CloudTrail 写日志、S3 复制、备份一起拦掉。

> 🎯 题干「加了数据边界策略后，**日志 / 复制 / 备份**突然失败」→ 漏了服务主体例外

### 数据边界三个维度

| 条件键 | 边界 | 通常写在 |
|---|---|---|
| `aws:PrincipalOrgID` | **谁来访问**（身份） | RCP / 资源策略 |
| `aws:ResourceOrgID` | **访问谁的资源**（资源） | SCP / VPC 端点策略 |
| `aws:SourceVpc` / `aws:SourceIp` | **从哪里来**（网络） | 资源策略 / 端点策略 |

> 💡 记法：**条件键名写的是对方**。RCP 挂在资源上 → 检查来访者 → **Principal**OrgID；SCP 挂在身份上 → 检查目标 → **Resource**OrgID。

| 需求 | 方案 |
|---|---|
| 组织外的人不能读本组织 S3 | **RCP** + `aws:PrincipalOrgID` |
| 本组织员工不能把数据写到组织外的桶 | **SCP** + `aws:ResourceOrgID` |

---

## 3. AWS Control Tower

**= 把 Organizations、SCP/RCP、Config、CloudTrail、Identity Center、Service Catalog 组装起来的编排层。**

> 🎯 识别词：「**快速**搭建多账户环境」「**按 AWS 最佳实践**」「**最少运维**」→ Control Tower
> 已有自研结构只想加一条限制 → 直接写 SCP

### Landing Zone

```
Root
├── 管理账户          ← Control Tower 装在这里
├── Security OU
│   ├── Log Archive   ← 集中存 CloudTrail + Config 日志
│   └── Audit         ← 安全团队，对所有账户只读 / 审计
└── Sandbox OU        ← 业务账户
```

自动开启：组织级 CloudTrail、各账户 Config、IAM Identity Center。

### ⭐ Controls（旧名 Guardrails）

| 类型 | 底层技术 | 时机 |
|---|---|---|
| **Preventive** | **SCP / RCP** | 调用 API 时直接拒绝 |
| **Detective** | **AWS Config 规则** | 资源建好后发现并报告 |
| **Proactive** | **CloudFormation Hooks** | CFN 部署时检查 |

强制程度：Mandatory / Strongly recommended / Elective。
**Region deny control** 底层也是 SCP（`RequestedRegion` + `NotAction`）。

🚨 **Proactive 只覆盖 CloudFormation 部署**，控制台 / CLI / SDK 直接调 API 拦不住。

| 题干 | 选 |
|---|---|
| **任何 / 所有**方式创建都要拦 | Preventive |
| 通过 **IaC 流水线**部署时检查 | Proactive |
| 找出**已存在**的不合规资源 | Detective |

例：「所有 EBS 卷必须加密 + 找出已有未加密卷」→ **Preventive（SCP Deny `ec2:CreateVolume` when `ec2:Encrypted=false`）+ Detective（Config）**，实务再开 EBS default encryption。

### Account Factory 家族

| 方式 | 场景 |
|---|---|
| **Account Factory**（Service Catalog） | 控制台点选开账户，自带基线 |
| **AFT**（Account Factory for **Terraform**） | Terraform / GitOps 流水线 |
| **AFC**（Account Factory Customization） | 开账户时套蓝图（预置 VPC、角色） |
| **CfCT**（Customizations for Control Tower） | 给已有账户持续推送 SCP / CFN 模板 |

### 其他

- **已有 Organization 不用重建**：Register 现有 OU、Enroll 现有账户
- **Drift**：绕过 Control Tower 直接改它管理的 SCP / OU → 漂移 → Repair / Re-register
- 不要在控制台直接改 Control Tower 生成的 SCP

---

## 4. IAM Identity Center（原 AWS SSO）

### Permission Set → IAM 角色

```
定义 Permission Set「DevReadOnly」→ 分配：开发组 → 账户 A、B、C
  ↓
各账户自动建角色 AWSReservedSSO_DevReadOnly_xxxx
  ↓
用户登录门户选账户 = assume 这个角色
```

Permission Set 是模板，改了自动同步到所有账户。

### 身份源（三选一）

| 身份源 | 场景 |
|---|---|
| 自带目录 | 小公司，无现成身份系统 |
| **Active Directory**（AWS Managed Microsoft AD / **AD Connector**） | 本地 AD |
| **外部 IdP**（Okta / Entra ID）：**SAML 2.0** 登录 + **SCIM** 自动同步 | 已有 Okta / Entra |

> 🎯 判据
> `Entra / Okta + 入离职自动生效` → 外部 IdP + **SCIM**
> `本地 AD 登录，不同步目录到云` → **AD Connector**（只做代理）
> `按部门 / 成本中心属性控权限` → **ABAC**（W5）
> `数百员工 × 多账户 × 集中管理` → Identity Center，排除逐账户建 IAM 用户的选项

- 一个 Organization 一个实例，部署在**一个区域**
- 可指定成员账户做**委托管理员**，日常不用登录管理账户

---

## 5. AWS RAM（Resource Access Manager）

### 常考共享对象

| 资源 | 场景 |
|---|---|
| ⭐ **VPC 子网**（VPC Sharing） | 网络团队集中管 VPC，业务账户把资源建在共享子网 |
| ⭐ **Transit Gateway** | 中心 TGW 共享给所有账户挂载 |
| **Route 53 Resolver 规则** | 统一混合云 DNS 转发（W6） |
| Prefix List / IPAM Pool / License Manager | — |

### ⭐ VPC Sharing 权限边界

| Owner（网络账户） | Participant（业务账户） |
|---|---|
| 拥有 VPC、子网、**路由表**、NACL、IGW、NAT GW | 在共享子网里建 EC2 / RDS / Lambda |
| 看不到也改不了参与者的资源 | 管理自己建的资源 |
| — | 🚨 **不能修改 VPC / 子网 / 路由表** |

好处：VPC 少、CIDR 不冲突、业务账户之间不用 Peering / TGW。

- 开启 **RAM ↔ Organizations 集成** → 组织内共享**免邀请**
- RAM 用于**本身不支持资源策略**的资源（子网、TGW）；S3 用 bucket policy

---

## 6. 跨账户角色

### 两边都要放行

```
账户 A（调用方）                   账户 B（目标）
身份策略：                         RoleX 信任策略：Principal = 账户 A   ②
  Allow sts:AssumeRole  ①         RoleX 权限策略：能做什么
  Resource: B 的 RoleX
```

① ② 缺一不可。

### 三个考点

1. **ExternalId**：第三方 SaaS 厂商访问你的账户 → 信任策略要求 `sts:ExternalId`，防**混淆代理人**（confused deputy）。识别词：「第三方」「供应商」「多个客户共用同一厂商」
2. **扮演角色后原权限暂时失效**：需要同时用两边权限（A 桶读 → B 桶写）→ 用**资源策略**直接授权，调用方保留原权限
3. **Role chaining** 会话最长 **1 小时**

---

## 7. 我的失分点（W4）

| # | 错误 | 正确理解 |
|---|---|---|
| 1 | 以为管理账户也受 SCP 约束 | 管理账户**豁免** SCP / RCP |
| 2 | 「SCP 不能拦外部访问，因为 SCP 是上限」 | 真正原因是**作用范围**：SCP 管不到外部 principal |
| 3 | RCP 讲过一次就忘了 | 补笔记≠记住，开头 10 分钟复习要覆盖它 |
| 4 | 数据边界 RCP 没想到服务主体 | 服务主体**没有 OrgID** → 必须加 `PrincipalIsAWSService` 例外 |
| 5 | 用 Proactive 拦「任何」新建 EBS 卷 | Proactive **只管 CFN 部署**，「任何方式」要 Preventive |

---

## 8. 自测题（不写答案）

1. 一个 OU 挂了「Deny 非 ap-northeast-1」的 SCP 后，IAM 控制台打不开了。原因和修法？
2. 组织外的账户通过 bucket policy 读你的桶。挂一条 Deny 所有 S3 的 SCP 能拦住吗？
3. 安全团队要同时防「外部来读」和「内部往外写」，各用哪种策略、哪个条件键？
4. 数据边界 RCP 上线后 S3 跨区域复制失败。最可能漏了什么？
5. 「所有 RDS 必须加密 + 找出已有未加密实例」需要哪几类 Control Tower 控制项？
6. 团队全部用 Terraform，想用流水线开新账户并带上标准 VPC，选什么？
7. 员工在 Okta，要求离职后 AWS 访问自动失效。Identity Center 怎么配？
8. 业务账户想给共享子网加一条去 TGW 的路由，谁来做？
9. 需要从账户 A 的桶读、写到账户 B 的桶，用跨账户角色有什么问题？更好的做法？
10. 第三方厂商访问你的账户，信任策略除了账户 ID 还要什么？防的是什么问题？
