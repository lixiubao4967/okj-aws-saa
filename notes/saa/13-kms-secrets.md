# KMS + 密钥与凭据管理

> P4 安全 · 2026-09-13（原定 9/15，提前 2 天）
> **Domain 1「Design Secure Architectures」占 30%，全考试权重最大。**

---

## 1. KMS 的三种密钥 —— 先分清「谁管密钥」

| 类型 | 谁创建/管理 | 能否轮换 | 能否跨账户 | 费用 |
|---|---|---|---|---|
| **AWS Owned Key** | AWS 全权拥有，你看不见 | 不可控 | ❌ | 免费 |
| **AWS Managed Key**（`aws/s3` 这类） | AWS 代管，每服务一把 | **每年自动**，不可关 | ❌ | 免费 |
| **Customer Managed Key（CMK）** | 🔑 **你创建、你控制策略** | **可开自动轮换（每年）**，也可手动 | ✅ **可以** | $1/月 + API 调用 |

> 🔑 **判据**：`需要控制密钥策略`、`需要审计谁用了密钥`、`跨账户共享`、`自定义轮换`
> → **Customer Managed Key**
> **只要题干问「更多控制权」，答案就是 CMK。**

### CMK 的三种密钥来源

| 来源 | 适用场景 |
|---|---|
| **AWS 生成**（默认） | 绝大多数场景 |
| **导入自己的密钥材料（BYOK）** | 合规要求「密钥必须由我方生成」 |
| **CloudHSM 自定义密钥库** | 要求**专用硬件 HSM、单租户**、FIPS 140-2 Level 3 |

---

## 2. 🚨 信封加密（Envelope Encryption）—— 必考原理

**KMS 的 `Encrypt` API 有 4 KB 上限。** 那 5 GB 的文件怎么加密？→ 信封加密：

```
① 向 KMS 要一个数据密钥  →  GenerateDataKey
   KMS 返回两份：
     · 明文数据密钥（Plaintext Key）    ← 用来干活
     · 加密后的数据密钥（Encrypted Key）← 存起来

② 用【明文数据密钥】在本地加密那 5GB 数据
   （不经过 KMS，无大小限制）

③ 把【加密后的数据密钥】和密文存在一起，然后
   立即从内存抹掉明文数据密钥

④ 解密时：把加密的数据密钥送给 KMS → Decrypt
         → 拿回明文密钥 → 本地解密
```

> 🔑 **考点信号**：「加密**大文件** / 超过 4KB」→ **信封加密 / GenerateDataKey**
> S3 的 SSE-KMS 底层就是这套机制。

---

## 3. ⭐ 跨账户 / 跨 Region 加密

### 🚨 KMS 特有的「双重授权」

跨账户使用 CMK **需要两边都授权**（高频陷阱）：

```
① 密钥所在账户 A：在【Key Policy】里允许账户 B 使用
② 使用方账户 B：给 IAM 用户/角色附加 kms:Decrypt 等权限

两者缺一不可
```

**为什么 KMS 和别的服务不一样**：大多数服务只看 IAM 策略，
但 KMS 密钥**自带 Key Policy**，且 **Key Policy 是主、IAM 策略是辅**：

```
默认 Key Policy 里有一句「允许本账户的 IAM 策略生效」
  → 删了这句，即使 IAM 给了 kms:* 也照样用不了

跨账户时 Key Policy 必须显式列出对方账户
  → 光在对方账户的 IAM 里加权限不够
```

> 🚨 **排错信号**：「IAM 权限看起来是对的，但访问 KMS 加密的资源报 AccessDenied」
> → **检查 Key Policy**

### 🚨 KMS 密钥是 Region 级别的，不能跨 Region 使用

| 场景 | 做法 |
|---|---|
| **加密的 EBS 快照跨 Region 复制** | 复制时**指定目标 Region 的 KMS 密钥**重新加密 |
| **加密的 AMI 共享给其他账户** | 必须用 **CMK**（AWS Managed Key 不能共享），并在 Key Policy 授权 |
| **S3 跨区复制（CRR）加密对象** | 目标 Region 需要有自己的 KMS 密钥 |
| **多 Region 场景想省事** | 用 **Multi-Region Keys**（同一密钥 ID 在多 Region 有副本） |

---

## 4. ⭐ Secrets Manager vs SSM Parameter Store（必考对比）

