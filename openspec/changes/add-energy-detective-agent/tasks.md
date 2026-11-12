# Tasks

## 1. 資料與設定

- [ ] 1.1 下載 BDG2 raw electricity、weather、metadata，核對欄位、單位、授權與時區，記錄於 `data/README.md`；驗證：文件列出來源 URL 與檔案版本
- [ ] 1.2 依覆蓋率條件選定單棟建築與連續 12 週期間，寫入 `config/experiment.json`（8/2/2 週切分、q=0.99、預算 10）；驗證：設定檔可被 pydantic 載入
- [ ] 1.3 建立 `requirements.txt`（鎖定 pandas、numpy、pydantic、LLM SDK 版本）；驗證：乾淨 venv 安裝成功

## 2. 資料重播與品質（meter-data-quality）

- [ ] 2.1 實作 `src/data.py`：寬表轉長表、場址對照、`replay_until` 限制、原始定位；驗證：單元測試確認截止點後資料不可見
- [ ] 2.2 實作 `src/quality.py`：重複／衝突、缺值、負值、時距、單位、天氣對齊與有效遮罩；驗證：測試涵蓋衝突排除、缺值不補零、有效率 <90% 回傳資料不足

## 3. 基準與異常（anomaly-analysis）

- [ ] 3.1 實作時間切分與順序斷言；驗證：錯誤順序的設定回傳 error
- [ ] 3.2 實作 B0/B1 基準、校準門檻與測試期標記；驗證：合成小資料上 B0/B1 MAE 與手算一致
- [ ] 3.3 實作同方向事件合併（缺值中斷）；驗證：用 DESIGN.md 的 240 kWh 示例測試，第三小時缺值時拆成兩個事件

## 4. 領域規則（domain-rule-reasoning）

- [ ] 4.1 實作 `src/rules.py` 四條規則，含 rule_id、版本、前置條件與停用原因；驗證：無作息時 `schedule_check` 停用、無天氣時 `weather_context` 停用
- [ ] 4.2 實作假說輸出格式與禁止根因檢查；驗證：「哪台空調壞了」案例輸出資料不足說明

## 5. 證據與報告（evidence-reporting）

- [ ] 5.1 實作 `src/evidence.py` 證據登錄與凍結；驗證：每個事件都有 evidence_id
- [ ] 5.2 實作數值／單位／期間比對與固定模板降級；驗證：竄改報告數值時驗證失敗並改用模板

## 6. Agent 與控制器（investigation-planning）

- [ ] 6.1 實作 `src/agent.py` 需求解析、結構化計畫、白名單與一次修正；驗證：未知工具案例被拒絕
- [ ] 6.2 實作 `src/main.py` 工具包裝器、10 次預算、一次 I/O 重試與執行紀錄；驗證：模擬 I/O 失敗只重試一次，輸出 `outputs/<run_id>/` 四個檔案

## 7. 評估與交付

- [ ] 7.1 實作 `src/evaluate.py`：B0/B1 比較、合成注入（尖峰／持續增加／零值各 10）、10 個固定案例 × 3 次、三組對照；驗證：產生 `metrics.json` 與結果表
- [ ] 7.2 更新 `DESIGN.md` 的 Results 與 Limitations、README 的 How to Run；驗證：依 README 指令可從零重現一次執行

## Workflow follow-up

- 實作與驗證完成後執行 `openspec archive add-energy-detective-agent`，將 specs 併入 `openspec/specs/`。
