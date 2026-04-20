---
name: review-pr
description: >
  PR review 與 release 管理全流程。涵蓋 PR 程式碼審查、squash merge、release 分支 cherry-pick、上線後分支清理。
  當使用者提到以下任何關鍵字時觸發此 skill：review PR、審 PR、code review、merge PR、
  發 release、cherry-pick、release 分支、上 prod、發佈流程、分支管理、分支清理、
  PR 規範、PR checklist。也適用於使用者要求檢查 Azure DevOps PR、
  幫忙決定哪些 PR 該進 release、或整理 release notes 的場景。
  即使使用者只是簡單說「幫我看這個 PR」或「這次要上哪些功能」，也應觸發此 skill。
---

# PR Review 與 Release 管理流程

本 skill 定義了從 PR review 到上線的完整工作流程，適用於採用 **Release Branch Workflow** 的團隊。

## 流程總覽

```
feature/* → PR → squash merge → dev（整合測試）
                                    ↓
                        從 prod 切出 release 分支
                                    ↓
                        cherry-pick 本次要上的 commit
                                    ↓
                        release → PR → prod
                                    ↓
                        prod merge 回 dev + 清理分支
```

---

## 階段一：PR Review

當使用者請你 review PR 時，依序檢查以下項目。

### 1. PR 格式檢查

- **標題格式**是否正確：`[scope][類型] 簡短描述 (#工作項目編號)`
  - scope（monorepo 必須有）：`erp`、`eip`、`ui-kit`、`utils`、`api`、`infra`
  - 類型：`feat`、`fix`、`refactor`、`chore`、`docs`、`style`、`test`、`perf`
  - 範例：`[erp][feat] 新增 ECPay 發票整合 (#1234)`
- **PR 描述**是否包含：
  - 對應的 Azure DevOps Work Item 編號（`AB#xxxx`）
  - 預計上線的版本或 sprint
  - 變更摘要
  - 測試範圍
  - 相依 PR（若有）

如果格式不符，列出具體缺漏並建議修正。

### 2. PR 粒度檢查

一個 PR 應對應**一個發佈單位**（會一起上線、一起 rollback 的東西）。

標記為問題的情況：
- 同一 PR 裡包含**多個不相關功能**（例如同時改登入和報表）→ 建議拆 PR
- Monorepo 中一個 PR **跨多個 app 做不同目的的修改** → 建議拆開
- PR 過大（超過 500 行變更）且包含不同性質的修改 → 建議拆開

可接受的情況：
- 改共用 package（如 `ui-kit`）同時更新所有使用它的 app → OK，但需標註影響範圍
- 跨 app 的同一目的修改（如升級 Vue 版本）→ OK，scope 用 `[erp,eip]`

### 3. 程式碼品質檢查

依序檢查：

**架構與設計**
- 邏輯是否清晰、職責劃分是否合理
- 是否有不必要的複雜度
- 是否遵循既有的程式碼慣例與風格

**Vue.js / 前端特定**
- 元件是否適當拆分（單一職責）
- Props 是否有適當的型別定義和預設值
- 是否有潛在的效能問題（不必要的 re-render、大量 watcher）
- 響應式設計是否考慮到（mobile-first）
- Vuetify 元件使用是否正確（density、spacing）

**共用程式碼影響（Monorepo）**
- 如果修改了 `packages/` 下的共用程式碼，檢查影響範圍
- 確認 PR 描述中是否標註了所有受影響的 app
- 確認是否有 breaking change

**安全性**
- 是否有硬編碼的 secret 或 API key
- 使用者輸入是否有適當的驗證/sanitize
- API 呼叫是否有錯誤處理

**可維護性**
- 命名是否清晰（變數、函式、元件）
- 是否有足夠的註解（複雜邏輯處）
- 是否有 TODO/FIXME 需要追蹤

### 4. Review 產出格式

Review 完成後，以下列格式輸出：

```
## PR Review 結果

### 📋 格式檢查
- [x/✗] 標題格式
- [x/✗] Work Item 連結
- [x/✗] 預計上線版本
- [x/✗] 變更摘要
- [x/✗] 測試範圍

### 📐 粒度
- [OK / 建議拆分] 原因：...

### 🔍 程式碼
（依嚴重程度排列）
- 🔴 必須修正：...
- 🟡 建議改善：...
- 🟢 小建議：...

### 📦 影響範圍（Monorepo）
- 影響的 app：erp-web / eip-web / 無
- 共用 package 變更：有 / 無
- Breaking change：有 / 無

### ✅ 結論
- [ ] 可合併
- [ ] 需修正後再合併
- [ ] 需要重大修改
```

