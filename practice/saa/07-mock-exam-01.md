# 混合模拟卷 ① — Claude 出题（65 题，分 3 批）

> 2026-09-14 起 · 跨域混合、题干按真考规格加长
> **目的**：练「从长题干里提炼约束词」+「跨域切换」，作为真实题库前的热身
> ⚠️ 答案不写在本文件里

**建议节奏**：每批限时 ≈ 题数 × 2.4 分钟（第 1 批 22 题 ≈ **53 分钟**）

---

## 第 1 批（Q1–Q22）

**Q1.** 一家媒体公司在数百个 S3 存储桶中存放用户上传的文档，总量约 40 TB。近期合规审计提出两项要求：第一，安全团队必须能够**自动识别哪些对象中包含护照号、信用卡号等个人身份信息**；第二，一旦这些敏感对象被访问或删除，必须能够**追溯是哪个 IAM 主体在什么时间操作的**。团队希望尽量使用 AWS 托管服务，减少自研。以下哪个组合能同时满足这两项要求？

A) Amazon Macie + CloudTrail Management Events
B) Amazon GuardDuty + S3 服务器访问日志
C) Amazon Macie + CloudTrail Data Events
D) AWS Config Rules + CloudTrail Data Events

---

**Q2.** 某金融交易平台在东京区域的三个可用区各部署了一个私有子网，运行着自动扩缩的 EC2 实例。所有私有子网的默认路由都指向位于 **ap-northeast-1a 公有子网中的单个 NAT Gateway**。上个月 1a 区发生故障，导致**全部三个可用区**的实例都无法访问外部许可证服务器，业务中断 40 分钟。此外，财务部门也指出当前架构产生了可观的**跨可用区数据传输费用**。架构师应如何调整？

A) 在每个可用区的公有子网各部署一个 NAT Gateway，并让各私有子网路由到本可用区的 NAT Gateway
B) 在 ap-northeast-1b 再部署一个 NAT Gateway 作为备用，故障时手动切换路由
C) 将 NAT Gateway 替换为放在 Auto Scaling 组中的 NAT 实例
D) 为 VPC 创建 S3 Gateway Endpoint，让流量不再经过 NAT Gateway

---

**Q3.** 一个电商平台使用 DynamoDB 存储商品目录。促销期间，同一批热门商品的详情会被**反复读取**，当前读取延迟为个位数毫秒，但业务方要求降到**微秒级**。开发团队希望**对现有应用代码的改动尽可能小**（当前代码使用 AWS SDK 的 DynamoDB 客户端）。

A) 在应用前增加 ElastiCache for Redis 缓存层，由应用自行管理缓存读写
B) 提高该表的预置 RCU 并启用自适应容量
C) 启用 DynamoDB Global Tables，就近读取
D) 部署 DynamoDB Accelerator (DAX) 集群

---

**Q4.** 某基因测序公司每晚运行一批数据处理作业，使用 50 台计算优化型 EC2 实例，单次约 4 小时。该作业**已实现检查点机制，被中断后可以从断点自动恢复**，业务上也允许当晚稍晚完成。目前全部使用按需实例，月账单中这部分占比最高。成本优化团队希望**最大幅度降低这部分支出**。

A) 购买 3 年期标准预留实例
B) 购买 Compute Savings Plans
C) 使用 Spot 实例（配合 Spot Fleet 分散实例类型）
D) 迁移到专用主机以获得更低单价

---

**Q5.** 一个 Lambda 函数被配置在 VPC 的私有子网中，用于处理订单。它需要：① 从 Secrets Manager 读取数据库凭证；② 调用**外部第三方支付网关的公网 API**。部署后发现两项调用**全部超时**。数据库连接本身是正常的。架构师应如何修复，同时保持成本尽可能低？

A) 在公有子网部署 NAT Gateway，并将私有子网的默认路由指向它
B) 仅为 Secrets Manager 创建 Interface Endpoint
C) 把该 Lambda 移出 VPC 以恢复公网访问能力
D) 为 Secrets Manager 创建 Gateway Endpoint

---

**Q6.** 某支付网关需要处理来自商户的交易指令。业务要求：**同一个商户的交易必须严格按照提交顺序处理**，不同商户之间可以并行；同时**绝对不允许同一笔交易被重复处理**（会造成重复扣款）。系统在大促期间流量会突增约 10 倍，需要有缓冲能力。

A) SQS 标准队列，并在消费端应用中实现去重逻辑
B) SNS FIFO Topic 直接触发 Lambda
C) Kinesis Data Streams，使用随机 UUID 作为 partition key
D) SQS FIFO 队列，以 merchant_id 作为 MessageGroupId

---

**Q7.** 一家 SaaS 公司的用户分布在北美、欧洲和亚洲。应用由两部分组成：托管在 us-east-1 的 S3 静态资源（JS/CSS/图片），以及同区域 ALB 后面的动态 API。欧洲和亚洲用户同时抱怨**静态资源加载慢**和 **API 响应慢**。团队希望用**一套方案同时改善两者**。

