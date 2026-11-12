# Spec Delta

## Purpose

確保調查報告中的每個數值與主張都能回連到唯讀證據紀錄，LLM 文字無法通過驗證時以固定模板降級，讓管理者可逐項核對。

## ADDED Requirements

### Requirement: 證據登錄
每個工具結果 SHALL 登錄為唯讀證據，至少含 `evidence_id`、工具名、參數、資料版本、建築／電表、期間、計算方法、數值與單位、原始定位與品質警告。

#### Scenario: 分析完成
- **WHEN** `analyze_energy` 回傳事件
- **THEN** 每個事件都有對應 evidence_id，報告階段只能引用已凍結的證據

### Requirement: 數值與引用驗證
報告中的每個事實陳述 SHALL 以 evidence_id 與欄位指向證據；系統 MUST 比對建築、期間、單位與數值，而非只檢查 ID 存在。

#### Scenario: 數值不符
- **WHEN** LLM 報告寫「偏離 300 kWh」但證據為 240 kWh
- **THEN** 驗證失敗，系統改用固定模板產生報告

### Requirement: 報告內容與限制
報告 SHALL 以繁體中文呈現資料期間、事件、基準、偏離量、適用規則、尚缺證據、下一步查核與限制；資料不足或工具失敗時 MUST 明示，允許沒有異常結論。

#### Scenario: 資料不足
- **WHEN** 品質檢查判定覆蓋不足
- **THEN** 報告說明不足原因與品質證據，不列任何異常事件

### Requirement: 可重現輸出
每次執行 SHALL 保存 `report.md`、`evidence.json`、`tool_log.json`、`metrics.json`，含資料版本、參數與模型版本。

#### Scenario: 執行失敗
- **WHEN** 執行中途因錯誤停止
- **THEN** 系統仍保存已取得的證據與執行紀錄
