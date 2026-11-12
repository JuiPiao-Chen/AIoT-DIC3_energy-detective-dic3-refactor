# Spec Delta

## Purpose

將管理者的自然語言調查需求轉為經驗證的範圍與受限工具計畫，並控制工具白名單、依賴順序、呼叫預算與重試，避免 Agent 越界或無限循環。

## ADDED Requirements

### Requirement: 需求解析與範圍驗證
系統 SHALL 將使用者問題轉為 `building_id`、`start`、`end`（半開區間）與調查目標，並對照資料清單確認建築與期間可用。缺少或不可用時 MUST 回覆缺少的欄位與可用範圍，不得自行換棟或擴大期間。

#### Scenario: 需求完整
- **WHEN** 使用者問「請調查建築 X 在測試期的用電異常」且 X 存在於資料清單
- **THEN** 系統產生含建築與期間的結構化需求並進入規劃

#### Scenario: 缺少建築
- **WHEN** 使用者問「最近有哪些用電異常？」未指定建築
- **THEN** 系統回覆 NEED_INPUT 並列出可用建築與期間，不執行分析

### Requirement: 工具白名單與參數限制
系統 SHALL 只允許 `inspect_dataset`、`load_window`、`check_quality`、`analyze_energy`、`apply_domain_rules`、`build_report` 六個工具；MUST 拒絕未知工具、任意路徑、URL、SQL 與程式碼，且 LLM 不得修改時間切分與統計門檻。

#### Scenario: 未知工具
- **WHEN** LLM 計畫中出現不在白名單的工具或越界參數
- **THEN** 控制器拒絕該計畫，只依驗證錯誤修正一次；仍不合法則以失敗狀態結束並保存紀錄

### Requirement: 呼叫預算與重試
每次執行 SHALL 最多 10 次工具呼叫（含最多一次同參數的暫時性 I/O 重試），計畫格式最多修正一次；欄位缺漏、檔案不存在、品質不足與演算法錯誤 MUST NOT 重試。

#### Scenario: 暫時性讀取失敗
- **WHEN** `load_window` 第一次回傳暫時性 I/O 錯誤
- **THEN** 系統以相同參數重試一次；再失敗則回傳失敗與已取得證據

#### Scenario: 預算用盡
- **WHEN** 工具呼叫次數達 10 次
- **THEN** 系統停止執行並輸出帶紀錄的受限報告

### Requirement: 依觀察調整下一步
分析完成後，Agent SHALL 依事件與品質觀察做一次決策：呼叫適用的 `apply_domain_rules` 或直接 `build_report`，並記錄選擇理由與被跳過的規則。

#### Scenario: 無事件
- **WHEN** 分析結果沒有候選事件
- **THEN** Agent 直接產生報告並記錄跳過規則的理由
