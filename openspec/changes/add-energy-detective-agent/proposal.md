# Proposal：Energy Detective 建築用電異常調查 AI Agent

> 本 change 由組內 repo [energy-detective-agent](https://github.com/benson103081/energy-detective-agent) 的 [DESIGN.md](https://github.com/benson103081/energy-detective-agent/blob/main/DESIGN.md) 轉換而來，作為 Spec-Driven Development 的起點。Proposal 階段，尚無實作與實驗結果。

## Why

建築用電持續成長，2025 年建築占全球用電增量近 45%（IEA, 2026）[D1]，但管理者面對電表異常時，仍須自行判斷基準是否合理、資料是否缺漏、是否受天氣影響，以及下一步該查什麼。單純異常分數無法區分天氣、營運變化與資料問題；通用 LLM 直接回答則可能編造數值或根因。需要一個「工具算數、規則約束、LLM 協調」的受限調查 Agent，讓每個結論都能回連可重跑的證據。

### Abstract A–F 摘要對應

| 段落 | 內容 |
|---|---|
| A. Attention Getter | 2025 年全球用電成長約 3%，建築占增量近 45%；2025–2030 年建築預估占新增需求 49%。[D1][D2] |
| B. But | 異常分數無法區分天氣、營運與資料問題；BDG2 cleaned 會移除部分離群與零值，也沒有現場根因真值；建築 LLM 工具研究有查詢錯誤與回答遺漏。[D3][D6][A4] |
| C. Cure | 提出 Energy Detective：以自然語言指定建築與期間，取得附數值、來源、適用規則與下一步查核事項的調查報告。 |
| D. Development | BDG2 歷史電表＋天氣＋metadata；Python 執行品質檢查、B0/B1 條件基準與事件分析；受限規劃 Agent 選工具；明示領域規則約束推論。 |
| E. Experiments | 時間切分歷史重播；B0 vs B1（MAE）、合成異常敏感度、10 個固定 Agent 案例 × 3 次、三組對照（固定報表／固定流程＋LLM／受限 Agent）。 |
| F. Findings | 預期產出可重現原型與附證據報告；本階段不宣稱提高準確率、證實故障或達成節電。 |

## What Changes

- 新增自然語言需求解析與受限規劃：轉成 `building_id`、`start`、`end`、調查目標，並以白名單工具、固定依賴與 10 次呼叫預算執行。
- 新增 BDG2 CSV 歷史重播與時序資料品質檢查（缺值、重複、負值、單位、時區、天氣對齊），不補零、不插補。
- 新增 B0（全域中位數）／B1（星期×小時中位數）基準、校準期 99 百分位門檻與同方向連續事件合併。
- 新增條件式領域規則（同時段比較、非使用時段、天氣背景、長段零值），輸出待查假說而非根因定論。
- 新增證據登錄與報告驗證：每個數值以 `evidence_id` 回連；驗證失敗時降級為固定模板。
- 新增離線評估流程（B0/B1、合成注入、10 案例、三組對照）。

## Capabilities

### New Capabilities
- `investigation-planning`：自然語言需求解析、範圍驗證、受限工具計畫、預算與重試控制。
- `meter-data-quality`：BDG2 歷史重播讀取與時序資料品質檢查、資料不足判定。
- `anomaly-analysis`：時間切分、B0/B1 基準、校準門檻、候選異常與事件合併。
- `domain-rule-reasoning`：條件式領域規則、適用前提與待查假說。
- `evidence-reporting`：證據登錄、數值與引用驗證、繁中調查報告與模板降級。

### Modified Capabilities
（無；專案尚無既有 spec）

## Impact

- 預定新增 `src/`（agent、data、quality、analysis、rules、evidence、evaluate、main）與 `config/experiment.json`；對應表見 `design.md`。
- 依賴：Python、pandas、numpy、pydantic、單一 LLM 供應商 SDK；不需向量資料庫、微調或雲端 IoT。
- 資料：BDG2 `data/meters/raw/electricity.csv`、`data/weather/weather.csv`、`data/metadata/metadata.csv`（遵守原授權，不再散布原始資料）。
- 文件：組內 repo 的 `DESIGN.md` 維持為人讀總覽；行為契約以本 change 的 specs 為準。
