---
name: review
description: 程式碼審查與風格檢查。使用此技能當被要求審查程式碼、review、品質檢查、lint、檢查程式碼、修正風格時。
---

# Review Skill

整合程式碼審查（唯讀）和 Lint 風格檢查（可自動修正）的統一品質技能。

## 使用方式

```
/review [選項] [目標]
```

| 指令 | 說明 |
|------|------|
| `/review [目標]` | 唯讀審查報告（不修改程式碼） |
| `/review --fix [目標]` | 審查 + 自動修正 Lint 問題 |

範例：
```
/review packages/syncobox-task/app/pages/task-list.vue
/review --fix syncobox-ui 的 ProjectList 頁面
/review syncobox-bimContent 所有 components
```

## 判斷模式

解析 `$ARGUMENTS`：
- 包含 `--fix` → 審查 + 自動修正模式
- 不含 `--fix` → 唯讀審查模式（預設）

---

## 執行流程

### Phase 1：收集範圍

確認要檢查的頁面/模組路徑：

```
packages/{package}/app/
├── pages/{指定頁面}.vue        # 頁面本身
├── components/{相關元件}/      # 頁面使用的元件
├── composables/{相關邏輯}.ts   # 頁面使用的 composable
├── services/{相關服務}.ts      # 頁面使用的 service
└── types/{相關型別}.ts         # 頁面使用的型別定義
```

### Phase 2：逐項檢查

掃描路徑下所有 `.vue`、`.ts` 檔案，依據以下 5 大類標準審查。

#### 1. UI 規範檢查

| 項目 | 檢查內容 |
|------|----------|
| Vuetify 元件 | 是否使用 Vuetify 元件取代原生 HTML |
| Script Setup | 是否使用 `<script setup lang="ts">` |
| Props 定義 | 是否使用 TypeScript 泛型定義 props |
| 響應式狀態 | 是否優先使用 `ref` |
| API 呼叫 | 是否使用 `useFetch` 或 `useCustomFetch` |
| 樣式 | 是否使用 Vuetify utility classes 而非 scoped style |

禁止使用的原生 HTML：
```
❌ <button>、<input>、<select>、<table>、<div>(容器)
✅ <v-btn>、<v-text-field>、<v-select>、<v-data-table>、<v-sheet>/<v-card>
```

#### 2. TypeScript 規範檢查

| 項目 | 檢查內容 |
|------|----------|
| 型別定義 | 禁止使用 `any` |
| 未使用變數 | 不應有未使用的變數或 import |
| Console | 不應有 `console.log`（除錯誤處理外） |
| 共用型別 | 是否從 shared package 導入型別 |

#### 3. Service Class 規範檢查

| 項目 | 檢查內容 |
|------|----------|
| 命名規範 | Service class 以 `Service` 結尾 |
| 單一職責 | 每個 service 只負責一個領域 |
| 錯誤處理 | API 呼叫是否有適當的錯誤處理 |
| 型別安全 | 回傳值是否有明確型別定義 |

#### 4. 單元測試覆蓋檢查

| 項目 | 檢查內容 |
|------|----------|
| 測試檔案存在 | 元件/composable/service 是否有對應的 `.test.ts` |
| 測試路徑 | 測試檔案是否在 `tests/` 目錄下 |
| 測試命名 | 測試檔案命名是否符合 `{Name}.test.ts` |
| 覆蓋率標準 | 行覆蓋率 ≥ 80%、分支覆蓋率 ≥ 70% |

```
目標檔案                                    測試檔案
─────────────────────────────────────────────────────────────
app/pages/{page}.vue                  →  tests/pages/{page}.test.ts
app/components/{Component}.vue        →  tests/components/{Component}.test.ts
app/composables/{useName}.ts          →  tests/composables/{useName}.test.ts
app/services/{ServiceName}.ts         →  tests/services/{ServiceName}.test.ts
```

#### 5. E2E 測試覆蓋檢查

| 項目 | 檢查內容 |
|------|----------|
| E2E 測試存在 | 頁面是否有對應的 `.spec.ts` 測試 |
| 測試路徑 | 在 `SyncoBox.Testing.E2E.Playwright/tests/` 下 |
| 測試命名 | `{編號}-{功能名稱}.spec.ts` |
| 關鍵流程覆蓋 | 涵蓋頁面主要使用者流程 |
| 測試結構 | 使用 `test.step` 結構化步驟 |

### Phase 3：自動修正（僅 --fix 模式）

若 `$ARGUMENTS` 包含 `--fix`，執行自動修正：

```bash
# 1. 自動修正 Lint 問題
pnpm --filter @syncobox/{package-name} lint --fix

# 2. 再次驗證
pnpm --filter @syncobox/{package-name} lint
```

主要修正項目：
- Vue 規則：`multi-word-component-names`, `no-mutating-props`, `require-default-prop`, `valid-v-slot`
- TypeScript：型別安全、未使用變數、禁止 `any`
- Console：移除 `console.log`（保留 `console.error`）

常見修正範例：

```typescript
// ❌ 未使用變數 → 移除
const unused = 'value';

// ❌ any 型別 → 定義 interface
const data: any = {};
// ✅
interface UserData { id: string; name: string; }
const data: UserData = { id: '1', name: 'Test' };

// ❌ Props 未定義預設值
const props = defineProps<{ title?: string }>();
// ✅ 使用 withDefaults
const props = withDefaults(defineProps<{ title?: string }>(), { title: '' });
```

### Phase 4：輸出報告

```markdown
## 程式碼審查報告

**審查範圍**: `packages/syncobox-ui/app/pages/example.vue`

### ✅ 通過項目
- [x] 使用 `<script setup lang="ts">`
- [x] Props 使用 TypeScript 泛型
- [x] 無 `any` 型別

### ⚠️ 需改進項目
- [ ] 第 25 行：使用原生 `<button>`，應改用 `<v-btn>`
- [ ] 第 42 行：發現 `console.log`，請移除
- [ ] 缺少單元測試檔案

### 測試覆蓋率

#### 單元測試
| 檔案 | 測試檔案 | 狀態 |
|------|----------|------|
| `pages/example.vue` | `tests/pages/example.test.ts` | ❌ 缺少 |
| `services/ExampleService.ts` | `tests/services/ExampleService.test.ts` | ⚠️ 覆蓋率 65% |

#### E2E 測試
| 頁面功能 | E2E 測試檔案 | 狀態 |
|----------|--------------|------|
| 模型檢視頁面 | `tests/bim/01-model-views.spec.ts` | ❌ 缺少 |

### 建議修正
1. 將 `<button>` 改為 `<v-btn>`
2. 移除 `console.log`
3. 建立缺少的測試檔案
```

## 執行指令參考

```bash
# Lint 檢查
pnpm --filter @syncobox/{package-name} lint

# Lint 自動修正
pnpm --filter @syncobox/{package-name} lint --fix

# 單元測試覆蓋率
pnpm test --coverage -- packages/{package-name}/tests/

# 型別檢查
pnpm --filter @syncobox/{package-name} typecheck

# E2E 測試
pnpm e2e:local
```
