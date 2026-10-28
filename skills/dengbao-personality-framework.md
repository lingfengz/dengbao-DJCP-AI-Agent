---
name: dengbao-personality-framework
description: "Use when defining or activating the personality and knowledge framework for a dengbao (等级保护) domain expert AI Agent. Standalone — no other skill required."
version: 2.3.0
author: lingfengz
license: MIT
metadata:
  hermes:
    tags: [等保, 等级保护, personality, 人格, memory, 记忆, djcp, security, compliance]
---

# 等保 DJCP Expert

This file defines the personality, core knowledge, and behavioral framework for a 等级保护（DJCP）AI Agent. See the primary format file (AGENTS.md at repo root) for the full knowledge base.

## Core Knowledge (2025 baseline + 2026 laws/standards — updated 2026-10-28 / v2.3)

- Standards: GB/T 22239-2019, GB/T 28448-2019, GB/T 28449-2018, GB/T 22240-2020, GB/T 25070-2019, GA/T 2182-2024 (Level-5 assessment reference)
- Policy documents: 公网安〔2025〕1001号 (scoring abolished, re-filing), 公网安〔2025〕1846号 (24-item Q&A), 公网安〔2025〕2391号 (high-risk determination guideline 2025, 81 cases: 26 removed / 40 added / 6 scope-adjusted)
- Major risk hazards (重大风险隐患): 33 trigger items — Appendix G of report template 2025 (26 mapped to assessment units: 17 general + 3 cloud + 5 ICS + 1 IoT; 7 comprehensive incl. penetration testing, internet exposure management). Principles: 相关性 / 严重性 / 高发性
- Assessment conclusion 2025: compliant = rate > 90% AND no major risk hazard; basically compliant = 60%~90% (regardless of high-risk issues); non-compliant = < 60%
- Assessment items (GB/T 28448-2019): Level 2 = 135, Level 3 = 211
- Level 5: filed at provincial PSB cyber division; assessed per Level-4 requirements + GA/T 2182-2024; annually; not equal to CII
- Filing rules: certificate valid 3 years (from 2025-01-01, +1 year after assessment); all Level-2+ systems re-filed with 2025 templates; cross-province systems filed at provincial level
- 2026 laws: 网络安全法 2025 revision (eff. 2026-01-01, penalties up to ¥10M + CII procurement 1-10x fine, AI clauses); 网络数据安全风险评估办法 (eff. 2026-08-20, assessor accreditation + major-risk notification); 公安机关网络空间安全监督检查办法 (MPS Order 176, **in force since 2026-10-01**: annual on-site check for Level 3+, repeals 2018 Order 151)。《网络安全等级保护条例》: still listed as "预备制定" in 国务院 2026 年度立法工作计划 (国办发〔2026〕14号, 2026-05-08), not yet issued —— the older "2025 年度计划" wording is outdated
- 2026 GA/T standards (all in force): 1390.6~9-2025 & 2347/2348-2025 (edge computing/big data/IPv6/blockchain/cloud/5G, 2026-02-01); GA/T 2380/2381-2026 (data security basic reqs + assessor org capability, 2026-06-01); GA/T 2394/2395-2026 (data security assessment reqs + process guide, 2026-07-01); GA/T 2396~2399-2026 KII series (2026-07-01: active defense implementation / critical business-chain vulnerability analysis & handling / monitoring & warning / incident handling)
- Data security track (GA/T 2380-2026): within one security level, systems split into general vs **important-data processing systems**; important-data systems add enhanced requirements on top of the general baseline — full lifecycle encryption & integrity, audit over all operation types, tamper-proof tagging of important data, annual data-security assessment with reports retained ≥3 years, end-to-end data supply-chain control. Assessment records for data-security units should be kept separate from GB/T 28448-2019 general items. **Evidence level: these enhancement points come from public interpretations and have NOT been verified against the standard text — confirm with the official document before any formal verdict.**
- GB/T 36959-2026《网络安全技术 网络安全等级保护测评机构能力要求和评估规范》(note the prefix change from 「信息安全技术」 to 「网络安全技术」): published 2026-05-25, replaces 36959-2018 effective 2026-12-01
- Grading template 2025: matrix differs from GB/T 22240-2020 (citizen/legal-person row → L1/2/2; national-security-or-regional-security/public-interest row → L4/5/5); mandatory data-level rule: important data ≥ Level 3, core data ≥ Level 4

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
