# 等保 DJCP AI Agent 人格与记忆基础

> **等保 2.0 全栈 AI Agent 的「灵魂」定义**  
> 覆盖定级 → 差距分析 → 整改设计 → 测评实施 → 报告生成 → 政策释疑全流程  
> **独立使用，无需任何外部依赖**

[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Skill](https://img.shields.io/badge/skill-ready-green?style=flat-square)](skills/等保DJCP-AI-Agent-人格与记忆.md)

---

## ⚠️ 免责声明

本项目所有内容仅供**网络安全学习、合规评估和授权测试**参考。任何未经授权的安全评估行为均属违法，使用者需自行承担法律责任。本项目不对任何滥用行为承担责任。

---

## 简介

本仓库定义了一个**完整的等保 DJCP（等级保护）AI Agent 人格（Personality）与持久记忆（Persistent Memory）体系**。

**一句话**：让任何 AI 像「等保测评专家」一样思考、判断、回答问题——只需一个文件作为 System Prompt。

### 它能做什么

| 功能 | 说明 |
|------|------|
| 🎯 **定级分析** | 按 GB/T 22240-2020 完成定级，生成定级报告 |
| 🔍 **差距分析** | 对标 GB/T 22239-2019，逐条判定、符合率统计、风险分级 |
| 🛠 **整改设计** | 差距→可落地方案（含产品选型、预算、时间线） |
| 📋 **测评报告** | 公安部格式正式测评报告 |
| ⚠️ **重大风险判定** | 32 项重大风险触发项（A-01~A-32）自动匹配 |
| 📚 **政策释疑** | 公网安〔2025〕1846号 24 项问答精准查询 |
| 📝 **访谈清单** | 分角色现场测评访谈提纲 |
| 📊 **项目管理** | 全流程 WBS、进度跟踪、质量管控 |

### 内置知识

| 类别 | 内容 |
|------|------|
| 📖 国家标准 | GB/T 22239 / 28448 / 22240 / 25070 |
| 📜 法律法规 | 网络安全法、数据安全法、个保法、关基条例 |
| 🆕 最新政策 | 公网安〔2025〕1846号（24 项释疑） |
| 📐 定级矩阵 | 受侵害客体 × 侵害程度 |
| 📊 安全类 | 10 大安全类，各等级要求项数（二级 44 / 三级 58 / 四级 59） |
| ⚠️ 重大风险 | 32 项触发项速查（A-01~A-32） |
| 🔑 一票否决 | 三级系统 5 个关键控制点 |
| 📋 结论判定 | 2025 版测评结论标准（符合/基本符合/不符合） |
| 🌟 第五级 | 第五级系统定义、备案、测评要求 |
| 📅 项目全流程 | 6 阶段生命周期 |

---

## 🚀 快速开始

### 方式一：作为 System Prompt 注入（推荐）

将 `skills/等保DJCP-AI-Agent-人格与记忆.md` 的完整内容作为 System Prompt 即可生效：

```bash
# Claude Code（追加到项目规则）
cat skills/等保DJCP-AI-Agent-人格与记忆.md >> CLAUDE.md

# Codex CLI（启动时加载）
codex --system "$(cat skills/等保DJCP-AI-Agent-人格与记忆.md)"

# GPT / DeepSeek / Kimi 等
cat skills/等保DJCP-AI-Agent-人格与记忆.md | pbcopy
# 粘贴到 AI 对话的 System Prompt 中
```

单个文件即开即用，**无需安装、无需依赖、无需配置**。

### 方式二：Hermes Agent 安装

```bash
git clone https://github.com/lingfengz/dengbao-DJCP-AI-Agent.git
cd dengbao-DJCP-AI-Agent
mkdir -p ~/.hermes/profiles/default/skills/dengbao-DJCP-AI-Agent
cp skills/* ~/.hermes/profiles/default/skills/dengbao-DJCP-AI-Agent/
hermes restart
```

---

## 使用验证

加载后，输入以下任意一句测试是否生效：

| 你说 | Agent 应该能 |
|------|-------------|
| "帮我定个级" | 引导你提供系统信息，按定级矩阵分析 |
| "看看差多少" | 对标 GB/T 22239 逐项分析差距 |
| "Redis 没设密码" | 自动匹配 A-18 重大风险，输出风险说明 |
| "第五级系统要去哪备案" | 回答「省级公安机关网安部门」 |
| "2025版报告模板什么时候启用" | 回答「2025年3月20日后」 |
| "三级系统有多少要求项" | 回答「58项」 |
| "重大风险判定三原则是什么" | 回答「相关性、严重性、高发性」 |

---

## 目录结构

```
等保DJCP-AI-Agent/
├── README.md                                  # 本文件
├── LICENSE                                    # MIT License
└── skills/
    └── 等保DJCP-AI-Agent-人格与记忆.md         # 核心 Skill 文件（单文件即用）
```

**仅一个核心文件**——拷贝 `skills/等保DJCP-AI-Agent-人格与记忆.md` 到任何地方即可开始使用。

---

## 版本信息

| 版本 | 日期 | 变更 |
|:---:|:---:|------|
| v1.0 | 2026-06 | 初始版本。基于 GB/T 22239/28448/22240/25070，公网安〔2025〕1846号 |

---

## License

MIT License  
Copyright (C) 2026 lingfengz  
版权所有（C）2026 lingfengz
