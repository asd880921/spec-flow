# 資料設計檢查清單

當需求影響 persisted data、schema、SQL、reporting、storage、同步、import/export 或資料生命週期時，讀取本文件，並在混合文件的後半段（SD 區）做出明確的資料設計決策。

## 必要決策

明確評估設計是否需要以下任一項：
- 無 schema 變更
- 在現有資料表新增欄位
- 新增資料表
- join 或 mapping 資料表
- lookup 或 config 資料表
- Enum 或常數異動
- Seed/config 資料異動
- Schema/資料交付產出物、資料修正、backfill 或 rollback script
- 僅查詢或業務邏輯層異動

## 必要說明

資料設計章節必須包含：
- 選定的方案
- 涉及的現有資料表（與 SA 描述及程式碼現況對照）
- 提案的新增或異動資料表與欄位（若有）
- 提案的 enum、lookup 或 config 異動（若有）
- 已知的關鍵關聯、唯一性規則、索引與生命週期規則
- 第一版必要索引、可延後的索引候選，以及排除推測性或非必要索引的說明
- 資料所有權與 source of truth
- 衍生值或歷史值的計算方式
- Schema/資料交付方式、資料修正、backfill 或 rollback 影響
- Reporting、export 或稽核影響（若相關）
- 未解決的設計問題（若有，登錄「待釐清事項」）

## 決策呈現格式

優先使用簡潔表格：

```text
| 來源 | 資料項目 | 設計決策 |
| --- | --- | --- |
| Code-confirmed / Inferred | <資料表 / 欄位 / enum / lookup / 部署產出物 / 查詢> | <決策與依據> |
```

實際 persisted 資料表使用 TableSchema 格式（見 `hybrid-document-template.md`）。

## 需向使用者提問的時機

當 SA、程式碼與使用者補充資料均無法確認以下任一項時，依「核心行為規範」於定稿前向使用者提問：
- 資料應儲存還是即時推導
- 現有資料表是否足夠權威
- 歷史資料是否需要資料修正、backfill 或 rollback 處理
- 新實體是否有自己的生命週期
- 是否需要一對多或多對多關聯
- 資料屬於 configuration、transaction、audit data 還是暫存狀態
- 刪除、保留、回復或 rollback 行為是否重要
- 新的 enum 值應沿用現有 enum 還是需要新的 enum/lookup

若不確定性影響低或可留待後續審查，記錄在「待釐清事項」而非阻擋文件產生。

## Release 交付

不要假設有 migration framework。當需要 schema 變更、seed/config 資料、資料修正、backfill 或 rollback 時，明確說明交付方式，例如 `.SQL` 或其他公司核准的部署產出物。若 SA、專案或使用者未指定交付方式，記錄在「待釐清事項」，不要自行猜測。
