# Week 1-2 综合题 — 错题记录

## 错题 1：EBS Multi-Attach 支持哪种类型？

**我的答案：** C) st1  
**正确答案：** B) io2 Block Express

### 错因分析
- 知道 io2 的 IOPS 更强，但没有记住 Multi-Attach 是 io 系列独有的能力
- 误以为 HDD 类型也可能支持 Multi-Attach

### 要点
- **Multi-Attach 只有 io1 / io2（含 Block Express）支持**
- 同一 AZ 内最多 16 个 Nitro 实例同时挂载
- AWS 不提供文件系统级锁，应用层需自行管理并发（如 GFS2、OCFS2）
- gp 系列、st1、sc1 均不支持 Multi-Attach

---

## 错题 2：哪种操作不会导致 Instance Store 数据丢失？

**我的答案：** C) 底层硬件故障  
**正确答案：** B) Reboot

### 错因分析
- 错误认为 Stop/Start 和 Reboot 原理一样
- 没理解 Stop/Start 会切换物理机，而 Reboot 不会

### 要点
| 操作 | Instance Store 数据 | 原因 |
|------|-------------------|------|
| **Reboot** | ✅ 保留 | 同一台物理机，只重启 OS |
| **Stop → Start** | ❌ 丢失 | 可能被调度到不同物理机 |
| **Terminate** | ❌ 丢失 | 实例销毁 |
| **硬件故障** | ❌ 丢失 | 本地磁盘随物理机损坏 |

> 口诀：**Instance Store = 临时存储，只有 Reboot 安全**

---

## 本次成绩：4 题对 2 题（50%）
- ✅ 题目 3：Scheduled Scaling 适用于固定时间高峰
- ✅ 题目 4：EBS 跨 Region 迁移 = Snapshot → Copy → Create
