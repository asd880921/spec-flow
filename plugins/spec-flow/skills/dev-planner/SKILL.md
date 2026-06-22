---
name: dev-planner
description: 根據上游 codebase-sa-sd-finalizer 產出的 SA + SD 混合文件（或其他任務內容、issue、規格），對照當前儲存庫狀態，規劃實作方向與 atomic commits，並將計畫持久化保存至 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md`。本 skill 只負責規劃 commit plan 與 Commit Checklist，不修改任何功能程式碼；實際逐一執行 commit 由下游 dev-builder 負責。當使用者希望在開始撰寫程式前，先依 SA + SD 文件取得具體 commit 計畫時使用。
---

# dev-planner

依上游 `codebase-sa-sd-finalizer` 產出的 **SA + SD 混合文件**（`原檔名-sd.md`），對照當前儲存庫狀態，規劃實作方向並拆分為 atomic commits，再把計畫持久化保存。

本 skill **只負責規劃**。它可以建立或更新計畫的 Markdown 檔案，但**不得修改任何功能程式碼**。實際逐一執行 Commit Checklist 由下游 `dev-builder` 負責。

優先的任務來源：
- 上游 `codebase-sa-sd-finalizer` 產出的 SA + SD 混合文件（`*-sd.md`）。
- 上游 `codebase-dev-doc-builder` 產出的輕量開發文件（`*-dev.md`，輕量路徑）。
- 提示中直接提供的任務內容。
- 透過 `gitlab-issue-fetch` 取得的 GitLab issue。
- 其他已提供的文件或任務描述。

本 skill 對「上游文件」的規範同時適用於 `*-sd.md` 與 `*-dev.md`：兩者都是計畫的主要事實來源，commit plan 不得建立在已知過期或矛盾的上游文件上。`*-dev.md` 為輕量路徑、無 SD 細節，規劃時就近依其「受影響範圍（現況與調整）」展開 commit。

## 核心行為規範（最高優先，調用時必須遵守）

**遇到無法判斷的情況必須立即暫停並向使用者提問，不得自行假設或繼續規劃。**

當規劃過程中出現以下任一情況時，必須**立即停下來**向使用者提問，取得明確回覆後才繼續：

- 上游 SA + SD 文件缺少足以拆分 commit 的必要資訊。
- 上游 SA + SD 文件內部矛盾。
- 上游 SA + SD 文件與現有程式碼不一致。
- 實作方向會影響 commit 拆分，但無法從文件與程式碼判斷。
- 系統邊界、模組責任、資料流、相依性或風險不清楚到無法安全規劃 commit。

**若使用者的回答與上游 SA + SD 文件不一致：必須先同步更新上游 SA + SD 文件，再繼續產出 commit plan。** 不得讓 commit plan 建立在已知過期或不一致的上游文件上。若無法直接更新上游文件，停下來請使用者先回到 `codebase-sa-sd-finalizer` 修正，再回來規劃。

## 輸入

請提供下列其中一項：
- 上游 SA + SD 混合文件（`*-sd.md`）或輕量開發文件（`*-dev.md`）的路徑（最主要來源）。
- 直接在提示中提供完整的任務內容。
- `project_id`（數字）與 `issue_number`（數字）。
- 其他足以用於實作規劃的文件或任務內容。

## 工作流程

1. 取得並通讀上游 SA + SD 混合文件。文件不完整且提供了 `project_id` 與 `issue_number` 時，先呼叫 `gitlab-issue-fetch` 補齊脈絡。
2. 在規劃前先讀取當前專案狀態，對照 SA + SD 文件確認受影響的模組、資料流、介面與既有實作。
3. 規劃過程中一旦觸發「核心行為規範」所列情況，立即停下來向使用者提問；若回答與上游文件不一致，先更新上游 SA + SD 文件再繼續。
4. 偵測當前的 Git 分支名稱，並轉換為安全的資料夾名稱（保留 branch 語意）。
5. 將主要計畫持久化儲存至 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md`。
6. 產出具體的實作計畫，並拆分為實務上可操作的 atomic commits。
7. 僅在 commit 規模夠大、確實需要時，才建立個別 commit 的詳細規劃檔案。
8. 呈現簡潔摘要與已儲存的檔案路徑。
9. 呈現完整 commit 計畫後停止；後續由 `dev-builder` 逐一執行。

## 規劃原則

計畫應基於：
- 上游 SA + SD 混合文件已確立的架構與系統設計結論。
- 相關討論或輔助背景資訊（如有）。
- 當前 codebase 的實際狀態。

要求：
- 所有規劃內容以繁體中文撰寫。
- 技術名稱、檔案路徑、branch 名稱、issue 編號及程式碼識別符保持原始形式不翻譯。
- 不產出抽象建議，每個 commit 須對應到實際的檔案、職責與程式碼變更。
- 提供足夠的實作方向，使 `dev-builder` 可僅憑計畫檔案安全地繼續執行。
- 不得回頭重做 SA / SD 的需求理解或架構判斷；計畫承接上游結論，僅展開為 commit 層級的實作步驟。

