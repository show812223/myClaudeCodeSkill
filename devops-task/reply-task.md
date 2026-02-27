讀取 Azure DevOps Task，智能判斷問題類型，掃描專案程式碼找出相關資訊，整理後以 Comment 回寫到 Task。

先閱讀 `.claude/skills/devops-task/SKILL.md` 的 "Part 1：reply-task" 章節，以及 `references/comment-template.md` 和 `references/parsing-patterns.md`，了解完整的執行規則和策略定義。

## 輸入

Task ID: $ARGUMENTS

可以是單一 ID（例如 `12345`）或多個 ID（例如 `12345,12346`）。

## 執行步驟

### 1. 讀取 Task

使用 Azure DevOps MCP 工具取得 Task 的完整資訊：
- 標題、描述（HTML）、Work Item Type（Task/Bug/User Story）
- 狀態、指派對象
- 現有 Comments（避免重複回答）

### 2. 分類問題類型

根據 Task 內容判斷類型，可以是一種或多種的組合：

**API 規格類**（關鍵字：API、端點、接口、參數、回傳格式、怎麼呼叫）
→ 掃描 Controllers / Minimal API，整理端點規格、參數、Response 型別

**Bug 排查類**（Work Item Type = Bug，或包含：錯誤、異常、500、回傳不對）
→ 定位相關程式碼，分析可能原因，檢查近期 git 變更

**Model 定義類**（關鍵字：欄位、資料結構、DTO、Model、schema、型別）
→ 掃描 Models/DTOs/Entities，列出屬性、型別、驗證規則、Mapping 關係

**通用類**（無法歸類）
→ 提取技術關鍵字，全文搜尋相關程式碼，以自然語言回覆

### 3. 掃描程式碼

根據分類結果掃描對應的檔案。排除：`bin/`、`obj/`、`Migrations/`、`Tests/`、`*.g.cs`

### 4. 組合回覆

根據問題類型使用對應的回覆格式。每則回覆固定包含：
- 狀態 emoji + 標題（✅ / 🔍 / 📋 / 💬）
- `> Task #{ID}: {標題}` 引用
- `> 由 Claude Code 自動產生 | {日期}` 簽名

### 5. 回寫到 Task

使用 Azure DevOps MCP `add_work_item_comment` 回寫。
- 內容 < 2000 字 → 單則 Comment
- 內容 > 2000 字 → 分成「摘要」+「詳細」兩則

### 6. 回報結果

輸出處理摘要：每個 Task 的問題分類、回覆要點、Azure DevOps 連結。
