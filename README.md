# DIC3 Project Refactor：Energy Detective × OpenSpec

以 [OpenSpec](https://github.com/Fission-AI/OpenSpec)（Spec-Driven Development）重構期末專題提案 **Energy Detective：建築用電異常調查 AI Agent**（第25組）。

- 原始專題（組內共用 repo）：<https://github.com/benson103081/energy-detective-agent>
- 本 repo 只放 DIC3 作業的重構產出，不修改組內 repo。
- Proposal 階段：程式與實驗尚未實作，文件中沒有任何實測結果。

## 作業步驟對應

| 步驟 | 檔案 |
|---|---|
| 1. 安裝 OpenSpec、熟悉 Spec-Driven Development | [`openspec/config.yaml`](openspec/config.yaml)、`.claude/`（`/opsx:propose`、`/opsx:apply` 等指令） |
| 2. A–F、5 Cores、3 Trade-offs、10 Details | [docs/refactor/01-AtoF-5cores-3tradeoffs-10details.md](docs/refactor/01-AtoF-5cores-3tradeoffs-10details.md) |
| 3. DESIGN.md → OpenSpec workflow | [openspec/changes/add-energy-detective-agent/](openspec/changes/add-energy-detective-agent/) |
| 4. Introduction refactor | [docs/refactor/02-introduction.md](docs/refactor/02-introduction.md) |
| 5. Related Work refactor | [docs/refactor/03-related-work.md](docs/refactor/03-related-work.md) |

## OpenSpec 結構

```text
openspec/
├── config.yaml                         # schema: spec-driven＋專案限制
├── specs/                              # archive 後的正式規格（目前為空）
└── changes/add-energy-detective-agent/
    ├── proposal.md                     # Why（含 A–F）、What Changes、Capabilities
    ├── design.md                       # 架構決策、5 cores／3 trade-offs → 設計對應
    ├── tasks.md                        # 7 組可驗證的實作任務
    └── specs/
        ├── investigation-planning/     # 需求解析、白名單、預算與重試
        ├── meter-data-quality/         # BDG2 重播、品質檢查、資料不足
        ├── anomaly-analysis/           # 時間切分、B0/B1、門檻、事件合併
        ├── domain-rule-reasoning/      # 條件式規則、待查假說
        └── evidence-reporting/         # 證據登錄、引用驗證、模板降級
```

DESIGN.md 的八個模組（①–⑧）分別落在上面五個 capability，對應表見 `design.md` D6。

## 驗證

```bash
npm i -g @fission-ai/openspec@1.15.0
openspec validate add-energy-detective-agent --strict
openspec status --change add-energy-detective-agent
```

## Spec-Driven 流程

`/opsx:propose`（已完成：proposal → specs → design → tasks）→ `/opsx:apply`（期末實作）→ `openspec archive add-energy-detective-agent`（併入 `openspec/specs/`）
