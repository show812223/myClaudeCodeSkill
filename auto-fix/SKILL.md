---
name: auto-fix
description: 自動修正 code review 問題並補齊測試。使用此技能當被要求自動修正、補齊測試、通過 code review、fix review 時。
---

# Auto Fix Skill

此技能用於自動修正 code review 發現的問題，並補齊缺少的單元測試與 E2E 測試。

## 執行流程

1. **分析問題** - 讀取目標檔案，識別所有問題
2. **修正 TypeScript 錯誤** - 補齊型別定義、移除 `any`
3. **執行 Lint Fix** - 自動修正程式碼風格
4. **移除 Console.log** - 移除除錯用的 console
5. **產生單元測試** - 依據 unit-test skill 產生測試
6. **產生 E2E 測試** - 依據 e2e-test skill 補齊測試
7. **驗證結果** - 執行測試確認通過

## 自動修正項目

### 1. TypeScript 型別修正

#### 修正隱含 `any` 類型

```typescript
// ❌ 修正前
const items = _categories.filter((x) => !x.hasChildren);
let folders = [];

// ✅ 修正後
const items = _categories.filter((x: IFolder) => !x.hasChildren);
const folders: IFolder[] = [];
```

#### 補齊缺少的 interface 屬性

```typescript
// ❌ 修正前 - reactive 物件缺少屬性
const modelViewer = reactive({
  dialog: false,
  modelName: "",
});

// ✅ 修正後 - 定義完整 interface
interface IModelViewer {
  dialog: boolean;
  modelName: string;
  models: BIMModel[];
  defaultOpenMarkupId: string;
}

const modelViewer = reactive<IModelViewer>({
  dialog: false,
  modelName: "",
  models: [],
  defaultOpenMarkupId: "",
});
```

#### 修正可能為 undefined 的屬性

```typescript
// ❌ 修正前
.sort((a, b) => a.sort - b.sort)

// ✅ 修正後
.sort((a, b) => (a.sort ?? 0) - (b.sort ?? 0))
```

### 2. 移除 Console.log

```typescript
// 自動移除以下模式
console.log("...", ...)  // 移除
console.warn("...")      // 移除
console.debug("...")     // 移除

// 保留錯誤處理
console.error(error)     // 保留
```

### 3. 移除未使用程式碼

- 移除未使用的變數
- 移除未使用的 import
- 移除未呼叫的函數
- 移除假資料 (fake data) 函數

### 4. Import 與 Components 檢查

#### 檢查 import 路徑是否正確

```typescript
// ❌ 錯誤的路徑
import { useCompanyStore } from '~stores/store/company';  // typo: store -> stores
import type { BIMModel } from '~bim/domain/bim';          // 路徑不存在

// ✅ 正確的路徑
import { useCompanyStore } from '~stores/stores/company';
import type { BIMModel } from '~bim/domain-classes/bim';
```

#### 檢查 components 是否正確註冊/導入

```vue
<template>
  <!-- ❌ 使用未導入的元件 -->
  <CustomDialog />
  <UnknownComponent />
  
  <!-- ✅ 確保元件已導入或為全域註冊 -->
  <BIMModelViewer />  <!-- 需確認是否為 auto-import 或手動 import -->
</template>

<script setup lang="ts">
// 檢查是否需要手動 import
import CustomDialog from '~/components/CustomDialog.vue';
</script>
```

#### 檢查項目

| 項目 | 檢查內容 |
|------|----------|
| 路徑存在性 | import 的檔案路徑是否存在 |
| 路徑別名 | 是否正確使用 `~ui`, `~stores`, `~bim` 等別名 |
| 元件註冊 | template 中使用的元件是否已導入或全域註冊 |
| 型別導入 | `type` 導入是否使用 `import type` 語法 |
| 循環依賴 | 是否存在循環 import |

#### 常見路徑別名對照

```typescript
// Nuxt Layer 別名
'~api'        → 'packages/syncobox-api/app'
'~ui'         → 'packages/syncobox-ui/app'
'~stores'     → 'packages/syncobox-stores/app'
'~bim'        → 'packages/syncobox-bim/app'
'~bimContent' → 'packages/syncobox-bimContent/app'
'~portal'     → 'packages/syncobox-sharedPortal/app'
'~forge'      → 'packages/syncobox-forge/app'
'~task'       → 'packages/syncobox-task/app'

// 當前 app 別名
'~/' or '@/'  → 當前 app 的根目錄
```

#### 自動修正動作

