# Spec Delta

## Purpose

以明示、可撤回的建築能源領域規則，把統計事件轉成「觀察事實 → 比較基準 → 可能解釋 → 尚缺證據 → 下一步查核」的待查假說，不從總表推定設備根因。

## ADDED Requirements

### Requirement: 規則前置條件
每條規則（`historical_deviation`、`schedule_check`、`weather_context`、`zero_run_check`）SHALL 保存 rule_id、版本、來源與前置條件；前置條件不滿足時 MUST 停用該規則並記錄原因。

#### Scenario: 無作息資料
- **WHEN** 使用者未提供作息或當地時區未知
- **THEN** `schedule_check` 停用，系統不自行假設週末或夜間停工

### Requirement: 假說輸出格式
規則輸出 SHALL 依序包含品質限制、支持證據、缺少證據、可能解釋與查核建議；可能解釋 MUST 標示為待驗證假說。

#### Scenario: 夜間偏高
- **WHEN** 已提供作息且非使用時段用電高於 B1
- **THEN** 輸出「建議確認加班、排程與待機負載」，並列出尚缺設備狀態紀錄

### Requirement: 禁止無根據根因
系統 MUST NOT 從建築總表資料輸出設備故障定論或已實現節電量。

#### Scenario: 使用者要求根因
- **WHEN** 使用者問「是哪台空調壞了？」
- **THEN** 系統回覆總表資料不足以判定，並列出確認根因所需的設備或工單資料

### Requirement: 長段零值分流
連續至少 6 個有效逐時零值 SHALL 觸發 `zero_run_check`，並建議先核對資料傳輸、停用或供電紀錄，不得直接判定斷電。

#### Scenario: 連續零值
- **WHEN** 測試期出現連續 8 小時零值
- **THEN** 系統列出「資料傳輸／建築停用／供電」三種待查解釋
