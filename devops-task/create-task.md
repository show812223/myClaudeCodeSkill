用自然語言描述需求，自動產生結構化的標題和描述，建立 Azure DevOps Work Item。

先閱讀 `.claude/skills/devops-task/SKILL.md` 的 "Part 3：create-task" 章節，以及 `references/task-template.md`，了解完整的產生規則。

## 輸入

需求描述: $ARGUMENTS

支援可選的標記：
- `[bug]` / `[task]` / `[story]` — 指定 Work Item 類型（預設自動判斷）
- `[project:名稱]` — 指定專案（預設使用當前專案）

範例：
- `建立使用者登入 API，支援帳號密碼和 OAuth`
- `[bug] 使用者清單 API 分頁參數無效時回傳 500`
- `[project:MyProject] 重構 Order Service`

## 執行步驟

### 1. 解析輸入

從自然語言輸入中提取：
- Work Item Type（`[bug]`/`[task]`/`[story]` 標記，或從內容自動判斷）
- 專案名稱（`[project:X]` 標記，或使用預設）
- 需求描述（去除標記後的文字）

### 2. 判斷 Work Item Type

如果沒有明確標記：
- 描述錯誤現象（錯誤、異常、500、壞掉、回傳不對）→ **Bug**
- 描述要做的事（建立、實作、新增、重構、優化）→ **Task**

### 3. 分析複雜度，產生描述

根據需求複雜度自動選擇描述深度：

**簡單**（< 20 字、單一功能）→ 1-2 段說明
**中等**（有規格、多面向）→ 需求說明 + 預期行為 + Acceptance Criteria
**複雜**（多元件、架構設計）→ 需求 + 技術規格 + 影響範圍 + AC + 注意事項

標題規則：不超過 80 字、動詞開頭、包含功能範圍。

如果在專案目錄下，可掃描現有程式碼來豐富描述（參考命名慣例、現有資料結構等）。

### 4. 確認

輸出產生的 Work Item 內容，等待使用者確認後才建立。

### 5. 建立

使用 Azure DevOps MCP `create_work_item` 建立 Work Item。
建立成功後輸出 Task ID 和 Azure DevOps 連結。
