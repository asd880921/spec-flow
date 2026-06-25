---
name: dev-builder
description: 依 dev-planner 產出的 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md` 逐次執行 Commit Checklist。每次被調用時只處理「下一個尚未完成的 commit item」，完成程式碼與必要測試後回報變更、列出應包含與排除的檔案、提供建議 commit message 與 `git add` / `git commit` 指令，然後停止等待使用者手動 commit。本 skill 不自行執行 `git commit`，也不會在程式修改後立即把 checklist 標記完成；只有在使用者明確告知已手動 commit 後，才將該 item 從 `[ ]` 更新為 `[x]`。當使用者要依既有實作計畫推進下一個 commit 時使用。
---

# dev-builder

依上游 `dev-planner` 產出的 `IMPLEMENTATION_PLAN.md`，**逐次**執行 Commit Checklist。

本 skill **不是一次完成整個開發目標的 skill**。每次被調用時，**只處理下一個尚未完成的 commit item**，完成後停止等待使用者手動 commit。

## 核心行為規範（最高優先，調用時必須遵守）

- **不得自行執行 `git commit`。** 完成程式修改後，僅提供建議的 `git add` / `git commit` 指令供使用者手動執行。
- **完成程式修改後，不得立即把 checklist 標記為完成。** 只有在使用者明確告知「已完成手動 commit」後，才能回到 `IMPLEMENTATION_PLAN.md`，將該 commit item 從 `[ ]` 更新為 `[x]`。
- **每次調用只處理一個未完成的 commit item。** 不得一次連續完成多個 commit。
- **不得自行重寫 commit plan。** 若發現計畫與實際程式碼不一致、commit 無法照規劃實作，或需要改變整體方向，必須**暫停並回報問題**，必要時要求使用者回到 `dev-planner` 或上游 `codebase-sa-sd-finalizer` 修正，再回來執行。

## 每次執行流程

1. 讀取 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md`。
2. 找出 Commit Checklist 中**下一個尚未完成（`[ ]`）的 commit item**。
3. 若該 commit 有詳細規劃檔案（`commits/NN-<slug>.md`），先讀取。
4. 說明本次 commit 的目的、預期變更檔案與邊界。
5. 依「TDD 規範」實作：涉及行為邏輯時測試先行（先寫測試、先看失敗、再最小實作通過）；其餘情況比照處理並說明原因。
6. 完成後回報本次變更。
7. 列出應包含與應排除的檔案。
8. 提供建議的 commit message。
9. 提供建議的 `git add` / `git commit` 指令。
10. 停止，等待使用者手動 commit。

## 執行原則

- 所有回報內容以繁體中文撰寫；技術名稱、檔案路徑、branch 名稱、issue 編號與程式碼識別符保持原始形式不翻譯。
- 變更程式碼前，先重新說明當前 commit 的目的、預期檔案與邊界，確認與計畫一致。
- 只實作當前 commit 範圍內的變更，不順手改動其他 commit 或計畫外的程式碼。
- 若實作過程中當前 commit 的細節有合理調整（仍在原 commit 範圍與目的內），同步更新該 commit 的詳細規劃檔案；但不得擴張或改變 commit 的整體方向。

## TDD 規範（commit 範圍內測試先行）

當前 commit 涉及任何**行為邏輯**（新功能、bug 修正、規則或邊界調整）時，於該 commit 範圍內採測試先行：

1. **先寫測試**：針對本次 commit 的目標行為，寫一個聚焦、命名清楚的測試，描述「應該發生什麼」，測真實程式碼而非 mock。
2. **先看它失敗**：執行測試，確認它**因「功能尚未實作」而失敗**，而非因語法或環境錯誤而失敗。此步驟不得跳過——沒看過它失敗，就不知道它測的是不是對的東西。
3. **最小實作通過**：寫剛好讓測試通過的程式碼，不超出本 commit 範圍、不順手加計畫外功能。
4. **確認全綠**：確認該測試通過，且既有測試維持通過、輸出乾淨。
5. **必要時整理（僅限本次新增的程式碼）**：僅在「本 commit 剛寫的程式碼」內、且測試保持綠燈時做最小整理（例如清掉本次引入的重複）。
   - **嚴格遵循現行架構、命名慣例與程式風格**，向既有程式碼看齊，不得引入新風格或新抽象。
   - **不得改動本次以外的既有程式碼**（不順手重構、不改周邊命名、不調整格式或註解）。
   - 預設不整理；可省略此步。若覺得需要較大範圍的重構，**不要在此進行**，依「暫停與回報」回到 `dev-planner` 另立 commit。

### 範圍與例外

- 測試與其對應的實作**屬於同一個 commit**，一起納入本輪變更，不另拆 commit（除非計畫已將測試獨立為一個 commit item）。
- 以下情況可不適用，且**需在回報中明確說明原因**：純設定檔、無邏輯的樣板/產生碼、純文件或一次性拋棄式變更。
- 與本 skill 既有規範一致：仍是**一輪一個 commit**、**不自行 commit**、**不擴張 commit 範圍**。若為了測試先行而需改變 commit 拆分或方向，依「暫停與回報」處理，回到 `dev-planner`。

## Commit 規則

沿用計畫中的 conventional commit 格式。

- 若有 issue 編號，每則 commit message 都必須包含 `#<issue_number>`。
- 允許的 type 僅包含 `feat`、`fix`、`refactor`、`chore`、`test`。
- Commit type prefix 保持英文，描述以繁體中文撰寫。

## 輸出格式

完成當前 commit 範圍的程式修改後，回應格式如下：

```text
Current commit completed:
- Commit message: feat: #<issue_number> 某個 commit 標題
- Files to include:
  - path/to/file
  - path/to/file
- Files to exclude:
  - path/to/other-file
- Pre-commit checks:
  - 建議先確認測試結果或 diff 邊界

Suggested commands:
git add path/to/file path/to/file
git commit -m "feat: #<issue_number> 某個 commit 標題"
```

若無 issue 編號，使用相同格式但省略 issue 參照。

## 標記完成與繼續

- 完成程式修改後**停止**，等待使用者手動執行 commit。**此時不更新 checklist。**
- 當使用者明確告知「已完成手動 commit」後，才讀取 `IMPLEMENTATION_PLAN.md`，將該 commit item 從 `[ ]` 更新為 `[x]`，並停止。
- 下一次使用者再調用 `dev-builder` 時，才繼續處理下一個未完成的 commit item。

## 暫停與回報

若發現以下任一情況，**暫停並回報，不自行重寫 commit plan**：
- 計畫與實際程式碼不一致。
- 當前 commit 無法照規劃實作。
- 需要改變整體實作方向或重新拆分 commit。

回報時清楚說明問題所在與影響，並建議使用者回到 `dev-planner`（調整 commit plan）或上游 `codebase-sa-sd-finalizer`（修正 SA + SD 文件）處理，待修正後再回來執行。

## 下一步引導（純提示，不主動調用）

主動告知使用者後續選項，但**不自行調用**下一個 skill，也不自行 commit，由使用者決定與調用：
- **順流下一步**：本輪標記 `[x]` 後，還有未完成 commit → 再次調用 `dev-builder`；全部完成 → 用 `dev-acceptance-verifier` 驗收。
- **回頭修正**：commit 級 → `dev-planner`；SD 設計級 → `codebase-sa-sd-finalizer`；需求／模組分解級 → `codebase-sa-builder`（母文件）。

完整流程見 `WORKFLOW.md`。
