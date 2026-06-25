## 安裝教程

### 前置需求：GitNexus

spec-flow 在分析既有專案時，SA / SD 階段會透過 GitNexus 查找程式碼現況，因此需先安裝並設定 GitNexus。

- 原倉庫：[abhigyanpatwari/GitNexus](https://github.com/abhigyanpatwari/GitNexus) —— 安裝與配置以官方說明為準。
- 中文教學（他人整理）：[GitNexus 安裝與配置 — aivi.fyi](https://www.aivi.fyi/llms/gitnexus#%E4%B8%89%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE)，可作輔助參考。

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
