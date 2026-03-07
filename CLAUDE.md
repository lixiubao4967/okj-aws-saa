# 项目说明

这是一个 AWS 认证考试学习仓库，目标是依次通过：
1. AWS Certified Solutions Architect - Associate (SAA-C03)
2. AWS Certified Solutions Architect - Professional (SAP-C02)

## 用户需求

- 遇到不懂的 AWS 知识点时，用简洁清晰的中文解释
- 学完一个模块后，可以要求我出题考察，或总结成笔记写入 notes/ 目录
- 刷题遇到不理解的题目，粘贴题目内容，我来分析解题思路
- 笔记写完后提醒用户 commit 并 push 到 GitHub

## 仓库结构

```
okj-aws-saa/
├── CLAUDE.md              # 本文件
├── README.md              # 总学习计划（SAA + SAP 两阶段 10 周计划）
├── notes/
│   ├── saa/               # SAA-C03 知识点笔记（按模块拆分）
│   └── sap/               # SAP-C02 进阶笔记
├── practice/
│   ├── saa/               # SAA 错题记录 + 模拟考试记录
│   └── sap/               # SAP 错题记录
└── cheatsheets/           # 考前速查表
```

## Git 信息

- 远程仓库：git@github.com:lixiubao4967/okj-aws-saa.git
- 主分支：develop
- SSH 认证（id_ed25519 已配置到 GitHub 账号 lixiubao4967）
