# TD Practice Test 2 复盘

> 成绩：**33/65 = 50%**（2026-09-28，Exam mode）
> 复盘完成：2026-10-02
> 用时 2h30（Udemy 限时 130 分钟，超 20 分钟；**但真考有 160 分钟，实际未超时**）

---

# 一、趋势：四个域全部上升，无一倒退

| Domain | Test 1 | **Test 2** | 变化 |
|---|---|---|---|
| **Secure Architectures** | 45% | ⭐ **67%** | ⭐ **+22** |
| **Resilient Architectures** | 37% | **47%** | **+10** |
| **High-Performing** | 35% | **40%** | +5 |
| **Cost-Optimized** | 33%（n=3） | **40%**（n=10） | +7 |
| **总分** | **38%** | ⭐ **50%** | ⭐ **+12** |

> 🔑 **安全域 +22 分最大**——正是复习投入最多的域（WAF/Shield 边界、加密体系、Object Lock、IAM 联合身份）。
> **说明方法有效，同样的打法用在其他域应该也能见效。**

---

# 二、🔬 本轮最重要的方法论验证

## 被动阅读 vs 主动提取

| 日期 | 方式 | 结果 |
|---|---|---|
| 9/28 | **通读 35 分钟** cheatsheets | 15 题快速回忆 → ⭐ **43%** |
| 9/28 | **主动回忆 + 针对性补空白** | 8 题复测 → ⭐ **94%** |
| 10/1 | Route 53 读一遍 + **立即回忆** | 6 题 → ⭐ **6/6 全对** |
| 10/1 | Data Transfer Terminal（9/29 学的） | 两天后遇到 → ⭐ **主动用上，推理完整** |

### ✅ 结论（后续一律按此执行）

```
❌ 通读 cheatsheets / 重读笔记     ——  效果 43%
⭐ 我问你答 / 关键词→服务 快速回忆 ——  效果 94%
```

**testing effect：提取一次 > 重读五次。**

---

# 三、🔴 新发现的失误模式

## ① 「一行表格 = 没学过」（本轮 3 次）

| 知识点 | 笔记位置 | 结果 |
|---|---|---|
| **Lake Formation** | `07-analytics.md:129` 一行 | ❌ 完全不懂 |
| **DLM** | `01`、`03` 各一行 | ❌ 答错 |
| **RDS Proxy** | `04-database:90` 一行 | ❌ 答错 |

> 🔑 **规律：笔记里只占一行的条目 ≈ 没学过。**
> 需要「是什么 + 什么时候用 + 和谁区分」三要素才能形成记忆。

## ② 题意方向读反（新模式，Q36）

```
题干：客户的防火墙只允许访问白名单 IP
误读：以为是"我要限制谁能访问我" → 想用 NACL
实际：⭐ "我需要固定 IP，好让对方加白名单" → NLB + EIP
```

**✅ 防御动作**：读到"防火墙/白名单/访问控制"时先问 **「谁在限制谁？」**
```
我限制别人  →  安全组 / NACL / WAF
别人限制我  →  ⭐ 我需要固定 IP（NLB+EIP / Global Accelerator）
```

## ③ 英文长句读不懂（Q36 自述）

**✅ 三个考场技巧**
```
① 抓主谓宾，跳过 -ed 定语和介词短语
② 先读最后的【问句】，再回头扫题干
③ 长题干先跳到"问句前的最后 2-3 句"——要求通常在那里
```

**高频易误读句式**：`without having to` / `rather than` / `other than` / `but` / `as well as` / `must` / `-ed` 短语

## ④ 后半段疲劳失分（Q58）

Q58 是最简单的题（CDN = CloudFront），却选了 ElastiCache。**第 58 题 + 超时 20 分钟 = 疲劳/时间压力**。

**✅ 考试当天对策**
```
160 分钟 ÷ 65 题 = 2 分 27 秒
⭐ 留 15 分钟检查 → 实际每题 2 分 13 秒
⭐ 做到第 40 题看一眼时间：>100 分钟就要加速
⭐ 累了深呼吸 10 秒再继续——疲劳最容易错【简单题】
```

## ⑤ 老问题复现：选"中间选项"（第 4 次）

