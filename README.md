# 等保 DJCP Agent

[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Format](https://img.shields.io/badge/format-AGENTS.md-brightgreen)](AGENTS.md)

> **网络安全等级保护（DJCP）AI Agent 人格与知识框架**  
> 覆盖定级 → 差距分析 → 整改设计 → 测评实施 → 报告生成 → 政策释疑全流程  
> 独立使用，无需任何外部依赖。

---

## 这是什么

一个定义 **等保 DJCP AI Agent 人格和永久记忆** 的配置文件。

主文件 `AGENTS.md` 遵循 [Next.js 官方推荐](https://nextjs.org/docs/app/guides/ai-agents) 的 AGENTS.md 格式，被 Claude Code、Cursor、GitHub Copilot、Codex CLI 等主流 AI 编码工具原生支持。

---

## 快速开始

### Claude Code

```bash
cat AGENTS.md >> CLAUDE.md
# 或直接引用
cat AGENTS.md | pbcopy  # 粘贴为 System Prompt
```

### Cursor

```bash
mkdir -p .cursor/rules
ln -s ../AGENTS.md .cursor/rules/dengbao-agent.md
```

### Codex CLI

```bash
codex --system "$(cat AGENTS.md)"
```

### ChatGPT / DeepSeek / Kimi 等

直接将 `AGENTS.md` 的全部内容粘贴到 System Prompt / 自定义指令中。

### Hermes Agent

```bash
cp skills/dengbao-personality-framework.md ~/.hermes/profiles/default/skills/dengbao-personality/
```

---

## 包含什么

| 类别 | 内容 |
|------|------|
| 📖 国家标准 | GB/T 22239 / 28448 / 22240 / 25070 全部现行 |
| 📜 法律法规 | 网络安全法、数据安全法、个保法、关基条例 |
| 🆕 最新政策 | 公网安〔2025〕1846号 24项释疑 |
| 📐 定级矩阵 | 受侵害客体 × 侵害程度 × 5级 |
| 📊 安全类 | 10 大安全类 + 各等级要求项数 |
| ⚠️ 重大风险 | 32 项触发项（A-01~A-32）速查 |
| 🔑 一票否决 | 三级系统 5 个关键控制点 |
| 📋 结论标准 | 2025 版：符合 ≥ 90% / 基本符合 ≥ 60% / 不符合 < 60% |
| 🌟 第五级 | 定义、备案机关、测评要求 |

---

## 验证

加载后输入以下任一句：

| 输入 | 预期 |
|------|------|
| "帮我定个级" | 引导提供系统信息 → 按定级矩阵分析 |
| "Redis 没设密码" | 匹配 A-18 重大风险 → 输出说明 |
| "三级系统多少项" | "58项（技术34 + 管理24）" |
| "第五级找谁备案" | "省级公安机关网安部门" |
| "重大风险三原则" | "相关性、严重性、高发性" |

---

## 目录结构

```
等保DJCP-AI-Agent/
├── AGENTS.md                # 主文件（社区标准格式）
├── skills/
│   └── dengbao-personality-framework.md  # Hermes 兼容格式
├── README.md
└── LICENSE
```

---

## License

MIT License  
Copyright (C) 2026 lingfengz
