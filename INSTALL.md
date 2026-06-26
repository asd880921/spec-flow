## 安裝教程

### 前置需求：程式碼知識圖譜查找能力

spec-flow 在分析既有專案時，SA / SD 階段會透過「程式碼知識圖譜查找」能力查找程式碼現況。**工作流本身不綁定特定產品**——你可以使用任何能在既有 codebase 中查找模組、流程、資料流與既有實作的工具（一般以 MCP server 形式提供）。

設定時只需在 Claude Code / Codex 環境中接上一個提供此能力的工具/MCP server 即可；之後 skill 會自動透過該能力查找現況，skill 文件不需改動。若該能力不可用或查無結果，skill 會自動退用一般的程式碼搜尋與檔案閱讀。

### Claude Code 安裝/更新/移除 指令說明
安裝指令 (於終端開啟 Claude Cli 後，依序輸入以下指令)：
```
/plugin marketplace add asd880921/spec-flow
```
```
/plugin install spec-flow
```
```
/reload-plugins
```

更新指令：
```
/plugin marketplace update spec-flow
```
```
/reload-plugins
```

移除指令：
```
/plugin uninstall spec-flow
```
```
/plugin marketplace remove spec-flow
```

### Codex 安裝/更新/移除 指令說明
安裝指令 (開啟 Command 終端後直接輸入)：
```powershell
codex plugin marketplace add asd880921/spec-flow
```
```powershell
codex plugin add spec-flow@spec-flow
```
更新指令：

```powershell
codex plugin marketplace upgrade spec-flow
```
```powershell
codex plugin add spec-flow@spec-flow
```

移除指令：
```powershell
codex plugin remove spec-flow@spec-flow
```
```powershell
codex plugin marketplace remove spec-flow
```