---

## 階段二：Merge 到 Dev

### Merge 規則

- **強制使用 Squash Merge**：一個 PR 壓成一顆 commit
- Squash 後的 commit message 自動使用 PR title，所以 PR title 必須清晰
- **不要在 merge 後立刻刪除 feature 分支**（關掉 Azure DevOps 自動刪除選項）

### Merge 後確認

- dev 環境是否正常部署
- 是否需要通知 QA 或其他相關人員

---

## 階段三：Release 準備

當使用者要準備 release 時，協助以下工作。

### 1. 決定要上的功能

協助使用者：
- 列出目前在 dev 但還沒上 prod 的 PR/commit
- 根據 PR 標題的「預計上線版本」標註，篩選本次要上的功能
- 確認是否有相依性需要一起帶（特別是共用 package 的修改）

如果有 Azure DevOps 工具可用，查詢方式：
- 搜尋 dev 分支上還沒合到 prod 的 commit
- 對照 Work Item 的 iteration / sprint 標籤

### 2. 建立 Release 分支

```bash
# 從 prod 切出（不是從 dev！）
git checkout prod && git pull
git checkout -b release/YYYY-MM-DD
```

Monorepo 且不同 app 獨立上線時：
```bash
git checkout -b release/erp-YYYY-MM-DD
git checkout -b release/eip-YYYY-MM-DD
```

### 3. Cherry-pick

由於強制 squash merge，每個 PR 就是一顆 commit，直接挑：

```bash
git cherry-pick <sha>
```

如果遇到衝突：
```bash
# 解衝突後
git add .
git cherry-pick --continue

# 或放棄
git cherry-pick --abort
```

確認影響範圍（monorepo）：
```bash
git show --stat <sha>            # 看改了哪些檔案
git show <sha> -- packages/      # 看是否動到共用 package
```

### 4. Release 驗證

- 部署到 staging / pre-prod 環境
- 確認所有 cherry-pick 的功能正常
- 如果有共用 package 修改，所有受影響的 app 都要驗

### 5. 產生 Release Notes

協助使用者整理本次 release 包含的功能清單：

```
## Release YYYY-MM-DD

### 新功能
- [erp] ECPay 發票整合 (#1234)
- [eip] 新增加班申請流程 (#1345)

### Bug 修正
- [erp] 修正發票金額計算 (#1567)

### 改善
- [ui-kit] 重構 Button 元件 (#1890)

### 影響範圍
- erp-web ✅
- eip-web ✅
- ui-kit ✅
```

---

## 階段四：上線與清理

### 上線步驟

```bash
# 1. Release PR merge 到 prod
# 2. 打 tag
git checkout prod && git pull
git tag -a vYYYY.MM.DD -m "Release YYYY-MM-DD"
git push --tags

# 3. Prod merge 回 dev（重要！不能省略）
git checkout dev && git pull
git merge prod && git push
```

### 分支清理

確認功能已在 prod 上正常運行後，才清理分支：

```bash
# 列出已合到 prod 的 feature 分支
git branch -r --merged prod | grep feature/

# 批次刪除
git push origin --delete feature/xxx feature/yyy

# 同步本地
git fetch --prune
```

Monorepo 額外注意：共用 package 相關的 feature 分支，建議確認**所有受影響的 app 都穩定後（約 1 週）**再刪除。

---

## 快速判斷表

| 使用者說 | 執行階段 |
|---|---|
| 「幫我 review 這個 PR」 | → 階段一 |
| 「這個 PR 可以 merge 嗎」 | → 階段一 + 階段二 |
| 「這次要上哪些功能」 | → 階段三 步驟 1 |
| 「幫我準備 release」 | → 階段三 完整流程 |
| 「幫我整理 release notes」 | → 階段三 步驟 5 |
| 「上完 prod 了，接下來呢」 | → 階段四 |
| 「可以清分支了嗎」 | → 階段四 分支清理 |

---

## Azure DevOps 設定備忘

以下設定只需做一次：

- **Repo Settings → Policies → Limit merge types** → 只勾選 Squash merge（dev 和 prod 都設）
- **PR Complete 對話框** → 取消勾選「Delete source branch after merging」
- **Branch Policies → dev** → 必須有至少 1 個 reviewer
- 建議建立 **Release Work Item**，底下 link 本次要上的所有 User Story / Bug
