---
name: context7
description: 獲取最新的程式庫文檔。使用此技能當需要查詢 Vue、Vuetify、Nuxt、Pinia 等程式庫的最新 API 文檔時。
---

# Context7 Skill

此技能用於透過 Context7 MCP Server 獲取程式庫的最新文檔和程式碼範例。

## 適用場景

- 需要查詢程式庫的最新 API 用法
- 不確定某個 API 的正確使用方式
- 需要最新版本的程式碼範例
- 避免使用過時或不存在的 API
- **確認 Vuetify 元件是否符合 Material Design 3 規範**
- **查詢 Vuetify 元件的正確 props 和 slots**

## 支援的程式庫

| 程式庫 | Context7 ID |
|--------|-------------|
| Vue 3 | `/vuejs/core` |
| Vuetify 3 | `/vuetifyjs/vuetify` |
| Nuxt 3 | `/nuxt/nuxt` |
| Pinia | `/vuejs/pinia` |
| VueUse | `/vueuse/vueuse` |
| Vitest | `/vitest-dev/vitest` |
| Playwright | `/microsoft/playwright` |

## MCP 工具

此 skill 使用 Context7 MCP Server 提供的兩個工具：

### 1. resolve-library-id

解析程式庫名稱為 Context7 ID。

**使用方式**：
```
mcp__context7__resolve-library-id
- libraryName: "vuetify"
```

**回傳範例**：
```
/vuetifyjs/vuetify
```

### 2. get-library-docs

根據 Context7 ID 獲取相關文檔。

**使用方式**：
```
mcp__context7__get-library-docs
- context7CompatibleLibraryID: "/vuetifyjs/vuetify"
- topic: "v-data-table" (選填)
- tokens: 5000 (選填，預設 5000)
```

**回傳範例**：
```markdown
# v-data-table

The `v-data-table` component is used for displaying tabular data...

## Props
- items: Array of data items
- headers: Array of column definitions
...
```

## 執行流程

1. **確認需要查詢的程式庫** - 根據開發任務確認需要哪個程式庫的文檔
2. **解析程式庫 ID** - 使用 `resolve-library-id` 獲取 Context7 ID
3. **獲取文檔** - 使用 `get-library-docs` 獲取相關文檔
4. **應用到開發** - 根據最新文檔編寫程式碼

## 使用範例

### 範例 1：查詢 Vuetify 元件

```
用戶：我要使用 v-data-table-server

AI 執行流程：
1. 調用 mcp__context7__resolve-library-id，libraryName: "vuetify"
2. 調用 mcp__context7__get-library-docs，context7CompatibleLibraryID: "/vuetifyjs/vuetify"，topic: "v-data-table-server"
3. 根據返回的文檔說明正確用法
```

### 範例 2：查詢 Vue Composition API

```
用戶：defineModel 怎麼用？

AI 執行流程：
1. 調用 mcp__context7__resolve-library-id，libraryName: "vue"
2. 調用 mcp__context7__get-library-docs，context7CompatibleLibraryID: "/vuejs/core"，topic: "defineModel"
3. 根據返回的文檔說明正確用法
```

### 範例 3：查詢 Nuxt 功能

```
用戶：useFetch 和 $fetch 有什麼差別？

AI 執行流程：
1. 調用 mcp__context7__resolve-library-id，libraryName: "nuxt"
2. 調用 mcp__context7__get-library-docs，context7CompatibleLibraryID: "/nuxt/nuxt"，topic: "useFetch"
3. 根據返回的文檔說明差異
```

### 範例 4：UI 開發時查詢 Vuetify 元件

```
場景：開發表單頁面，需要使用 v-form 和驗證

AI 執行流程：
1. 調用 mcp__context7__get-library-docs，context7CompatibleLibraryID: "/vuetifyjs/vuetify"，topic: "v-form validation"
2. 根據文檔使用正確的 validation rules 和 submit 處理
3. 確保符合 Material Design 3 的表單設計原則
```

### 範例 5：確認 M3 元件用法

```
場景：需要使用 Material Design 3 的 FAB (Floating Action Button)

AI 執行流程：
1. 調用 mcp__context7__get-library-docs，context7CompatibleLibraryID: "/vuetifyjs/vuetify"，topic: "v-btn fab"
2. 確認 Vuetify 3 的 FAB 實現方式 (icon + position)
3. 應用正確的 M3 樣式
```

## UI 開發常用查詢

在進行 UI 開發時，以下是常見的 Context7 查詢：

| 需求 | topic 參數 |
|------|-----------|
| 表格元件 | `v-data-table`, `v-data-table-server` |
| 表單驗證 | `v-form validation`, `validation rules` |
| 對話框 | `v-dialog`, `v-bottom-sheet` |
| 導航 | `v-navigation-drawer`, `v-app-bar`, `v-tabs` |
| 選擇器 | `v-select`, `v-autocomplete`, `v-combobox` |
| 日期選擇 | `v-date-picker` |
| 通知 | `v-snackbar`, `v-alert` |
| 佈局 | `v-row`, `v-col`, `grid system` |
| 樣式 | `spacing`, `typography`, `colors` |
| 主題 | `theme`, `dark mode` |

## 自動觸發條件

AI 應在以下情況自動調用此 skill：

1. **不確定 API 用法時** - 當對某個 API 的參數或用法不確定
2. **開發新功能時** - 需要使用較少用的元件或 API
3. **API 可能有更新時** - 使用可能已更新的 API
4. **用戶詢問 API 時** - 用戶直接詢問某個 API 的用法
5. **UI 開發時** - 使用 Vuetify 元件前，確認最新的 props、slots、events
6. **確認 M3 規範時** - 需要確認某個 UI 模式是否符合 Material Design 3
7. **樣式設定時** - 需要查詢 Vuetify utility class 的正確用法

## 注意事項

- Context7 需要網路連線才能運作
- 如果 MCP Server 未連線，應告知用戶並提供已知的文檔連結
- 優先使用 topic 參數縮小查詢範圍，提高效率
- tokens 參數可根據需要調整，預設 5000 適合大多數情況
