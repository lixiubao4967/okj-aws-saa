# Week 3：高可用 + 负载均衡

## 1. ELB（Elastic Load Balancing）

- AWS 托管服务，自动扩容、高可用

### 三种类型

| 类型 | 简称 | 工作层 | 核心特点 |
|------|------|--------|---------|
| Application LB | ALB | Layer 7 (HTTP/HTTPS) | 基于 URL/Host/Header 路由，最常用 |
| Network LB | NLB | Layer 4 (TCP/UDP) | 超低延迟、百万级并发、静态 IP |
| Gateway LB | GWLB | Layer 3 (IP) | 第三方安全设备流量检查（GENEVE 协议，端口 6081） |

### 选型决策树

```
按 URL/Header 路由？ → ALB
极低延迟 / 静态 IP / TCP？ → NLB
第三方防火墙/IDS/IPS？ → GWLB
静态 IP + URL 路由都要？ → NLB → ALB 串联
```

### Target Group

三种 ELB 都有 Target Group：

| | ALB | NLB | GWLB |
|--|-----|-----|------|
| 目标类型 | EC2 / IP / Lambda | EC2 / IP / ALB | 安全设备（EC2） |
| 分发依据 | HTTP 内容 | Flow Hash（连接级别） | Flow Hash |

### ALB 核心概念

- **Listener**：监听端口和协议（如 HTTPS:443）
- **Target Group**：一组后端目标
- **路由规则**：基于 path（`/api/*`）、host（`api.example.com`）、header 等分发

### 关键特性

| 特性 | 说明 |
|------|------|
| Cross-Zone LB | 跨 AZ 均匀分流（ALB 默认开，NLB 默认关） |
| Sticky Sessions | 同一用户→同一实例（ALB 支持，基于 Cookie） |
| SSL Termination | ELB 层解密 HTTPS，后端用 HTTP |
| Connection Draining | 实例下线前等待现有请求完成（默认 300 秒） |

---

## 2. Route 53（DNS 服务）

- 53 号端口 → 取名 Route 53
- **100% SLA**，AWS 唯一

### 常见记录类型

| 记录类型 | 作用 |
|---------|------|
| A | 域名 → IPv4 |
| AAAA | 域名 → IPv6 |
| CNAME | 域名 → 另一个域名（**不能用于根域名**） |
| Alias | AWS 专有，域名 → AWS 资源（**可用于根域名，免费**） |

> 根域名指向 ALB/CloudFront → 永远选 Alias Record

### Hosted Zone

- **Public**：解析公网域名
- **Private**：只在指定 VPC 内解析

### 7 种路由策略

| 策略 | 适用场景 | 关键词 |
|------|---------|--------|
| Simple | 最简单，不做健康检查 | 无特殊需求 |
| Weighted | 按权重分配流量 | A/B 测试、蓝绿部署 |
| Latency-based | 导向延迟最低的 Region | 最佳性能体验 |
| Failover | Primary 挂 → 切 Secondary | 灾备（DR） |
| Geolocation | 按用户所在国家/大陆 | **合规要求（如 GDPR）** |
| Geoproximity | 按地理距离 + Bias 偏移 | 需要 Traffic Flow |
| Multi-Value | 类似 Simple + 健康检查 | 返回最多 8 个健康 IP |

> Geolocation = 合规/本地化；Latency-based = 最佳性能。不要混淆！

### 健康检查

- HTTP / HTTPS / TCP
- 默认 30 秒间隔，连续 3 次失败判定不健康
- 可组合（Calculated Health Check）：AND / OR / NOT

---

## 3. CloudFront（CDN）

- 全球 400+ Edge Location 缓存内容

### 两种 Origin

| Origin 类型 | 说明 |
|-------------|------|
| S3 Bucket | 静态文件 |
| Custom Origin | 任何 HTTP 端点（ALB、EC2 等） |

### S3 Origin 安全 — OAC

- **OAC（Origin Access Control）**：S3 只接受 CloudFront 的请求
- 旧版叫 OAI，新题选 OAC
- "禁止直接访问 S3" → OAC + Bucket Policy

### 缓存机制

- **TTL**：通过 Cache-Control / Expires 控制
- **Cache Invalidation**：手动清缓存，要收费
- **Cache Policy**：定义哪些 Header/Cookie/Query String 影响缓存键

### 安全特性

| 特性 | 说明 |
|------|------|
| HTTPS | 可用 ACM 免费证书 |
| Geo Restriction | 按国家白名单/黑名单 |
| Signed URL | 单个文件的临时访问 |
| Signed Cookie | 多个文件的临时访问 |
| Field-Level Encryption | Edge 层加密敏感字段 |

### Signed URL 区分

- **CloudFront Signed URL** → 通过 CDN 分发付费内容
- **S3 Pre-Signed URL** → 临时直接访问 S3 文件

> 单个文件 → Signed URL；多个文件 → Signed Cookie

### CloudFront vs S3 Cross-Region Replication

| | CloudFront | S3 CRR |
|--|-----------|--------|
| 原理 | 缓存，有 TTL | 实际复制对象 |
| 覆盖 | 全球自动 | 手动选目标 Region |
| 适合 | 静态内容全球分发 | 特定 Region 低延迟读取 |

### Lambda@Edge vs CloudFront Functions

| | CloudFront Functions | Lambda@Edge |
|--|---------------------|-------------|
| 语言 | JavaScript | Node.js / Python |
| 执行时间 | < 1ms | 最长 5-30 秒 |
| 适用 | Header 修改、URL 重写 | 复杂逻辑、访问外部服务 |

---

## 架构全貌

```
用户 → Route 53（DNS）→ CloudFront（CDN）→ ELB（负载均衡）→ EC2 / ASG
```

---

## 学习进度

- [x] ELB（ALB / NLB / GWLB）— 已答题通过
- [x] Route 53（路由策略 × 7）— 已答题通过
- [x] CloudFront（缓存 + 安全）— 已答题通过
- [x] Week 3 综合检测 — 7/7 全对
- [ ] 进入 Week 4
