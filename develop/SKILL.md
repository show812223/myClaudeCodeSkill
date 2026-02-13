---
name: develop
description: 完整功能開發流水線。一鍵觸發：設計 → 實作 → 測試 → 審查。使用此技能當被要求開發新功能、新增頁面、建立元件時。
---

# 功能開發流水線

此技能觸發完整的功能開發自動化流水線。每個階段由專屬 agent 串行執行，確保檔案不會被同時編輯。

## 使用方式

```
/develop [需求描述]
```

範例：
```
/develop 在 syncobox-task 中新增任務看板頁面，需要拖拽排序功能
/develop 建立使用者管理頁面，包含 CRUD 和搜尋功能
```

## 流水線步驟（嚴格串行）

### Phase 1：需求分析（主對話）

在開始委派之前，先完成：
1. 分析 `$ARGUMENTS` 中的需求描述
2. 確認目標 package（根據功能對應正確的 package）
3. 搜尋類似實作作為參考（使用 Grep/Glob）
4. 確認需要建立/修改的檔案清單
5. 讀取 `.github/copilot-instructions.md` 了解規範

### Phase 2：UI 設計與功能實作（frontend-dev agent）

委派給 `frontend-dev` agent，提供：
- 需求描述
- 目標 package 和檔案路徑
- 參考的類似實作
- 需要建立的 interface / domain-class

**frontend-dev 負責**：
- 設計 UI 版面（Vuetify 3 元件）
- 建立 Service Class（繼承 PagingTool）
- 實作頁面邏輯（composables、utils）
- 定義 TypeScript interface（domain-classes/）
- 建立 i18n 翻譯（zh-TW + en-US）

**完成條件**：所有原始碼檔案建立完成，`pnpm lint` 通過

### Phase 3：測試撰寫（test-engineer agent）

等待 Phase 2 完成後，委派給 `test-engineer` agent，提供：
- Phase 2 建立的檔案清單
- Service Class 的方法列表和介面
- 需要 mock 的 API 端點

**test-engineer 負責**：
- 撰寫 Service Class 的單元測試（100% 覆蓋率目標）
- 撰寫 E2E 測試（主要使用者流程）

**完成條件**：`pnpm test` 通過，覆蓋率達標

### Phase 4：程式碼審查（code-reviewer agent）

等待 Phase 3 完成後，委派給 `code-reviewer` agent：
- 審查所有 Phase 2 + Phase 3 的變更
- 執行 `git diff` 查看變更
- 對照規範交叉檢查
- 輸出審查報告

**完成條件**：輸出分類標記的審查報告

### Phase 5：自動修復（auto-fixer agent，條件式）

僅在 Phase 4 發現 CRITICAL 或 WARNING 問題時執行：
- 根據審查報告逐項修復
- 執行 `pnpm lint` + `pnpm test` 驗證修復

**跳過條件**：若審查報告無 CRITICAL/WARNING，直接進入 Phase 6

### Phase 6：最終驗證（主對話）

1. 執行 `pnpm lint` — 確認無 lint 錯誤
2. 執行 `pnpm test` — 確認所有測試通過
3. 輸出完成摘要，列出所有新建/修改的檔案

## 檔案衝突防護

- 每個 Phase 串行執行，絕不同時運行
- frontend-dev 只編輯原始碼，test-engineer 只編輯測試檔案
- code-reviewer 完全唯讀
- auto-fixer 只修改審查報告中指定的檔案

## 專案結構參考

| 別名 | 路徑 |
|------|------|
| `~api` | `packages/syncobox-api/app` |
| `~ui` | `packages/syncobox-ui/app` |
| `~stores` | `packages/syncobox-stores/app` |
| `~bim` | `packages/syncobox-bim/app` |
| `~portal` | `packages/syncobox-sharedPortal/app` |
| `~task` | `packages/syncobox-task/app` |
| `~/` | `apps/monolith_tsmc/app/` |
