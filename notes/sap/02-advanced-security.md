# 高级安全：权限边界 / ABAC / KMS 进阶 / Security Hub 汇总 / 后量子加密

> SAP-C03 W5 · 2026-11-03 – 11-09
> **Domain 2「Security, Compliance, and Governance」占 25%**

---

## 1. 权限边界（Permissions Boundary）

**挂在单个 IAM 用户或角色上的权限上限**，只设上限，不授予权限。

```
最终权限 = 身份策略 ∩ 权限边界
```

### 三种「上限」对比

| | SCP | 权限边界 | Session Policy |
|---|---|---|---|
| 作用范围 | OU / 整个账户 | **单个用户或角色** | 单次 AssumeRole 会话 |
| 谁来设 | 组织管理员 | 账户内 IAM 管理员 | 调 AssumeRole 时传入 |
| 典型用途 | 组织级合规 | ⭐ **安全委派建角色的权限** | 临时缩小某次会话权限 |

> 🎯 题干「只限制**某团队新建的角色**」→ 权限边界；SCP 太粗，会影响整个账户

### ⭐ 防权限提升

开发者要自己建 IAM 角色 → 只允许建**挂了指定边界**的角色：

```json
{
  "Effect": "Allow",
  "Action": ["iam:CreateRole", "iam:AttachRolePolicy", "iam:PutRolePolicy"],
  "Resource": "arn:aws:iam::111122223333:role/app-*",
  "Condition": {
    "StringEquals": {
      "iam:PermissionsBoundary": "arn:aws:iam::111122223333:policy/DevBoundary"
    }
  }
}
```

条件键：**`iam:PermissionsBoundary`**（Permission**s** 有 s；`StringEquals` 是运算符不是键）

### 🚨 还要 Deny 两条绕过路径

| 路径 | 比喻 | 要 Deny 的 API |
|---|---|---|
| ① 改边界策略的内容 | 撬锁 | `iam:CreatePolicyVersion`、`iam:SetDefaultPolicyVersion`、`iam:DeletePolicy` |
| ② **边界策略不动，直接从角色上摘掉 / 换掉** | **拆锁** | **`iam:DeleteRolePermissionsBoundary`**、**`iam:PutRolePermissionsBoundary`** |

> 🔑 API 名里带 **Role + PermissionsBoundary** → 操作的是「角色和边界之间的关系」

> 🎯 识别词：「开发者**自行创建 IAM 角色**」「**不能超过**规定权限」「防止**权限提升**」

### 完整求值顺序

```
1. 任何地方显式 Deny                → ❌
2. SCP / RCP 不允许                → ❌
3. 资源策略允许（同账户）            → ✅ 可能直接放行
4. 身份策略 ∩ 权限边界 ∩ 会话策略     → 三者都允许才放行
5. 都没有允许                       → ❌ 隐式拒绝
```

---

## 2. ABAC（Attribute-Based Access Control）

**用户标签和资源标签一致就允许。**

| | RBAC | ABAC |
|---|---|---|
| 写法 | 策略写死资源 ARN | 只写「标签相等」 |
| 新增项目 / 资源 | **每次改策略** | 打标签即可，**策略不动** |

> 🎯 识别词：「团队 / 项目**快速增长**」「**减少策略数量**」「新资源**不用改权限**」→ ABAC

### 三个条件键

| 条件键 | 指的是 |
|---|---|
| `aws:PrincipalTag/xxx` | **发起请求的人**的标签 |
| `aws:ResourceTag/xxx` | **被访问资源**上已有的标签 |
| `aws:RequestTag/xxx` | **这次请求要新打上**的标签（创建资源时） |

```json
{
  "Effect": "Allow",
  "Action": ["ec2:StartInstances", "ec2:StopInstances"],
  "Resource": "*",
  "Condition": {
    "StringEquals": { "aws:ResourceTag/Project": "${aws:PrincipalTag/Project}" }
  }
}
```

### 🚨 两个陷阱

1. **标签必须防篡改**：限制 `ec2:CreateTags` / `iam:TagUser` 等，否则改标签就能越权
2. **创建时强制带标签**：`aws:RequestTag/Project = ${aws:PrincipalTag/Project}`，否则出现无标签、谁都管不了的资源

