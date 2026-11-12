# I. Introduction（Refactored）

> DIC3 Project Refactor 步驟 4。依 A2Z SOP 的 G–L 重組，同時滿足作業規範的 A–F（Attention／Background／Problem／Limitations／Innovation／Contributions）。引用編號同組內 repo [proposal_source.md §6](https://github.com/benson103081/energy-detective-agent/blob/main/research/proposal_source.md)。

| A2Z | 規範 A–F | 本節段落 |
|---|---|---|
| G. General | A. Attention Getter、B. Background | 1.1 |
| H. However | C. Problem | 1.2 |
| I. In literature | — | 1.3 |
| J. Judgement | D. Existing Limitations | 1.4 |
| K. Keypoint | E. Innovation、F. Contributions | 1.5 |
| L. List | — | 1.6 |

## 1.1 General：建築用電與能源管理現況

IEA《Global Energy Review 2026》估計，2025 年全球用電需求增加約 3%，建築部門占當年用電**增量**近 45%（不是總量占比）[D1]；《Electricity 2026》預測 2025–2030 年建築將占新增用電需求的 49%（預測值）[D2]。用電成長使建築營運分析更重要，但全球統計不能證明個別建築在浪費。

實務上，能源管理資訊系統（EMIS）把量測資料整理成營運資訊。LBNL Smart Energy Analytics Campaign 涵蓋 104 個組織、6,500 棟建築，顯示資料分析與營運改善流程有實務價值 [D7]。公開資料方面，Building Data Genome 2（BDG2）提供 19 個場址、1,636 棟非住宅建築、3,053 個各類計量表的 2016–2017 年逐時資料 [D3]，讓方法能在共同資料上重現。本專題的目標使用者是校園或辦公建築的設施與能源管理人員（情境設定，尚未經訪談驗證）。

## 1.2 However：從「看到異常」到「知道下一步」的落差

管理者看到「週末用電變高」時，還要判斷比較期間是否合理、氣溫是否不同、資料是否缺漏，以及應該查營運排程還是電表。從圖表或異常分數走到可執行的調查，需要領域判讀。

**研究問題：** 只有建築總表、天氣與有限建築資訊時，結合分析工具、明示規則與 LLM，能否比固定報表或「固定流程＋LLM」產生數值正確、證據完整且不過度推論的調查建議？這是待驗證假設，不預設 Agent 較佳。

## 1.3 In literature：現有解法的三個方向

1. **統計與預測基準**：以歷史基準或預測殘差找偏離；DOE M&V 要求以天氣等變數調整基準 [D8]。
2. **LLM 輔助異常偵測**：以 LLM 生成可執行規則（Argos [A2]），或以多模態 LLM 複核候選異常以降低誤報 [A3]；Time-RA 進一步要求輸出異常類別與文字說明 [A5]。
3. **建築資料 LLM Agent**：ReAct 交替推理、行動與觀察 [A1]；Mulayim 等人以 GPT-4o 透過 Brick 模型查詢並分析 BAS 時序 [A4]。

## 1.4 Judgement：既有方法的限制

| 方法 | 優點 | 本問題下的限制 |
|---|---|---|
| 固定門檻／統計基準 | 可重現、便宜 | 無法區分天氣、營運與資料問題；不提供查核建議 [D8] |
| 資料清理後偵測 | 基準較乾淨 | BDG2 cleaned 會移除部分離群與零值，可能刪掉欲偵測的現象 [D3][D6] |
| LLM 規則／複核 | 可解釋、可降低誤報 | 需要標註或真值；誤報降低與漏報有取捨；非建築領域 [A2][A3] |
| 建築 LLM Agent | 能查詢與分析時序 | 有查詢錯誤、程式錯誤與遺漏感測器；依賴預建語意模型 [A4] |
| 直接把總表交給 LLM | 互動自然 | 可能編造數值與根因；總表沒有設備根因真值 |

此外，異常電量不等於可實現節電：實際節電需要改善前後量測與合理基準 [D8]。

## 1.5 Keypoint：本專題的設計精神、創新與貢獻

**設計精神：LLM 協調、工具計算、規則約束。** 本專題的創新是「可追溯的調查流程」，不是新的異常偵測演算法或全球首創：

1. **受限工具 Agent**：LLM 只能從六個白名單工具與允許參數中規劃，並在 10 次呼叫預算內，依觀察結果做一次下一步決策。
2. **先品質、後異常**：品質檢查是必要前置；不補零、不插補，資料不足時明確停止。
3. **條件式領域規則**：每則建議固定為「觀察事實 → 比較基準 → 可能解釋 → 尚缺證據 → 下一步查核」，規則是可撤回的假設。
4. **證據驗證報告**：每個數字以 evidence_id 回連，比對建築、期間、單位與值；驗證失敗時降級為固定模板。

**預期貢獻**（本階段尚未實作或證明）：
- C1：以 BDG2 歷史重播驗證工具使用與調查流程可行性的原型。
- C2：保留品質旗標、來源與假設的證據格式。
- C3：固定報表、固定流程＋LLM、受限 Agent 的三組公平比較，分別檢查數值正確、證據覆蓋與無根據根因斷言。

## 1.6 List：文件結構

第 II 節整理 AI 技術與建築能源領域的相關研究並歸納 Research Gap；第 III 節為 Deep Research（5 cores、3 trade-offs、10 details）；第 IV 節為 Proposed Scheme（八模組架構、流程圖、Algorithm 1）；第 V 節為評估協議；第 VI 節為參考文獻。
