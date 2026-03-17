---
name: manage
description: 智能工作流管理器。分析需求後自動路由到正確的 skill 或組合多個 skill 執行。使用此技能當不確定該用哪個 skill、需要組合多個工作流、或想用自然語言描述任務時。
---

# Manage - 智能工作流管理器

分析自然語言需求，自動路由到正確的 skill 或規劃多 skill 的組合工作流。

## 使用方式

```
/manage [自然語言需求]
```

範例：
```
/manage 用戶反映登入後看不到 dashboard 資料
/manage 新增一個專案管理頁面，含甘特圖
/manage 這次 sprint 的程式碼品質檢查，有問題就修
/manage 幫 user-settings 頁面生成操作手冊
/manage 建立一個 Task：實作檔案上傳功能
```

## 執行流程

### Phase 1：需求分析

解析 `$ARGUMENTS`，判斷：
1. 需求類型（單一 skill 或複合工作流）
2. 涉及的 skill
3. 執行順序

### Phase 2：路由決策

根據以下規則路由到對應的 skill：

```
分析需求關鍵字
  │
  ├─ bug、錯誤、修復、壞了、不能用、crash、報錯
  │  → /develop --fix [描述]
  │
  ├─ 新功能、新增頁面、建立、開發、實作
  │  → /develop [描述]
  │
  ├─ 重構、清理、改善、技術債、優化結構
  │  → /refactor [目標]
  │
  ├─ 審查、review、品質、檢查程式碼、lint
  │  → /review [目標]
  │  （若包含「修正」「fix」→ /review --fix [目標]）
  │
  ├─ 測試、test、覆蓋率、vitest、e2e
  │  → /test [類型] [目標]
  │  （判斷 unit / e2e / all）
  │
  ├─ 手冊、文件、截圖、操作說明、文檔
  │  → /web-doc-gen [描述]
  │
  ├─ task、工作項目、DevOps、建立任務
  │  → /create-task 或 /reply-task 或 /resolve-task
  │  （根據動詞判斷：建立→create、回覆→reply、解決→resolve）
  │
  ├─ API 怎麼用、文檔查詢、Vuetify 用法
  │  → /context7 [查詢]
  │
  ├─ UI 設計、設計系統、配色、字體
  │  → /ui-ux-pro-max [需求]
  │
  └─ 包含多個動作（複合需求）
     → 進入 Phase 3 規劃組合工作流
```

### Phase 3：複合工作流規劃（僅複合需求）

當需求涉及多個 skill 時：

1. 列出需要執行的 skill 和順序
2. **向用戶確認執行計畫**（等待用戶同意後才執行）
3. 依序執行每個 skill

**複合工作流範例**：

需求：「新增一個專案管理頁面，完成後生成操作手冊，再建立 DevOps Task 追蹤」
```
Step 1: /develop 新增專案管理頁面
Step 2: /web-doc-gen 為專案管理頁面生成操作手冊
Step 3: /create-task 建立追蹤 Task
```

需求：「這次 sprint 做一次完整品質檢查，有問題就修」
```
Step 1: /review packages/syncobox-task/  → 產出報告
Step 2: /auto-fix 根據報告修正問題
Step 3: /test all 確認測試通過
```

需求：「重構 TaskService，補齊測試，審查品質」
```
Step 1: /refactor TaskService
Step 2: /test unit TaskService
Step 3: /review packages/syncobox-task/app/services/TaskService.ts
```

### Phase 4：執行

- **單一 skill**：直接呼叫對應的 skill
- **複合工作流**：依序呼叫，每個 skill 完成後報告進度

### Phase 5：摘要

輸出所有執行結果的總結：
- 執行了哪些 skill
- 每個 skill 的產出
- 是否有未解決的問題

## 可用 Skill 清單

| Skill | 指令 | 用途 |
|-------|------|------|
| develop | `/develop [需求]` | 新功能開發（完整流水線） |
| develop --fix | `/develop --fix [bug]` | 快速 Bug 修復 |
| refactor | `/refactor [目標]` | 程式碼重構 |
| review | `/review [目標]` | 程式碼審查（唯讀） |
| review --fix | `/review --fix [目標]` | 審查 + 自動修正 |
| test | `/test [unit/e2e/all] [目標]` | 撰寫測試 |
| auto-fix | `/auto-fix` | 自動修正 review 問題 |
| web-doc-gen | `/web-doc-gen [描述]` | 生成網頁操作手冊 |
| devops-task | `/create-task`, `/reply-task`, `/resolve-task` | Azure DevOps 整合 |
| context7 | `/context7 [查詢]` | API 文件查詢 |
| ui-ux-pro-max | `/ui-ux-pro-max [需求]` | UI/UX 設計系統 |
| ui-development | `/ui-development [需求]` | Vue/Vuetify 元件建構 |

## 注意事項

- 如果無法確定需求類型，**詢問用戶確認**再執行
- 複合工作流必須**等用戶確認計畫**後才開始執行
- 每個 skill 完成後報告進度，讓用戶掌握狀態
- 如果某個 skill 執行失敗，停下來報告並詢問用戶下一步
