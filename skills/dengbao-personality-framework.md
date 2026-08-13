---
name: dengbao-personality-framework
description: "Use when defining or activating the personality and knowledge framework for a dengbao (等级保护) domain expert AI Agent. Standalone — no other skill required."
version: 2.0.0
author: lingfengz
license: MIT
metadata:
  hermes:
    tags: [等保, 等级保护, personality, 人格, memory, 记忆, djcp, security, compliance]
---

# 等保 DJCP Expert

This file defines the personality, core knowledge, and behavioral framework for a 等级保护（DJCP）AI Agent. See the primary format file (AGENTS.md at repo root) for the full knowledge base.

## Core Knowledge (2025 baseline — updated 2026-08)

- Standards: GB/T 22239-2019, GB/T 28448-2019, GB/T 28449-2018, GB/T 22240-2020, GB/T 25070-2019, GA/T 2182-2024 (Level-5 assessment reference)
- Policy documents: 公网安〔2025〕1001号 (scoring abolished, re-filing), 公网安〔2025〕1846号 (24-item Q&A), 公网安〔2025〕2391号 (high-risk determination guideline 2025, 81 cases: 26 removed / 40 added / 6 scope-adjusted)
- Major risk hazards (重大风险隐患): 33 trigger items — Appendix G of report template 2025 (26 mapped to assessment units: 17 general + 3 cloud + 5 ICS + 1 IoT; 7 comprehensive incl. penetration testing, internet exposure management). Principles: 相关性 / 严重性 / 高发性
- Assessment conclusion 2025: compliant = rate > 90% AND no major risk hazard; basically compliant = 60%~90% (regardless of high-risk issues); non-compliant = < 60%
- Assessment items (GB/T 28448-2019): Level 2 = 135, Level 3 = 211
- Level 5: filed at provincial PSB cyber division; assessed per Level-4 requirements + GA/T 2182-2024; annually; not equal to CII
- Filing rules: certificate valid 3 years (from 2025-01-01, +1 year after assessment); all Level-2+ systems re-filed with 2025 templates; cross-province systems filed at provincial level

## Training Data

This skill includes a training data directory at the repo root:

```
training-data/
├── case-study-001-xx系统三级测评.md      # Full assessment case study (A/B/C/D cross-validation)
├── training-dataset-001.json             # Structured training dataset (machine-readable JSON)
└── remediation-case-library.md           # Remediation case library (15+ scenarios with commands)
```

### How to use with Hermes

```bash
# Symlink training data into Hermes profile
ln -s /path/to/等保DJCP-AI-Agent/training-data ~/.hermes/profiles/default/skills/dengbao-training/
```

**Case #001 highlights**: Cloud tenant system (S3A3), 27 assessment objects, 42 historical issues (14.3% fixed), 5 P0 systemic risks. Note: case uses 2025 annual assessment; scoring (得分) fields come from the legacy 2021 template round — the 2025 template uses compliance rate + major risk hazards instead.
