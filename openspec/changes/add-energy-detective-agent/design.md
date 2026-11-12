# Design

> 由組內 repo 的 [DESIGN.md](https://github.com/benson103081/energy-detective-agent/blob/main/DESIGN.md) 轉換。動機見 `proposal.md`（Why），行為契約見 `specs/`。完整 Mermaid 流程圖與 Pseudocode（Algorithm 1，44 行）保留在 [DESIGN.md](https://github.com/benson103081/energy-detective-agent/blob/main/DESIGN.md)。

## Context

- 資料只有 BDG2 建築總表、場址天氣與有限 metadata（2016–2017 逐時），沒有設備狀態、工單或故障真值。
- BDG2 raw 已初步整理；cleaned 會移除離群與零值，因此主分析用 raw。[D3][D6]
- Proposal 階段，`src/` 尚未建立；本 design 定義實作方式，供 `/opsx:apply` 依 tasks 實作。

## Goals / Non-Goals

**Goals:**
- 八模組受限 Agent：① 需求解析 → ② 受限規劃 → ③ CSV 重播 → ④ 品質檢查 → ⑤ 基準與異常 → ⑥ 領域規則 → ⑦ 證據報告 → ⑧ 離線評估。
- 數值全由 Python 計算，LLM 只做解析、規劃與文字；每個數字可回連 evidence_id。
- 單棟建築、時間切分、可重現的最低版本。

**Non-Goals:**
- 不判定設備故障根因，不估算已實現節電量。
- 不做即時 IoT 串接、多棟泛化、天氣迴歸模型、通用時序模型微調、向量資料庫。

## Decisions

### D1. 架構：「LLM 協調、工具計算、規則約束」
LLM 產生結構化計畫，控制器驗證白名單、依賴與預算後才執行工具。
- 替代方案：讓 LLM 直接讀資料回答 → 會編造數值，已被 A4 觀察到查詢錯誤與遺漏，不採用。
- 替代方案：完全固定流程 → 保留為對照組 B（固定流程＋LLM），以評估 Agent 規劃是否帶來增益。

### D2. 五個核心思維 → 設計決策對應（Deep Research Q1）

| # | 核心思維 | 設計決策 | 對應 spec |
|---|---|---|---|
| 1 | 「正常」必須有條件，不能用全期平均 | B1 星期×小時基準；天氣作為背景 | anomaly-analysis、domain-rule-reasoning |
| 2 | 先辨識資料問題，再討論營運異常 | 品質檢查是必要依賴；不補零；資料不足即停止 | meter-data-quality |
| 3 | 統計偏離 ≠ 設備根因 | 輸出待查假說與尚缺證據；禁止根因定論 | domain-rule-reasoning |
| 4 | 分析價值要經過管理者行動 | 合併連續點為事件；每則附下一步查核 | anomaly-analysis、evidence-reporting |
| 5 | 公開基準可重現，但不保證跨建築有效 | 單棟時間切分；跨建築列為限制 | anomaly-analysis |

### D3. 三個專業爭論 → 取捨決策（Deep Research Q2）

| # | 觀點 A ↔ 觀點 B | 本專題取捨 | 理由 |
|---|---|---|---|
| T1 | 積極清理 ↔ 保留可疑原值 | 用 raw＋品質旗標分流；cleaned 只做選配敏感度分析 | cleaned 會移除欲偵測的現象 [D6] |
| T2 | 通用預訓練模型 ↔ 目標建築調整 | 逐棟中位數 B0/B1 作可重現基準；不做微調 | 短上下文零樣本表現落後，且微調成本高 [D9] |
| T3 | 以偏離量優先篩選 ↔ 先完成節能量測驗證 | 只回報「相對基準偏離量」，不稱節電量 | 整棟總表無法分離措施成效 [D7][D8] |

### D4. 統計方法
殘差 `r_t = y_t − baseline_t`；門檻＝校準期 |r| 第 99 百分位；只在測試期標記；同方向相鄰點合併，遇缺值中斷。選中位數而非平均或深度模型：對離群穩健、可解釋、最低依賴。

### D5. 安全邊界
每次執行最多 10 次工具呼叫、1 次 I/O 重試、1 次計畫修正；報告數值驗證失敗即降級為固定模板。使用者輸入與資料文字一律視為資料，不得覆寫白名單或設定。

### D6. 實作對應

| 檔案 | 模組 | Pseudocode 行 |
|---|---|---|
| `config/experiment.json` | ①② | 01–09 |
| `src/agent.py` | ①②⑦ | 05–09、32–39 |
| `src/data.py` | ③ | 03、10–13 |
| `src/quality.py` | ④ | 14–16 |
| `src/analysis.py` | ⑤ | 17–30 |
| `src/rules.py` | ⑥ | 32、35–37 |
| `src/evidence.py` | ⑦ | 30、38–42 |
| `src/evaluate.py` | ⑧ | — |
| `src/main.py` | ①–⑦ | CLI 入口 |

## Risks / Trade-offs

- [無故障真值，無法報 precision/recall] → 合成注入只測敏感度；人工審閱另計，明確標示來源。
- [中位數基準受假日、季節、營運變動影響] → 列為限制；季節漂移與天氣模型列為擴充。
- [LLM 語意錯置，ID 對但結論錯] → 結構化主張比對建築、期間、單位與值，加逐份人工審閱。
- [單棟結果無法外推] → 報告失敗案例；跨建築／臺灣適用性列為未驗證。
- [Agent 增益可能只來自文字表達] → 三組對照固定同一 B1 與工具，分離規劃與文字生成差異。

## Open Questions

- 最終選用哪一棟 BDG2 建築，以覆蓋率條件在資料盤點後決定（不影響 spec）。
- LLM 供應商與版本於實驗前登錄。
