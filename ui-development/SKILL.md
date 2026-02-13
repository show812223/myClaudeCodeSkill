---
name: ui-development
description: 建立 Vue 3 + Vuetify 3 元件與頁面。使用此技能當被要求建立元件、新增頁面、開發介面時。
---

# UI Development Skill

此技能用於在 Nuxt 4 Monorepo 專案中建立 Vue 3 元件與頁面。

## 設計原則：Material Design 3

所有 UI 開發必須遵循 **Material Design 3 (M3)** 規範，並優先使用 **Vuetify 3** 實現：

### M3 核心原則

| 原則 | 說明 | Vuetify 實現 |
|------|------|--------------|
| **動態色彩** | 基於用戶偏好的色彩系統 | 使用 Vuetify theme 和 `color` prop |
| **無障礙設計** | 確保對比度和可讀性 | 使用語義化元件，設定 `aria-*` 屬性 |
| **響應式佈局** | 適應不同螢幕尺寸 | 使用 `v-row`/`v-col` 和 breakpoint class |
| **一致的間距** | 統一的留白系統 | 使用 Vuetify spacing class (`ma-*`, `pa-*`) |
| **明確的層級** | 清晰的視覺階層 | 使用 `elevation-*` 和 typography class |

### M3 元件對應

| M3 元件 | Vuetify 元件 | 使用場景 |
|---------|--------------|----------|
| Filled Button | `<v-btn variant="flat">` | 主要操作 |
| Outlined Button | `<v-btn variant="outlined">` | 次要操作 |
| Text Button | `<v-btn variant="text">` | 低強調操作 |
| FAB | `<v-btn icon>` + `position="fixed"` | 頁面主要動作 |
| Card | `<v-card>` | 內容容器 |
| Dialog | `<v-dialog>` | 模態互動 |
| Snackbar | `<v-snackbar>` | 簡短通知 |
| Navigation Drawer | `<v-navigation-drawer>` | 側邊導航 |
| App Bar | `<v-app-bar>` | 頂部導航 |
| Bottom Navigation | `<v-bottom-navigation>` | 行動版底部導航 |

## 檔案位置

| 類型 | 路徑 |
| ---- | ---- |
| 元件 | `packages/{package}/app/components/{ComponentName}.vue` |
| 頁面 | `packages/{package}/app/pages/{page-name}.vue` |
| Composable | `packages/{package}/app/composables/{useName}.ts` |
| **Service Class** | `packages/{package}/app/services/{PageName}Service.ts` |

## Package 對應

| 別名 | 路徑 |
| ---- | ---- |
| `~api` | `packages/syncobox-api/app` |
| `~ui` | `packages/syncobox-ui/app` |
| `~stores` | `packages/syncobox-stores/app` |
| `~bim` | `packages/syncobox-bim/app` |
| `~bimContent` | `packages/syncobox-bimContent/app` |
| `~portal` | `packages/syncobox-sharedPortal/app` |
| `~task` | `packages/syncobox-task/app` |

## 執行流程

1. 分析需求，確認目標 package
2. **使用 `/ui-ux-pro-max` skill 設計 UI/UX** (新頁面或重大 UI 變更時必須執行)
3. 檢查是否有可重用的現有元件
4. **使用 `/context7` skill 查詢最新 API 文檔** (使用不熟悉的 API 時執行)
5. **建立 Service Class** (頁面邏輯必須放在 Service Class) → 參考 [templates/service-class.md](templates/service-class.md)
6. 建立 `.vue` 檔案 (僅處理 UI 渲染與事件綁定) → 參考 [templates/vue-component.md](templates/vue-component.md)
7. 執行 lint 驗證
8. **使用 `/unit-test` skill 自動產生單元測試** (必須執行)
9. **使用 Playwright MCP 測試頁面** (視覺化驗證) → 參考 [guides/playwright-mcp.md](guides/playwright-mcp.md)

## 整合的 Skills

### /context7 - 程式庫文檔查詢

