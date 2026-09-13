# P4 安全 + 监控 — 错题记录

## 成绩趋势

| 日期 | 模块 | 得分 |
|------|------|------|
| 2026-09-13 | KMS + 密钥凭据管理 | **9/10（90%）** ✅ |

答对：CMK 判据、信封加密、KMS 双重授权、Secrets Manager 轮换、Parameter Store 成本、
　　　IAM Role（识破 Parameter Store 存 access key 的诱饵）、External ID、CloudHSM、反向题 Q10
答错：加密 EBS 快照跨 Region 复制

---

## 错题 14：加密 EBS 快照跨 Region 复制

**场景：** 加密的 EBS 快照要复制到另一个 Region 做灾备，关于 KMS 密钥哪项正确？

**我的答案：** C) 快照不支持跨 Region 复制　**正确答案：** D) 复制时需指定目标 Region 的 KMS 密钥

### 错因 —— 选了一个「功能不存在」的选项

**加密的 EBS 快照完全支持跨 Region 复制**，这正是最标准的灾备做法。

### 知识点：复制过程中密钥会换掉

```
源 Region (ap-northeast-1)          目标 Region (us-west-2)
  快照（用 KMS-Key-Tokyo 加密）
         │
         │  CopySnapshot 时指定
         │  KmsKeyId = KMS-Key-Oregon
         ▼
                                    快照（用 KMS-Key-Oregon 加密）
```

**为什么必须换密钥**：KMS 密钥是 **Region 级资源**，东京的密钥在俄勒冈不存在，
目标 Region 无法用它解密。AWS 在复制时**解密 → 用目标 Region 密钥重新加密**
（内部完成，你只需指定目标密钥）。

> 🔑 串记 **Multi-Region Keys**：嫌每次指定麻烦，就用它 —— 同一密钥 ID 在多 Region
> 各有副本，复制时不用重新指定。

### 🚨 新增解题规则：警惕「XX 不支持 YY」类选项

这次的错误类型和之前不同：
- 之前错题（Athena 联邦查询、DynamoDB join、Firehose 重放）= **把某服务能力记错**
- 这次 = **否定了一个实际存在且常用的能力**

```
AWS 考题极少用「这个功能不存在」当正确答案
  —— 出题人想考「你知不知道该怎么做」，不是「你知不知道做不到」

判断倾向：
  选项越【具体、像操作步骤】→ 越可能是答案
  选项越【笼统的否定判断】  → 越可能是干扰项
```

---

## 💡 Q7 答对值得记一笔（最狡猾的诱饵）

**场景：** EC2 上的应用需要读写 S3，最安全的凭证方式？
**D 选项诱饵：** 把 access key 存在 Parameter Store 里由应用读取

听起来比写在代码里安全，但**根本没解决问题**：
1. 应用还是得**先有凭证**才能读 Parameter Store → 先有鸡还是先有蛋
2. 长期 access key 本身就是风险（不会自动过期、泄露后一直有效）

### 🔑 IAM Role 的本质 = 「不需要凭证的凭证」

EC2 通过实例元数据服务自动拿到**临时的、自动轮换的**凭证，
**全程没有任何长期密钥落地。**

> **凡是「EC2 / Lambda / ECS 怎么访问别的 AWS 服务」→ 答案永远是 IAM Role，不要犹豫。**