| | **Secrets Manager** | **SSM Parameter Store** |
|---|---|---|
| **自动轮换** | 🔑 **✅ 内置，可自动轮换 RDS 密码** | ❌ 无（要自己写 Lambda + EventBridge） |
| 与 RDS/Aurora/Redshift 集成 | ✅ **原生集成** | ❌ |
| 费用 | 🚨 **$0.40/密钥/月**（贵） | **标准层免费**；高级层收费 |
| 加密 | 强制 KMS | 可选（SecureString 用 KMS） |
| 存储上限 | 64 KB | 标准 4 KB / 高级 8 KB |
| 适合放什么 | **数据库密码、API 密钥**等需轮换的凭据 | **配置项、参数**（也能放密码，但不自动轮换） |

> 🔑 **一句话判据**
> `自动轮换` / `rotate` → **Secrets Manager**（独有能力）
> `成本最低` / `只是配置参数` → **Parameter Store**

**Parameter Store 三种类型**：`String`、`StringList`、**`SecureString`**（KMS 加密）

---

## 5. 其他加密 / 凭据高频点

| 场景 | 答案 |
|---|---|
| EC2 访问 S3/DynamoDB | 🚨 **IAM Role**，**绝不要**把 access key 写进代码或 user data |
| 临时跨账户访问 | **STS AssumeRole**（临时凭证） |
| 第三方/外部账户访问你的资源 | AssumeRole + **External ID**（防「混淆代理人」攻击） |
| 给员工临时的 AWS 控制台访问 | **IAM Identity Center**（原 AWS SSO） |
| 移动 App 用户获取 AWS 凭证 | **Cognito Identity Pool** |
| 移动 App 用户注册登录 | **Cognito User Pool** |

---

## 6. 测验（10 题）

> ⚠️ 自测规则：答案不写在本文件里。**选项中的加粗只用于强调题干关键词，绝不标记正确答案。**

**Q1.** 合规部门要求：能够**审计谁在什么时候使用了加密密钥**，并且**自定义密钥策略**。应使用？
A) AWS Owned Key　B) AWS Managed Key　C) Customer Managed Key（CMK）　D) S3 SSE-S3

**Q2.** 应用需要加密一个 **2 GB 的文件**。但直接调用 KMS Encrypt 报错。正确做法？
A) 分成 4KB 小块逐个调 Encrypt　B) 换用 CloudHSM　C) 改用 SSE-S3　D) 用 GenerateDataKey 做信封加密

**Q3.** 账户 A 的 CMK 需要给账户 B 使用。账户 B 的 IAM 角色已有 `kms:Decrypt` 权限，但仍报 AccessDenied。缺了什么？
A) 账户 B 需要 MFA　B) 需要开启密钥轮换　C) 需要在账户 A 的 Key Policy 里授权账户 B　D) 需要 VPC Endpoint

**Q4.** RDS 数据库密码需要**每 30 天自动轮换**，且不希望自己写轮换逻辑。
A) SSM Parameter Store SecureString　B) Secrets Manager　C) KMS 直接存密码　D) S3 加密文件

**Q5.** 团队要存储约 200 个**应用配置项**（非敏感），要求**成本最低**。
A) Secrets Manager　B) SSM Parameter Store 标准层　C) DynamoDB 加密表　D) S3 + KMS

**Q6.** 加密的 EBS 快照需要**复制到另一个 Region** 做灾备。关于 KMS 密钥，正确的是？
A) 原密钥自动跨 Region 可用　B) 必须先解密再复制　C) 快照不支持跨 Region 复制　D) 复制时需指定目标 Region 的 KMS 密钥

**Q7.** EC2 上的应用需要读写 S3。**最安全**的凭证方式？
A) 把 access key 写在代码里　B) 把 access key 放在 EC2 user data　C) 给 EC2 附加 IAM Role　D) 存在 Parameter Store 里由应用读取 access key

**Q8.** 一家外部咨询公司需要临时访问你账户的资源。为防止「混淆代理人」攻击，应该？
A) 创建 IAM 用户给他们　B) AssumeRole + External ID　C) 共享 root 凭证　D) 开放公网访问

**Q9.** 合规要求密钥必须存放在**专用硬件 HSM、单租户环境**中。应选？
A) AWS Managed Key　B) CMK（AWS 生成）　C) CMK + CloudHSM 自定义密钥库　D) Parameter Store SecureString

**Q10.** 关于 Secrets Manager 和 Parameter Store，下列**错误**的是？
A) Secrets Manager 支持自动轮换 RDS 密码　B) Parameter Store 标准层免费　C) Parameter Store 的 SecureString 用 KMS 加密　D) Parameter Store 内置自动轮换功能