在使用 Vue、Vuetify、Nuxt、Pinia 等程式庫的 API 時，使用 Context7 獲取最新文檔：

**自動觸發條件**：
- 不確定 API 用法或參數時
- 使用較少用的元件或 API 時
- 用戶詢問某個 API 的正確用法時

**常用查詢**：

| 程式庫 | Context7 ID | 常見查詢 topic |
|--------|-------------|----------------|
| Vuetify 3 | `/vuetifyjs/vuetify` | v-data-table, v-dialog, v-form |
| Vue 3 | `/vuejs/core` | defineModel, defineProps, watch |
| Nuxt 3 | `/nuxt/nuxt` | useFetch, useAsyncData, middleware |
| Pinia | `/vuejs/pinia` | defineStore, storeToRefs |

**使用方式**：
1. 調用 `mcp__context7__resolve-library-id` 解析程式庫 ID
2. 調用 `mcp__context7__get-library-docs` 獲取文檔

### /ui-ux-pro-max - UI/UX 設計

在開發新頁面或進行重大 UI 變更時必須使用：

```bash
# 生成設計系統
python3 skills/ui-ux-pro-max/scripts/search.py "<產品類型> <關鍵字>" --design-system -p "專案名稱"

# 取得 Vue 相關的實作指南
python3 skills/ui-ux-pro-max/scripts/search.py "<關鍵字>" --stack vue
```

### /unit-test - 單元測試

元件開發完成後必須產生測試：

| 項目 | 最低覆蓋率 |
| ---- | ---------- |
| 元件 (Components) | 80% |
| Composables | 90% |
| Service Classes | 100% |

### Playwright MCP - 視覺化測試

詳細流程參考 [guides/playwright-mcp.md](guides/playwright-mcp.md)

## 核心規範

### Service Class 架構

開發新頁面時，**必須建立對應的 Service Class**：

| Vue 檔案職責 | Service Class 職責 |
| ------------ | ------------------ |
| UI 渲染 | 業務邏輯處理 |
| 事件綁定 | API 呼叫 |
| 響應式狀態展示 | 資料轉換與驗證 |

完整模板參考 [templates/service-class.md](templates/service-class.md)

### Vue 元件規則

1. 使用 `<script setup lang="ts">`
2. Props/Emits 使用 TypeScript 泛型定義
3. 禁止原生 HTML，使用 Vuetify 元件
4. 使用 Path Alias，不用相對路徑
5. 禁止使用 `any` 型別

完整模板參考 [templates/vue-component.md](templates/vue-component.md)

### Vuetify 元件對應規則

**禁止使用原生 HTML 元素，必須使用對應的 Vuetify 元件：**

| 禁止 | 使用 | 說明 |
| ---- | ---- | ---- |
| `<button>` | `<v-btn>` | 支援 variant, color, size |
| `<input>` | `<v-text-field>` | 支援 validation, variant |
| `<select>` | `<v-select>` / `<v-autocomplete>` | 支援搜尋、多選 |
| `<textarea>` | `<v-textarea>` | 支援 auto-grow |
| `<checkbox>` | `<v-checkbox>` | 支援 indeterminate |
| `<radio>` | `<v-radio-group>` + `<v-radio>` | 群組管理 |
| `<div>` (容器) | `<v-sheet>` / `<v-card>` | 支援 elevation, rounded |
| `<table>` | `<v-data-table>` | 支援排序、分頁、篩選 |
| `<dialog>` | `<v-dialog>` | 支援 persistent, fullscreen |
| `<img>` | `<v-img>` | 支援 lazy loading, aspect-ratio |
| `<a>` (導航) | `<v-btn :to="...">` / `<NuxtLink>` | 支援 router 整合 |
| `<ul>/<li>` | `<v-list>` + `<v-list-item>` | 支援 interactive items |
| `<nav>` | `<v-navigation-drawer>` / `<v-app-bar>` | M3 導航元件 |
| `<progress>` | `<v-progress-linear>` / `<v-progress-circular>` | 支援 indeterminate |
| `<tooltip>` | `<v-tooltip>` | 支援 location, delay |
| `<menu>` | `<v-menu>` | 支援 activator slot |