### Identity Center + ABAC

```
Entra：张三 department = Finance
  ↓ Identity Center「attributes for access control」
AWS 会话：aws:PrincipalTag/department = Finance
  ↓
ABAC 策略：只能访问 department = Finance 的资源
```

> 🎯 「员工属性已在**企业 IdP** 维护」+「按部门控制访问」→ Identity Center 属性 + ABAC

---

## 3. KMS 进阶

### 3.1 多区域密钥（Multi-Region Keys）

多个区域各一把**密钥材料相同、key ID 相同**的密钥 → 在 A 区加密，B 区本地直接解密。

- key ID 以 **`mrk-`** 开头
- 副本**手动创建**；每个副本是**独立资源**（自己的 key policy / grant / 别名）
- 轮换由主密钥统一进行，副本同步
- 🚨 **单区域密钥不能转成多区域**（只能新建，再重新加密数据）
- 🚨 **AWS 托管密钥不能是多区域密钥**

> 🎯 识别词：「**客户端加密**数据要多区域解密」「DR 区直接解密」「DynamoDB Global Tables + 客户端加密」「跨区域**签名验证**」
> ⚠️ 服务端加密 + 服务自己做跨区域复制 → 服务会用目标区域密钥重新加密，**不需要**多区域密钥

### 3.2 跨账户使用 KMS

```
账户 A（所有者）Key Policy：Principal = 账户 B，Allow kms:Decrypt…   ①
账户 B（使用方）IAM 策略：Allow kms:Decrypt，Resource = A 的密钥 ARN   ②
```

1. **Key Policy 是主门**：它不允许，IAM 写了 Allow 也没用
2. ⭐ **AWS 托管密钥不能跨账户**（它的 key policy 你改不了）

### ⭐ 跨账户共享加密快照 / AMI

```
快照用 aws/ebs 或 aws/rds 加密 → 共享给 B ❌

1. A 新建 CMK，key policy 允许账户 B 使用
2. 复制快照，改用这把 CMK 重新加密
3. 把新快照共享给 B
4. B 的 IAM 策略允许使用 A 的 CMK
5. B 复制一份，用 B 自己的 CMK 再加密（不再依赖 A）
```

> 🎯 题干同时出现「**跨账户**」+「**加密**」→ 先问：用的是不是客户托管密钥？

### 3.3 其他

| 功能 | 用途 | 识别词 |
|---|---|---|
| **`kms:ViaService`** | 密钥只能经由指定服务使用 | 「不允许直接调 KMS 解密」 |
| **Grants** | 临时、可编程地委派使用权 | 「临时授权」「不想改 key policy」 |
| **导入密钥材料（BYOK）** | 自己生成后导入，可设过期，可**立即删除** | 「密钥必须自己生成」「要能立即销毁」 |
| **CloudHSM 自定义密钥存储** | 密钥在**独占** HSM（单租户） | 「单租户 HSM」「FIPS 140 Level 3」 |
| **外部密钥存储（XKS）** | 密钥材料**永不进入 AWS** | 「法规要求密钥**必须留在本地**」 |

---

## 4. Security Hub：多账户 + 多区域

> 2025 年推出新版 Security Hub，原功能改名 **Security Hub CSPM**；多账户管理思路相同。

### 多账户：委托管理员

```
管理账户 → 指定 Audit 账户为委托管理员
  ↓
Audit 账户
  ├─ 组织内所有账户自动启用（含以后新加入的）
  ├─ 中央配置：按 OU 统一开标准和控制项
  └─ 看到所有账户的发现
```

🚨 不要用管理账户当委托管理员；用了 Control Tower 一般选 **Audit 账户**。
GuardDuty / Inspector / Macie / Config / Firewall Manager 都是同一套**委托管理员模式**。

### 多区域：跨区域汇总

被链接区域（linked regions）的发现 → 自动复制到**汇总区域**（aggregation region / home region）。

> 🎯 「**一个控制台**看**所有账户 + 所有区域**」→ 委托管理员 **+** 跨区域汇总，两样都要配

### 其他