1. **修正錯誤路徑** - 搜尋正確的檔案位置並更新 import
2. **補齊缺少的 import** - 為 template 中使用的元件加入 import
3. **移除重複 import** - 合併相同來源的多個 import
4. **排序 import** - 依照規範排序 (types → external → internal)

## 自動產生測試

### 單元測試模板

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { mount } from '@vue/test-utils';
import { createVuetify } from 'vuetify';
import { createI18n } from 'vue-i18n';
import ComponentName from '@/pages/path/ComponentName.vue';

// Mock 外部依賴
vi.mock('~stores/stores/company', () => ({
  useCompanyStore: vi.fn(() => ({
    companyId: 'test-company-id',
    companyName: 'Test Company',
  })),
}));

vi.mock('~bim/composables/useBIMBIMCategories', () => ({
  useBIMBIMCategories: vi.fn(() => ({
    getCompanyBIMCategoriesAll: vi.fn().mockResolvedValue([]),
    getFolderItems: vi.fn().mockResolvedValue({ data: [] }),
    postReviewLogs: vi.fn().mockResolvedValue({}),
  })),
}));

const vuetify = createVuetify();
const i18n = createI18n({
  legacy: false,
  locale: 'zh-TW',
  messages: {},
});

describe('ComponentName', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('初始化', () => {
    it('應該正確渲染頁面', () => {
      const wrapper = mount(ComponentName, {
        global: {
          plugins: [vuetify, i18n],
          stubs: {
            // 依需求 stub 子元件
          },
        },
      });
      expect(wrapper.exists()).toBe(true);
    });

    it('應該顯示載入狀態', async () => {
      const wrapper = mount(ComponentName, {
        global: { plugins: [vuetify, i18n] },
      });
      // 驗證載入狀態
      expect(wrapper.find('.v-progress-circular').exists()).toBe(true);
    });
  });

  describe('功能測試', () => {
    it('應該正確處理資料夾點擊', async () => {
      // 測試邏輯
    });

    it('應該正確處理搜尋', async () => {
      // 測試邏輯
    });
  });

  describe('邊界情況', () => {
    it('應該處理空資料', async () => {
      // 測試邏輯
    });
  });
});
```

### E2E 測試模板

```typescript
import { test, expect } from '@playwright/test';
import { loginUser, accounts, useStorageState } from '../../shared/accounts';

test.describe('功能模組名稱', () => {
  useStorageState('companyManager');

  test('頁面載入與初始化', async ({ page }) => {
    await test.step('導航至目標頁面', async () => {
      await page.goto('/target-page');
      await page.waitForLoadState('networkidle');
    });

    await test.step('驗證頁面元素', async () => {
      await expect(page.locator('.page-title')).toBeVisible();
    });
  });

  test('主要功能流程', async ({ page }) => {
    await test.step('前置準備', async () => {
      await page.goto('/target-page');
      await page.waitForLoadState('networkidle');
    });

    await test.step('執行操作', async () => {
      // 操作邏輯
    });

    await test.step('驗證結果', async () => {
      // 驗證邏輯
    });
  });

  test('搜尋功能', async ({ page }) => {
    await test.step('輸入搜尋條件', async () => {
      await page.fill('[placeholder*="搜尋"]', 'test');
    });

    await test.step('驗證搜尋結果', async () => {
      // 驗證邏輯
    });
  });

  test('錯誤處理', async ({ page }) => {
    await test.step('模擬錯誤情況', async () => {
      // 錯誤處理測試
    });
  });
});
```

## 執行指令

```bash
# 1. 自動修正 Lint 問題
pnpm --filter @syncobox/{package-name} lint --fix

# 2. 執行單元測試
pnpm test -- {test-file-path}

# 3. 執行 E2E 測試
pnpm e2e test:onpremise -- --grep "{test-name}"

# 4. 檢查型別
pnpm --filter @syncobox/{package-name} typecheck
```

## 驗證清單

修正完成後，確認以下項目：

- [ ] `pnpm lint` 無錯誤
- [ ] `pnpm typecheck` 無錯誤
- [ ] 無 `console.log` (除 console.error)
- [ ] 無未使用的變數/import
- [ ] 單元測試存在且通過
- [ ] E2E 測試存在且通過
- [ ] 覆蓋率達標 (行 ≥80%, 分支 ≥70%)

## 使用範例

```
幫我自動修正這個檔案並補齊測試
```

```
讓這個頁面通過 code review
```

```
幫我 fix review 發現的問題
```

```
自動補齊 modelViews.vue 的單元測試和 E2E 測試
```