| 题 | 选了 | 正确 |
|---|---|---|
| 官方 Q1 | EKS 托管节点组 | ECS Fargate |
| 官方 Q8 | Instance Scheduler | Lambda |
| TD1 Q57 | ECS Auto Scaling | Lambda |
| **TD2 Q9** | **Beanstalk** | ⭐ **S3 静态托管** |

**✅ 题干给极端约束词（`most cost-effective` / `fastest` / `minimum`）→ 先看最轻量的选项能不能满足。**

---

# 四、本轮补齐的知识点

## 🔴 Route 53 全套（4 月学的，已遗忘，本轮重新激活）

| 内容 | 要点 |
|---|---|
| **Alias vs CNAME** | ⭐ **CNAME 不能用于根域名**（DNS 协议限制）；Alias 可以、**免费**、自动跟随 |
| **7 种路由策略** | Simple / **Weighted**（按比例）/ **Latency**（最快）/ **Failover**（主备）/ **Geolocation**（按国家，合规）/ **Geoproximity**（按距离，**可 bias 调整**）/ **Multivalue**（最多 8 个健康 IP） |
| ⭐ **Geolocation vs Geoproximity** | **Geolocation = 国家边界固定**（合规）<br>**Geoproximity = 地理距离 + bias 可调**（"更大一部分流量"） |
| ⭐ **Geolocation vs Latency** | **Geolocation 看人在哪**（合规）<br>**Latency 看哪个快**（性能） |
| **Hosted Zone** | Public（公网）/ Private（**只在指定 VPC 内**） |
| **健康检查** | HTTP/HTTPS/TCP，默认 30 秒，连续 3 次失败；**Failover 依赖它** |
| ⚠️ **DNS TTL** | Failover **不是即时**；要秒级 → **Global Accelerator** |
| ⚠️ **Route 53 是全球服务** | 没有 Region 概念 → "必须和 hosted zone 同 Region" 一律错 |
| **S3 静态网站 + 自定义域名** | ⭐ **bucket 名必须 = 域名** |

## 🆕 本轮新增服务

| 服务 | 要点 |
|---|---|
| ⭐ **AWS Control Tower** | **Landing Zone**（一键建多账户环境）+ **Account Factory**（标准化批量建账户）+ **Guardrails**（Preventive=SCP / Detective=Config）<br>⚠️ **Control Tower 管【账户】，RAM 管【资源共享】** |
| ⭐ **Data Transfer Terminal** | **AWS 实体站点**，带设备去现场高速上传；**绕过自己的带宽**<br>**vs Snow**：你带去 vs AWS 寄来 |
| ⭐ **Amazon Rekognition** | **图像/视频分析**：内容审核、人脸、物体识别<br>⚠️ `least effort` + AI → 用现成服务，**不要 SageMaker**（要训练） |
| **Amazon MQ** | 托管 **ActiveMQ/RabbitMQ**；⭐ 用于**迁移现有 JMS/AMQP 应用不改代码**；其他队列场景一律 SQS |
| **AWS SWF** | ⚠️ **遗留服务**，已被 Step Functions 取代 |
| **Elastic Fabric Adapter (EFA)** | **OS-bypass** 网络接口，HPC / ML 分布式训练 |
| **AppSync** | 托管 **GraphQL** |

## 🆕 本轮新增概念