## Commit 拆分原則

Commit Checklist 指「commit 任務清單」，每個 item 原則上對應**一個 atomic commit**，而非一般 TODO checklist。

- 每個 commit 聚焦單一且連貫的變更，可獨立理解。
- 避免將性質相同的細微修改拆分成無意義的獨立 commits（避免過度細碎）。
- 避免單一 commit 過大到無法在一次 `dev-builder` 執行中安全完成。
- 當檔案屬於同一條連貫路徑時，優先將一個端對端的功能切片合併為單一 commit。
- 涉及 migration、rollback、相容性風險或跨模組時，於該 commit 明確標示風險與必要細節。

## Commit 規則

使用 conventional commit 格式。

- 若有 issue 編號，每則 commit message 都必須包含 `#<issue_number>`。
- 允許的 type 僅包含 `feat`、`fix`、`refactor`、`chore`、`test`。
- Commit type prefix 保持英文，描述以繁體中文撰寫。

範例：

```text
feat: #<issue_number> 新增訂單匯出功能
- path/to/file: 說明該檔案預計變更內容
```

## 計畫檔案

主要計畫一律寫入 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md`。

規則：
- 將 branch 名稱轉換為安全的資料夾名稱，同時保留 branch 語意。
- 主要計畫保持簡潔、高訊號密度。
- 同時保留 Commit Checklist 與每個 commit 的目的、預期變更檔案、實作方向、邊界與完成條件。
- Commit Checklist 中所有 item 初始狀態皆為未完成 `[ ]`；標記完成是 `dev-builder` 在使用者確認手動 commit 後才進行的動作，dev-planner 不預先標記。
- 若後續因使用者回饋或上游文件更新而調整計畫，更新同一份主要計畫檔案。

使用以下結構：

```md
# 實作計畫

- 分支：<branch-name>
- 任務來源 / 上游文件：<SA + SD 文件路徑、issue number 或 prompt 摘要>

## Commit Checklist
- [ ] Commit 1
- [ ] Commit 2

## Commit 1：<commit message>
- 目的：...
- 變更檔案：
  - path/to/file: ...
- 實作方向：...
- 邊界 / 排除範圍：...
- 風險：<migration / rollback / 相容性 / 跨模組；若無填「無」>
- 完成條件：...
- 詳細規劃：commits/01-<slug>.md   <僅複雜 commit 才有>
```

僅在符合以下至少一項條件時，才建立 commit 詳細規劃檔案：
- 該 commit 預計涉及超過 6 個檔案。
- 該 commit 橫跨超過 1 個子系統或模組範圍。
- 該 commit 包含 migration、rollback 或相容性風險。
- 該 commit 的內容無法在 `IMPLEMENTATION_PLAN.md` 的短篇章節中清楚描述。

當需要時，將詳細規劃檔案寫入 `.ai/<sanitized-branch>/plan/commits/NN-<slug>.md`，僅包含當前 commit 的：目標、預期檔案、實作步驟、邊界或排除範圍、測試要點，以及（若有特殊 staging 考量時）手動 commit 指引。

預設不為每個 commit 建立詳細規劃檔案；小型 commit 僅保留在 `IMPLEMENTATION_PLAN.md` 即可。

## 輸出格式

建立或更新計畫檔案後，回應格式如下：

```text
Plan saved to: .ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md

Commit plan:
1. feat: #<issue_number> 某個 commit 標題
   - 目的：...
   - 主要變更：...
   - 詳細規劃：commits/01-<slug>.md
2. fix: #<issue_number> 某個 commit 標題
   - 目的：...
   - 主要變更：...
```

只有在建立了獨立的詳細規劃檔案時，才加入 `詳細規劃` 欄位。若無 issue 編號，使用相同格式但省略 issue 參照。

## 執行限制

在列出完整的 commit 計畫並儲存計畫 Markdown 檔案後：
- 停止。
- 不修改任何功能程式碼。
- 告知使用者後續由 `dev-builder` 逐一執行 Commit Checklist。

若使用者後續要求調整計畫：
- 先讀取現有的 `.ai/<sanitized-branch>/plan/IMPLEMENTATION_PLAN.md`。
- 若調整源自與上游 SA + SD 文件不一致的需求，先更新上游文件再更新計畫。
- 就地更新同一份計畫檔案，保留已確認的內容，除非新回饋有所矛盾。

## 下一步引導（純提示，不主動調用）

完成後主動告知使用者後續選項，但**不自行調用**下一個 skill，由使用者決定與調用：
- **順流下一步**：計畫完成後由 `dev-builder` 逐一執行 Commit Checklist。
- **回頭修正**：commit 級自行修訂計畫；SD 設計級 → `codebase-sa-sd-finalizer`；需求／模組分解級 → `codebase-sa-builder`。

完整流程見 `WORKFLOW.md`。