A) 为 S3 静态资源配置 CloudFront，动态 API 保持直连
B) 为 S3 启用 Transfer Acceleration
C) 将 S3 内容跨区复制到欧洲和亚洲区域
D) 配置 CloudFront，同时以 S3 为源（静态）和以 ALB 为源（动态）

---

**Q8.** 某公司的应用日志写入 S3，当前总量 500 TB 且持续增长。使用模式非常明确：**日志在写入后的 30 天内会被频繁查询**用于排障；**30 天后几乎不再访问**；出于合规要求必须**保留 7 年**；对于超过 90 天的日志，即使需要取回，**12 小时以内完成即可接受**。要求存储成本最低。

A) 全部保留在 S3 Standard
B) 全部启用 S3 Intelligent-Tiering，由 AWS 自动分层
C) 写入时即设为 S3 Standard-IA
D) 配置生命周期策略：Standard 保留 30 天 → 转 Glacier Flexible Retrieval → 90 天后转 Glacier Deep Archive

---

**Q9.** 一家外部数据分析供应商需要**只读访问**你账户中某个特定 S3 存储桶的内容。该供应商拥有自己的 AWS 账户，并且同时为多家客户提供服务。你的安全团队要求遵循最小权限原则，并且必须**防范「混淆代理人（confused deputy）」攻击**。

A) 在你的账户中创建一个 IAM 用户，把 access key 发给供应商
B) 创建跨账户 IAM 角色并配置 External ID，由供应商账户扮演该角色
C) 将存储桶设为公开，并用桶策略限制供应商的出口 IP
D) 配置桶策略，直接允许供应商账户的 root 主体访问

---

**Q10.** 某公司运行 RDS for MySQL（已启用 Multi-AZ）。两个痛点：① 每次故障转移时应用会出现约 **60–120 秒**的不可用，业务方希望缩短到 **30 秒以内**；② 数据分析团队的报表查询正在**拖慢生产库的写入性能**，希望把这类读负载分流出去。需要一个方案同时解决两者。

A) 增加一个 RDS 只读副本供报表使用
B) 提升 RDS 实例规格并启用增强监控
C) 迁移到 Aurora MySQL，创建 Aurora 副本承担报表读取，并配置故障转移优先级
D) 为现有 RDS 再启用一组 Multi-AZ 备用实例

---

**Q11.** 一个内容管理系统运行在 20 台 EC2 实例上，分布于 3 个可用区。这些实例需要**同时读写同一份文件数据**（大量中小文件），应用依赖标准的 **POSIX 文件系统语义**，且存储容量需要**随数据增长自动扩展**，无需人工预置。

A) 创建一个启用 Multi-Attach 的 io2 EBS 卷，挂载到所有实例
B) 创建 Amazon EFS 文件系统，在每个可用区创建挂载目标
C) 使用 S3 并通过 SDK 读写对象
D) 部署 FSx for Windows File Server

---

**Q12.** 一家企业在 4 个 AWS 账户中共运行约 200 台 EC2 实例（Amazon Linux 和 Ubuntu 混合），另有若干 ECR 容器镜像。安全团队要求**持续检测这些工作负载中的已知 CVE 漏洞**，并希望在有新漏洞公布时能自动重新评估，同时尽量**避免在每台实例上部署和维护第三方 agent**。

A) 启用 Amazon Inspector，并通过 Organizations 委派管理员集中管理
B) 启用 Amazon GuardDuty 并开启恶意软件防护
C) 在每台实例上部署第三方漏洞扫描 agent，结果汇总到 SIEM
D) 编写 AWS Config 自定义规则检查软件版本

---

**Q13.** 某公司的营销官网是纯静态站点，托管在 S3 并通过 CloudFront 分发，日均访问量较大。管理层提出新要求：**即使托管桶所在的整个 AWS 区域发生故障，网站也必须保持可访问**。团队希望方案尽量简单、无需维护服务器。

A) 将内容通过跨区复制（CRR）同步到第二个区域的 S3 桶，并在 CloudFront 中配置源站故障转移（Origin Failover）组
B) 为 S3 桶启用多可用区冗余
C) 在第二个区域部署 EC2 备份站点，用 Route 53 故障转移记录切换
D) 为 S3 桶启用版本控制和 MFA Delete

---

**Q14.** 合规部门要求：组织内**所有新建的 EBS 卷都必须加密**；对于已经存在的未加密卷，需要**自动检测出来并触发修复流程**，而不是依赖人工巡检。

A) 配置 AWS Config 托管规则 `encrypted-volumes`，并关联 SSM Automation 修复动作
B) 创建 CloudTrail 指标筛选器，在检测到 CreateVolume 时告警
C) 启用 Amazon Inspector 扫描 EC2 实例的存储配置
D) 使用 Amazon Macie 扫描 EBS 卷内容

---

