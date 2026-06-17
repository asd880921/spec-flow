# spec-flow

Spec 驅動的開發管線，一組會互相銜接的 skill：從需求一路走到交付。

## 這組有哪些 skill

| Skill | 階段 | 做什麼 |
| --- | --- | --- |
| `codebase-sa-builder` | 需求分析（入口） | 由 PRD 產出架構層級 SA；反問單一 / 拆分（拆分產出 1 母文件＋N 模組 SA） |
| `codebase-sa-sd-finalizer` | 補系統設計 | 把一份 SA 複製為 `-sd.md`，收斂 SA＋補 SD＋合併驗收條件 |
| `dev-planner` | 規劃 commit | 由 `-sd.md` 規劃 atomic commit 計畫 |
| `dev-builder` | 寫程式 | 每次做一個 commit、給你 git 指令；不自己 commit |
| `dev-acceptance-verifier` | 驗收 | 逐條驗收條件判定（通過／不通過／需人工確認），唯讀 |
| `codebase-design-doc-cleaner` | 交付清洗 | 移除來源標記與待釐清事項，產出乾淨交付版 |

完整流程圖與使用方式見 [WORKFLOW.md](./WORKFLOW.md)。

## 調用方式

裝成 plugin 後，調用方式與一般 skill 相同：AI 會依各 skill 的 description 自動觸發，你也可指名調用。每支 skill 跑完會主動引導下一步、以及遇到問題時往上修正的去向（純引導、不會自動調用下一支）。

- Claude Code：skill 顯示為 `spec-flow:<skill>`。
- Codex：skill 屬於 plugin `spec-flow`。

## 安裝

**Claude Code**（在互動式終端）
```
/plugin marketplace add <此 marketplace 根目錄路徑>
/plugin install spec-flow
```

**Codex**（編輯 `~/.codex/config.toml`）
```toml
[marketplaces.spec-flow]
source_type = "local"
source = "<此 marketplace 根目錄路徑>"

[plugins."spec-flow@spec-flow"]
enabled = true
```

> 安裝並驗證後，移除原本散落在 `~/.codex/skills/` 與 Claude skills 目錄中的這 6 個獨立 skill 資料夾，避免與 plugin 版本重複。
