---
name: fix-bug
description: 快速 Bug 修復流水線。一鍵觸發：診斷 → 修復 → 測試 → 驗證。使用此技能當被要求修 bug、解決問題、修復錯誤時。
---

# Bug 修復流水線

此技能觸發快速的 bug 修復自動化流水線。跳過 UI 設計階段，直接進入診斷與修復。

## 使用方式

```
/fix-bug [bug 描述]
```

範例：
```
/fix-bug 登入頁面點擊登入後無反應，可能是 AuthService 的問題
/fix-bug ViewerWithTools 的工具列按鈕在切換模型後消失
/fix-bug syncobox-task 的任務列表分頁不正確
```

## 流水線步驟（嚴格串行）

### Phase 1：診斷（主對話）

1. 分析 `$ARGUMENTS` 中的 bug 描述
2. 搜尋相關檔案和程式碼（Grep/Glob）
3. 定位可能的問題來源
4. 確認修復策略

### Phase 2：修復（frontend-dev agent）

委派給 `frontend-dev` agent，提供：
- Bug 描述和複現步驟
- 定位到的問題檔案和行號
- 建議的修復方向

**frontend-dev 負責**：
- 修復程式碼邏輯
- 確保修復不引入新問題
- 執行 `pnpm lint` 驗證

**完成條件**：bug 已修復，`pnpm lint` 通過

### Phase 3：回歸測試（test-engineer agent）

等待 Phase 2 完成後，委派給 `test-engineer` agent：
- 為修復的 bug 新增回歸測試案例
- 確保既有測試仍通過

**完成條件**：`pnpm test` 通過，新增回歸測試

### Phase 4：最終驗證（主對話）

1. 執行 `pnpm lint`
2. 執行 `pnpm test`
3. 輸出修復摘要

## 檔案衝突防護

- frontend-dev 只修改原始碼
- test-engineer 只修改測試檔案
- 兩者串行執行，不會同時編輯