### Vuetify Class 優先規則

**禁止使用 scoped style 和 inline style，必須使用 Vuetify utility class：**

#### 間距 (Spacing)

| CSS | Vuetify Class | 範例 |
|-----|---------------|------|
| `margin` | `ma-*`, `mt-*`, `mb-*`, `ml-*`, `mr-*`, `mx-*`, `my-*` | `class="ma-4 mb-2"` |
| `padding` | `pa-*`, `pt-*`, `pb-*`, `pl-*`, `pr-*`, `px-*`, `py-*` | `class="pa-4 px-6"` |
| `gap` | `ga-*` | `class="ga-4"` |

**間距值**：0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 16 (1 = 4px)

#### 尺寸 (Sizing)

| CSS | Vuetify Class |
|-----|---------------|
| `width: 100%` | `w-100` |
| `height: 100%` | `h-100` |
| `width: auto` | `w-auto` |
| `height: 100vh` | `h-screen` |

#### Flexbox

| CSS | Vuetify Class |
|-----|---------------|
| `display: flex` | `d-flex` |
| `flex-direction: column` | `flex-column` |
| `justify-content: center` | `justify-center` |
| `justify-content: space-between` | `justify-space-between` |
| `align-items: center` | `align-center` |
| `flex-wrap: wrap` | `flex-wrap` |
| `flex-grow: 1` | `flex-grow-1` |
| `gap` | `ga-*` |

#### 文字 (Typography)

| 用途 | Vuetify Class |
|------|---------------|
| 標題 | `text-h1` ~ `text-h6` |
| 副標題 | `text-subtitle-1`, `text-subtitle-2` |
| 內文 | `text-body-1`, `text-body-2` |
| 說明文字 | `text-caption` |
| 對齊 | `text-left`, `text-center`, `text-right` |
| 粗細 | `font-weight-bold`, `font-weight-medium`, `font-weight-light` |
| 轉換 | `text-uppercase`, `text-lowercase`, `text-capitalize` |
| 截斷 | `text-truncate` |

#### 顏色 (Colors)

| 用途 | Vuetify Class |
|------|---------------|
| 背景色 | `bg-primary`, `bg-secondary`, `bg-surface`, `bg-error` |
| 文字色 | `text-primary`, `text-secondary`, `text-disabled` |
| 語義色 | `text-success`, `text-warning`, `text-error`, `text-info` |

#### 顯示 (Display)

| CSS | Vuetify Class |
|-----|---------------|
| `display: none` | `d-none` |
| `display: block` | `d-block` |
| `display: inline` | `d-inline` |
| 響應式隱藏 | `d-none d-md-flex` (md 以上顯示) |

#### 圓角與陰影

| 用途 | Vuetify Class / Prop |
|------|---------------------|
| 圓角 | `rounded`, `rounded-lg`, `rounded-xl`, `rounded-pill` |
| 陰影 | `elevation-0` ~ `elevation-24` |

#### 響應式 Breakpoints

| Breakpoint | 寬度 | Class 前綴 |
|------------|------|-----------|
| xs | < 600px | (無前綴) |
| sm | 600-959px | `sm-` |
| md | 960-1279px | `md-` |
| lg | 1280-1919px | `lg-` |
| xl | 1920-2559px | `xl-` |
| xxl | ≥ 2560px | `xxl-` |

**範例**：`class="d-none d-md-flex"` = 手機隱藏，平板以上顯示

## 相關檔案

- [templates/service-class.md](templates/service-class.md) - Service Class 完整模板
- [templates/vue-component.md](templates/vue-component.md) - Vue 元件完整模板
- [guides/playwright-mcp.md](guides/playwright-mcp.md) - Playwright MCP 測試指南
