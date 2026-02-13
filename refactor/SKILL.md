---
name: refactor
description: 程式碼重構流水線。一鍵觸發：審查 → 重構 → 測試 → 驗證。使用此技能當被要求重構、清理技術債、改善程式碼品質時。
---

# 程式碼重構流水線

此技能觸發程式碼重構自動化流水線。先審查發現問題，再逐項修復。

## 使用方式

```
/refactor [目標描述]
```

範例：
```
/refactor 重構 syncobox-bim 的 ModelViewerService，拆分過大的方法
/refactor 清理 syncobox-task 中的技術債，消除 any 類型
/refactor packages/syncobox-forge/app/components/ViewerWithTools.vue 改善程式碼品質
```

## 流水線步驟（嚴格串行）

### Phase 1：審查掃描（code-reviewer agent）

委派給 `code-reviewer` agent，提供：
- `$ARGUMENTS` 中的目標範圍
- 請求完整的審查報告

**code-reviewer 負責**：
- 掃描目標檔案/目錄
- 對照所有規範檢查
- 執行 `pnpm lint`
- 輸出分類標記的審查報告

**完成條件**：輸出完整審查報告

### Phase 2：自動修復（auto-fixer agent）

等待 Phase 1 完成後，委派給 `auto-fixer` agent：
- 傳入 code-reviewer 的審查報告
- 按 CRITICAL → WARNING → SUGGESTION 順序修復

**auto-fixer 負責**：
- 逐項修復報告中的問題
- 每次修復後確認 `pnpm lint` 通過
- 回報已修復和未能修復的項目

**完成條件**：所有可自動修復的問題已處理

### Phase 3：測試更新（test-engineer agent）

等待 Phase 2 完成後，委派給 `test-engineer` agent：
- 傳入被修改的檔案清單
- 更新受影響的測試

**test-engineer 負責**：
- 更新因重構而失敗的測試
- 為新拆分的方法補充測試
- 確保覆蓋率不下降

**完成條件**：`pnpm test` 全部通過

### Phase 4：最終驗證（主對話）

1. 執行 `pnpm lint`
2. 執行 `pnpm test`
3. 對比重構前後的程式碼品質
4. 輸出重構摘要

## 檔案衝突防護

- code-reviewer 完全唯讀（無 Edit/Write 工具）
- auto-fixer 只修改報告中指定的原始碼
- test-engineer 只修改測試檔案
- 三者嚴格串行，不會同時編輯