**Q15.** 一个面向移动端的 API 由 API Gateway + Lambda 构成，Lambda 查询 Aurora 数据库。监控显示：**平均响应时间约 200 ms，但 p99 会突增到 4 秒**。进一步排查发现，这些慢请求**都发生在新的执行环境初始化时**，与数据库查询耗时无关。业务要求消除这类突发延迟。

A) 将 Lambda 内存从 512 MB 提高到 3008 MB
B) 将 API Gateway 的集成超时从 29 秒提高到 60 秒
C) 引入 RDS Proxy 以复用数据库连接
D) 为该 Lambda 函数配置 Provisioned Concurrency

---

**Q16.** 某公司的开发和测试环境共有 60 台 EC2 实例，实际**只在工作日 08:00–20:00 被使用**，其余时间闲置，但目前全部按需实例 7×24 运行。团队希望在**不改变实例规格、不影响开发体验**的前提下大幅降低这部分成本。

A) 为这些实例购买 1 年期预留实例
B) 使用 Instance Scheduler（或 EventBridge + Lambda）在非工作时间自动停止、工作时间自动启动
C) 将开发测试环境全部迁移到 Spot 实例
D) 购买 Compute Savings Plans 覆盖这部分用量

---

**Q17.** 一款全球游戏需要在 **us-east-1 和 ap-northeast-1 两个区域同时接受玩家数据的写入**（活动-活动架构），并要求两边的数据能够**双向同步**，且对写入冲突有内置的处理机制。团队不希望自行开发复制逻辑。

A) 启用 DynamoDB Global Tables，在两个区域各创建一个副本表
B) 使用 DynamoDB Streams 触发 Lambda，自行把变更复制到另一区域的表
C) 在第二区域创建 DynamoDB 跨区只读副本
D) 在两个区域各部署 DAX 集群并互相同步

---

**Q18.** 某在线服务的登录接口部署在 ALB 之后，近期遭到**撞库攻击**：少量源 IP 在短时间内发起了数万次登录请求。同时，业务部门指出公司**并不在某些国家开展业务**，希望直接阻断来自这些国家的全部流量。需要一个方案同时应对这两点。

A) 在子网 NACL 中添加 deny 规则封禁这些 IP 和网段
B) 启用 AWS Shield Standard 并开启高级防护
C) 在 ALB 上关联 AWS WAF，配置基于速率的规则（Rate-based Rule）和地理匹配规则（Geo Match）
D) 收紧 ALB 的安全组入站规则

---

**Q19.** 一家制造企业需要把本地数据中心的 **80 TB 历史数据一次性迁移到 S3**。该站点的互联网出口带宽为 **100 Mbps**，且该链路还要承载日常业务流量。项目要求**在两周内完成迁移**。目前尚未部署任何专线。

A) 使用 S3 Transfer Acceleration 通过现有互联网链路上传
B) 申请 Direct Connect 专线后通过专线传输
C) 订购 AWS Snowball Edge 设备，本地拷贝后寄回 AWS
D) 使用 S3 分段上传（Multipart Upload）并行化传输

---

**Q20.** 某服务运行在 ECS Fargate 上，位于 ALB 之后。每次发布新版本时，用户会在**部署期间间歇性收到 5xx 错误**。排查发现旧任务在仍有活跃连接时就被停止，同时部署过程中一度只有部分任务处于健康状态。需要实现**发布期间零错误**。

A) 将部署方式改为 All at once 以缩短发布窗口
B) 将服务的期望任务数从 4 提高到 8
C) 将启动类型从 Fargate 改为 EC2
D) 配置 ALB 目标组的取消注册延迟（deregistration delay），并将 ECS 滚动更新的最小健康百分比设为 100%

---

**Q21.** 一家企业在 AWS Organizations 下管理 25 个成员账户。审计部门要求：**集中收集所有账户的全部 API 调用记录**；日志必须**可证明未被篡改**；并且需要**保留 7 年**，期间任何人（包括管理员）都不能删除。

A) 在每个成员账户分别创建 Trail，各自投递到本账户的 S3 桶
B) 在管理账户创建 Organization Trail，投递到中央 S3 桶，启用日志文件完整性校验，并对该桶启用 S3 Object Lock（合规模式）
C) 将各账户的 CloudTrail 日志发送到中央 CloudWatch Logs 日志组
D) 配置 AWS Config Aggregator 汇总所有账户的配置项

---

**Q22.** 某数据团队维护着一个 **7×24 运行的 Redshift 集群**，其唯一用途是让分析师**每月几次**对存放在 S3 上的 5 年期应用日志（CSV 格式，约 200 TB）执行临时 SQL 查询。财务发现该集群的成本与其使用频率严重不匹配。要求在保留 SQL 查询能力的前提下大幅降本。

A) 将 Redshift 集群降配到更小的节点类型并启用暂停/恢复
B) 把日志转换为 Parquet 并按日期分区，改用 Amazon Athena 直接查询 S3，下线 Redshift 集群
C) 保留 Redshift 集群并使用 Redshift Spectrum 查询 S3
D) 改用 EMR 集群按需拉起运行 Spark SQL
