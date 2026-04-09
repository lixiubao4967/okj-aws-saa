# Week 4：S3 存储 + 安全

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

## 8. S3 Access Points

- 为桶创建多个独立入口，每个入口有独立的 DNS 名称和访问策略
- 解决多团队访问同一桶时 Bucket Policy 过于复杂的问题
- 可限制只允许从特定 VPC 访问（网络隔离）
- 桶 Policy 可设为"只允许通过 Access Point 访问"，防止绕过

**选型：** 2-3 个简单规则用 Bucket Policy；团队多/策略复杂用 Access Points