| 概念 | 要点 |
|---|---|
| ⭐ **VPC Peering 不传递** | ① **不支持传递路由**（A-B-C，A 到不了 C）<br>② ⭐ **不支持 Edge-to-Edge**：peer 的 VPC **不能借用**对方的 **VPN/DX/IGW/NAT/Gateway Endpoint**<br>→ 多 VPC 都要连本地 ⇒ **Transit Gateway**（N+1 条 vs N(N-1)/2+N 条） |
| ⭐ **VPC Endpoint Policy** | 挂在端点上，控制**通过这个端点能访问哪些资源**；默认全放行；**不替代** IAM/bucket 策略<br>**方向**：限制"能访问哪些资源"→ Endpoint Policy；限制"哪些来源能访问我"→ Bucket Policy + `aws:SourceVpce` |
| ⭐ **RDS Proxy** | **连接池**，解决 Lambda 高并发导致的 `too many connections`；还能**减少故障切换时间 66%**、集成 Secrets Manager |
| ⭐ **DLM vs AWS Backup** | **DLM**：只管 EBS 快照 + AMI，⭐ **免费**<br>**AWS Backup**：跨服务、收费、有 **Vault Lock** |
| ⭐ **ECS 两种 IAM 角色** | **`taskRoleArn`** = 容器里应用的权限（访问 S3/SQS）<br>**`executionRoleArn`** = ECS 代理的权限（拉 ECR 镜像、写日志） |
| ⭐ **SNS Message Filtering** | **SNS 有过滤功能**（之前误以为没有）；filter policy 配在**订阅**上，按消息属性过滤 |
| ⭐ **DynamoDB 跨账户备份** | **on-demand backup 和 PITR 都不能跨账户** → ⭐ **必须用 AWS Backup** |
| ⭐ **NLB + EIP** | ⭐ **ALB 不能有固定 IP**；需要固定 IP → **NLB + Elastic IP** 或 **Global Accelerator**<br>组合：NLB（固定 IP）在前 + ALB（L7 路由）在后 |
| ⭐ **ALB 支持 gRPC** | gRPC 基于 **HTTP/2** = L7 → **只有 ALB**；NLB(L4)/GWLB(L3) 都不行 |
| **Site-to-Site VPN** | **CGW（本地侧）必须有静态公网 IP**；VGW **不能关联 EIP**；自动建 **2 条隧道** |
| **Canary vs Blue-Green** | **Canary** = 一套环境按比例放量，⭐ **便宜**<br>**Blue-Green** = 两套完整环境，⭐ **贵但回滚最快** |
| **ASG 默认终止策略** | ① 实例**最多的 AZ** → ② **最老启动模板** → ③ 最接近计费小时 → ④ 随机 |
| ⭐ **Transfer Acceleration vs CloudFront** | ⭐ **分界线是「有没有缓存」**<br>TA：优化**上传**，不缓存；CloudFront：优化**分发**，⭐ **缓存** |
| ⭐ **Cost 工具分工** | **Cost Allocation Tags**（分类，前提）/ **Cost Explorer**（分析）/ **Budgets**（超支告警）/ **CUR**（原始明细）<br>⚠️ 打标签后**必须在 Billing 控制台激活** |
| ⭐ **VPC 必须有 IPv4** | 不能删除、不能禁用；但**子网可以是 IPv6-only** |
| **EBS 事实** | 冗余**仅 AZ 内**；只能挂**同 AZ** 实例；快照存 **S3**；⭐ **Elastic Volumes 可在线改类型/大小/IOPS**（只能增大） |

---

# 五、🔧 TD 解析的三处不准确（已修正）

| TD 说法 | 实际 |
|---|---|
| "EC2 按小时计费" | ⭐ Linux **按秒计费**（最少 60 秒） |
| "Object Lock 三种模式" | ⭐ 两个**独立维度**：机制（Retention/Legal Hold）+ 模式（Governance/Compliance）；**Legal Hold 没有模式** |
| "One Zone-IA 不支持全球分发" | ⭐ **所有存储类都能配 CloudFront**；One Zone-IA 的真问题是**单 AZ 不够 durable** |

> 🔑 **遇到感觉别扭的解析，值得质疑。**

---

# 六、考前最后 2 天清单（10/3 收尾 · 10/4 考试）

## 🔴 P0：三个必做动作（每题）
```
① 约束句扫描  without / does not require / rather than / must / but / never
② 虚构能力检查 这个功能真的存在吗？这个操作做得到吗？
③ 要求计数    题干几个独立要求？多选选几个？
```

## 🔴 P0：历史重复错点
```
API Gateway + traffic spikes       → Throttling（429）
防 S3 意外删除                      → Versioning + MFA Delete
ASG 缩容：5xx → Deregistration Delay ／ 终止谁 → AZ均衡+最老启动模板
DynamoDB 跨账户备份                 → AWS Backup（不是 PITR/on-demand）
需要固定 IP                         → NLB + EIP（ALB 不行）
```

## 🟡 P1：节奏控制
```
160 分钟 ÷ 65 题 = 2 分 27 秒（留 15 分钟检查 → 实际 2 分 13 秒）
第 40 题时看表：已用 >100 分钟 → 加速
超 2 分半 → Mark for review，果断跳过
⭐ 后半段注意疲劳，简单题也要读完题干
```

## 🟢 P2：复习方式
```
❌ 不要通读  ⭐ 用「我问你答」的方式做快速回忆
```
