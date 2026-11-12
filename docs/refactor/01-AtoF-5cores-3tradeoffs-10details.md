# Energy Detective：A–F、5 Cores、3 Trade-offs、10 Details

> DIC3 Project Refactor 步驟 2。內容濃縮自組內 repo 的 [research/proposal_source.md](https://github.com/benson103081/energy-detective-agent/blob/main/research/proposal_source.md)，對應 OpenSpec change `openspec/changes/add-energy-detective-agent/`。Proposal 階段，無實測結果。

## A–F（Abstract）

| | 一句話 | 證據 |
|---|---|---|
| **A** Attention | 2025 年建築占全球用電增量近 45%，2025–2030 預估占新增需求 49%，建築營運分析更重要。 | [D1][D2] |
| **B** But | 異常分數無法區分天氣、營運與資料問題；總表無根因真值；建築 LLM 工具會查錯、漏答。 | [D3][D6][A4] |
| **C** Cure | Energy Detective：自然語言指定建築與期間，回傳附數值、來源、規則與下一步查核的調查報告。 | — |
| **D** Development | BDG2 重播 → 品質檢查 → B0/B1 基準 → 事件 → 領域規則 → 證據驗證報告；LLM 只協調。 | [DESIGN.md](https://github.com/benson103081/energy-detective-agent/blob/main/DESIGN.md) |
| **E** Experiments | B0 vs B1 MAE；合成注入敏感度；10 案例 × 3 次；固定報表／固定流程＋LLM／受限 Agent 三組對照。 | [evaluation_protocol.md](https://github.com/benson103081/energy-detective-agent/blob/main/research/evaluation_protocol.md) |
| **F** Findings | 預期：可重現原型與附證據報告；不宣稱提升準確率、確認故障或節電。 | — |

## 5 Cores（五個核心思維）

| # | 核心思維 | 系統設計影響 |
|---|---|---|
| 1 | 「正常」必須有條件，不能只用全期平均 | B1 星期×小時基準、天氣背景 |
| 2 | 先辨識資料問題，再討論營運異常 | 品質檢查為必要前置；不補零 |
| 3 | 可解釋的統計偏離 ≠ 設備根因 | 只輸出待查假說，禁止根因定論 |
| 4 | 分析價值要經管理者行動才形成效益 | 事件合併＋下一步查核 |
| 5 | 公開基準可重現，但不保證跨建築有效 | 單棟時間切分；跨建築列限制 |

## 3 Trade-offs（三個專業爭論）

| # | 觀點 A ↔ 觀點 B | 共識 | 本專題選擇 | 未解問題 |
|---|---|---|---|---|
| 1 | 積極清理 ↔ 保留可疑原值 | 保留版本、旗標與處理理由 | raw＋品質分流 | 無現場紀錄時如何區分真故障與量測問題 |
| 2 | 通用預訓練模型 ↔ 目標建築調整 | 同資料、同範圍比較，連同成本一起報告 | 逐棟中位數，不微調 | 多少本地資料值得調整 |
| 3 | 以偏離量優先篩選 ↔ 先做節能量測驗證 | 偏離量、潛在機會、已驗證節電三者分開 | 只報「相對基準偏離量」 | 多少異常負載可在不影響服務下消除 |

## 10 Details（十個深度理解問題）

| # | 層次 | 問題 |
|---|---|---|
| 1 | Evidence | 沒有故障標籤時，如何建立可審核的事件評估？ |
| 2 | What-if | 氣溫、星期、季節改變後異常還成立嗎？ |
| 3 | Comparison | raw 與 cleaned 的候選事件差異如何影響結論？ |
| 4 | Failure case | 零值、缺值、低負載如何分流？ |
| 5 | Design decision | 缺營運排程時，Agent 何時應停止推論？ |
| 6 | Trade-off | 何時用逐棟基準，何時需要跨建築參照？ |
| 7 | Evidence | 如何檢查引用是否真正支持主張？ |
| 8 | Trade-off | 事件合併與門檻調整如何影響召回、誤報與查核負擔？ |
| 9 | Comparison | Agent 增益來自工具選擇、規則，還是只是表達方式？ |
| 10 | Real-world | 從「偏離基準」到「確定根因」與「已驗證節電」還缺什麼？ |

判讀要點見 [proposal_source.md §4 Q3](https://github.com/benson103081/energy-detective-agent/blob/main/research/proposal_source.md)。
