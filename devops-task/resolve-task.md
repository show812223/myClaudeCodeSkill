讀取 Azure DevOps Task，理解需求，直接在專案中修改程式碼解決問題。不會更新 Azure DevOps Task 狀態或加 Comment。

先閱讀 `.claude/skills/devops-task/SKILL.md` 的 "Part 2：resolve-task" 章節，以及 `references/resolve-patterns.md` 和 `references/parsing-patterns.md`，了解完整的執行規則和策略定義。

## 輸入

Task ID: $ARGUMENTS

可以是單一 ID（例如 `12345`）或多個 ID（例如 `12345,12346`）。
ID 後面可附加額外指示（例如 `12345 只改 Service 層`）。

## 執行步驟

### 1. 讀取 Task

使用 Azure DevOps MCP 取得 Task 完整資訊：
- 標題、描述、Work Item Type、Acceptance Criteria
- 現有 Comments（可能有補充資訊）
- 關聯 Work Items

### 2. 分類問題類型

根據 Task 內容判斷類型：

**🐛 Bug 修復**（Type=Bug、錯誤現象、重現步驟）
→ 定位問題根因 → 修復程式碼 → 驗證

**✨ 新功能**（Type=Task/Story、需求規格、API 設計）
→ 分析架構 → 按分層順序實作（Entity → Service → Controller → DI）

**♻️ 重構優化**（重構、優化、效能、整理）
→ 影響分析 → 小步重構 → 確保行為不變

**🧪 補寫測試**（測試、test、coverage）
→ 分析測試基礎設施 → 撰寫測試 → 涵蓋正常/邊界/錯誤

可能是複合型（例如修 Bug + 補測試），依主要目標排序依序執行。

### 3. 分析專案程式碼

- 專案結構（*.sln / *.csproj）
- 分層架構慣例（Controller / Service / Repository / Model）
- Coding style（.editorconfig、既有程式碼風格）
- Error handling pattern（Exception / Result / ProblemDetails）
- 找到類似功能作為實作範本

排除路徑：`bin/`、`obj/`、`Migrations/`、`node_modules/`

### 4. 執行修改

直接修改專案程式碼。遵循原則：
- **與專案風格一致**（偵測 .editorconfig 和既有慣例）
- **最小變更**（Bug 修復不附帶重構）
- **不確定就問**（需求模糊時先詢問）

### 5. 驗證

- `dotnet build` — 確認編譯通過
- `dotnet test` — 確認既有測試不壞
- 如果是新功能，確認 DI 註冊完整

### 6. 回報結果

在終端輸出執行摘要（不回寫 Azure DevOps）：
- Task 資訊（ID、標題、類型）
- 修改了哪些檔案、改了什麼
- 建置和測試結果
- 需要使用者手動做的事（Migration、設定檔等）