- 🚨 控制项检查**依赖 AWS Config**：每个账户每个区域都要开，否则没有检查结果
- 标准：AWS Foundational Security Best Practices / CIS / PCI DSS / NIST 800-53
- 发现格式：**ASFF**
- 自动修复：发现 → **EventBridge** → Lambda / **SSM Automation**（W10 细讲）
- **Automation rules**：按条件自动改发现字段（降级 / 抑制），不用写 Lambda

### 相邻服务

| 服务 | 一句话 |
|---|---|
| **GuardDuty** | **发现威胁** |
| **Security Hub** | **汇总**发现 + 合规检查 |
| **Detective** | 发现之后**调查根因** |
| **Security Lake** | 安全日志统一成 **OCSF** 存进自己的 S3，供 SIEM 用 |

---

## 5. 后量子加密（C03 新增）

**威胁**：Harvest now, decrypt later —— 现在录下加密流量，将来用量子计算机解密。

| 密码类型 | 例子 | 受威胁？ |
|---|---|---|
| **非对称**（密钥交换、签名） | RSA / ECDH / ECDSA | 🚨 会被破解 |
| **对称**（数据加密） | AES-256 | ✅ 基本安全 |

→ 静态数据（S3 / EBS 的 AES-256）不是主要风险；**传输中的 TLS 握手**和**数字签名**才是。

| 算法 | 用途 | 替代 |
|---|---|---|
| **ML-KEM**（FIPS 203） | **密钥交换** | ECDH / RSA 密钥交换 |
| **ML-DSA**（FIPS 204） | **数字签名** | RSA / ECDSA 签名 |

### AWS 上的做法

1. **混合后量子 TLS**：ECDH + ML-KEM 同时用；KMS / ACM / Secrets Manager 等端点已支持，新版 SDK 开启即可；负载均衡和 CloudFront 也陆续提供后量子 TLS 策略
2. **KMS 的 ML-DSA 签名密钥**：长期有效、能抗量子的签名（固件、文档）
3. **加密敏捷性**：算法可替换，不写死

> 🎯 「担心**现在截获、以后解密**」→ 传输中数据启用**混合后量子 TLS（ML-KEM）**
> 🎯 「签名要**长期**抗量子」→ KMS **ML-DSA**
> ⚠️ 「把 S3 的 AES-256 换更强算法」= 多余的陷阱选项

---

## 6. 我的失分点（W5）

| # | 错误 | 正确理解 |
|---|---|---|
| 1 | 🚨🚨 **两次**漏掉「把边界从角色上摘掉」，都只答了改 / 删边界策略 | 拆锁路径：**`iam:DeleteRolePermissionsBoundary`** / `PutRolePermissionsBoundary` |
| 2 | 条件键答成运算符 `StringEquals` | 键是 `iam:PermissionsBoundary`，运算符和键要分清 |
| 3 | ABAC 只想到防篡改 | 还要**创建时强制带标签**（`aws:RequestTag`） |
| 4 | 防「现在截获」同时答了 ML-KEM 和 ML-DSA | TLS 密钥交换 → **ML-KEM**；ML-DSA 只管签名 |

---

## 7. 自测题（不写答案）

1. 🚨 让开发者自建角色但不能提权：要检查的条件键？除了改边界策略本身，还要 Deny 哪两个 API？
2. 权限边界、SCP、Session Policy 的作用范围分别是什么？
3. 200 个项目每月新增 10 个，只能访问自己项目的资源，用什么？要防哪两个问题？
4. ABAC 中 `aws:PrincipalTag` / `aws:ResourceTag` / `aws:RequestTag` 各指什么？
5. 员工属性在 Entra ID 里，要按部门控制 AWS 访问，怎么配？
6. 客户端加密数据要在 DR 区域本地解密，用什么？现有密钥能直接改吗？
7. 用 `aws/rds` 加密的快照共享给另一个账户后恢复失败。原因和步骤？
8. 跨账户用 KMS 密钥，key policy 不允许但 IAM 策略允许，能用吗？
9. 法规要求密钥材料不能存放在 AWS，选什么？要单租户 HSM 呢？
10. 80 个账户、4 个区域，一个地方看所有安全发现，新账户自动纳入。怎么配？
11. Security Hub 开了标准但没有检查结果，最可能的原因？
12. 「现在截获、以后解密」优先处理哪一部分？用哪个算法？
