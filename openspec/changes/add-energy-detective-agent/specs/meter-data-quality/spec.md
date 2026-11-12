# Spec Delta

## Purpose

以 BDG2 歷史 CSV 重播模擬 IoT 資料流，並在任何異常分析前先辨識資料問題，確保異常結論不是由缺值、重複或單位錯誤造成。

## ADDED Requirements

### Requirement: 歷史重播的可見時間限制
系統 SHALL 只提供 `replay_until` 之前的電表與天氣資料，並保留原始檔名與列／欄定位。

#### Scenario: 截止點之後的資料
- **WHEN** 請求期間超過 `replay_until`
- **THEN** 系統不讀取截止點之後的資料並在警告中註明

### Requirement: 品質檢查與有效遮罩
系統 SHALL 檢查時間格式、重複／衝突、缺值、時距、非有限值、負值、單位與天氣對齊，輸出品質摘要與可分析遮罩。系統 MUST NOT 對缺失量測補零或插補測試期用電。

#### Scenario: 數值衝突
- **WHEN** 同一時間戳出現兩個不同讀值
- **THEN** 該點標記為衝突並排除，記錄於品質證據

#### Scenario: 缺值
- **WHEN** 某小時讀值缺失
- **THEN** 該點保留為缺值，不補零，並在事件合併時視為中斷

### Requirement: 資料不足判定
系統 SHALL 在各分割段用電有效率低於 90%，或訓練期任一星期×小時組別少於 4 個有效觀測，或單位／時間軸無法確認時，回傳資料不足報告而非異常結論。

#### Scenario: 覆蓋不足
- **WHEN** 測試期有效率為 70%
- **THEN** 系統輸出 INSUFFICIENT_DATA 報告並附品質證據

### Requirement: 天氣缺漏降級
天氣無法同場址同時刻對齊時，系統 SHALL 繼續用電分析，但 MUST 停用天氣相關規則並在報告中標示限制。

#### Scenario: 無天氣資料
- **WHEN** 場址天氣缺漏
- **THEN** 系統完成用電分析並停用 `weather_context` 規則
