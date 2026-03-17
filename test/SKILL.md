---
name: test
description: 撰寫測試（單元測試 + E2E 測試）。使用此技能當被要求寫測試、單元測試、E2E 測試、vitest、playwright 測試、覆蓋率時。
---

# Test Skill

整合單元測試（Vitest）和 E2E 測試（Playwright）的統一測試技能。

## 使用方式

```
/test [類型] [目標]
```

| 指令 | 說明 |
|------|------|
| `/test unit [目標]` | 只寫 Vitest 單元測試 |
| `/test e2e [目標]` | 只寫 Playwright E2E 測試 |
| `/test all [目標]` | 兩者都寫（預設） |
| `/test [目標]` | 等同 `/test all [目標]` |

範例：
```
/test unit packages/syncobox-task/app/services/TaskService.ts
/test e2e syncobox-bim 的模型檢視頁面
/test all syncobox-ui 的 ProjectList 頁面
```

## 判斷測試類型

解析 `$ARGUMENTS` 第一個詞：
- 若為 `unit` → 只執行單元測試流程
- 若為 `e2e` → 只執行 E2E 測試流程
- 若為 `all` 或未指定 → 兩者都執行（先 unit 再 e2e）

---

# Part 1：單元測試（Vitest）

## 測試檔案位置

```
packages/{package-name}/tests/
├── components/
│   └── {ComponentName}.test.ts
├── composables/
│   └── {composableName}.test.ts
├── services/
│   └── {ServiceName}.test.ts      # Service Class 測試 (重點覆蓋)
└── utils/
    └── {utilName}.test.ts
```

## 測試覆蓋率策略

**專注於 Service Class 測試 - 100% 覆蓋率目標**

覆蓋率配置已設定為專注於 `services/` 目錄中的 class 檔案：
- `packages/**/app/services/**/*.ts`
- `apps/**/app/services/**/*.ts`

### 覆蓋率要求

| 指標 | 目標 |
|------|------|
| Statements | 100% |
| Branches | 100% |
| Functions | 100% |
| Lines | 100% |

### 達成 100% 覆蓋率的關鍵策略

1. **所有 public 方法必須測試**：每個 public method 至少有一個測試案例
2. **所有 private 方法透過 public 方法間接測試**：確保 private 方法的邏輯被完整執行
3. **所有分支條件必須覆蓋**：if/else、switch、三元運算子的每個分支都要測試
4. **所有錯誤處理路徑必須測試**：try/catch、throw、error callback
5. **所有邊界情況必須測試**：null、undefined、空陣列、空物件

### 重要規則

1. **禁止使用 `.skip` 跳過測試**
   - 不可使用 `it.skip()`、`describe.skip()` 或 `test.skip()`
   - 如果測試暫時無法通過，必須修復而非跳過

2. **覆蓋率未達 100% 時必須說明原因**
   - 執行 `pnpm test:coverage` 後，若覆蓋率未達 100%
   - 必須列出：未覆蓋的程式碼行數、未覆蓋原因分析、補足覆蓋率的建議測試案例

3. **覆蓋率報告分析格式**
   ```
   ## 覆蓋率分析報告

   ### 未達 100% 的項目
   | 檔案 | Statements | Branches | Functions | Lines | 未覆蓋行數 |
   |------|------------|----------|-----------|-------|-----------|
   | MyService.ts | 95% | 90% | 100% | 95% | 45-48, 72 |

   ### 未覆蓋原因分析
   1. **第 45-48 行**：錯誤處理的 catch 區塊未被觸發
      - 建議：Mock API 拋出錯誤以觸發 catch 路徑

   ### 需補充的測試案例
   - [ ] 測試 API 錯誤時的 catch 處理
   - [ ] 測試 data 為 null 的情況
   ```

## 單元測試執行指令

```bash
# 執行所有測試
pnpm test

# 監聽模式
pnpm test:watch

# UI 模式
pnpm test:ui

# 執行測試並生成覆蓋率報告 (專注於 services)
pnpm test:coverage

# 執行特定檔案
pnpm test -- packages/syncobox-ui/test/components/TreeView.test.ts
```

## 單元測試模板

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { ref } from 'vue';

// Mock 外部依賴
vi.mock('~api/composables/useCustomFetch', () => ({
  useCustomFetch: vi.fn(() => ({
    data: ref(null),
    pending: ref(false),
    error: ref(null),
  })),
}));

describe('ComponentName', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('初始化', () => {
    it('應該正確初始化預設狀態', () => {
      // Arrange → Act → Assert
    });
  });

  describe('功能測試', () => {
    it('應該正確處理使用者操作', async () => {
      // Arrange → Act → Assert
    });
  });

  describe('邊界情況', () => {
    it('應該處理空資料', () => {
      expect(() => functionUnderTest(null)).not.toThrow();
    });
  });
});
```

## Service Class 測試模板

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import MyService from '../app/services/MyService';

const mockUserStore = {
  userId: 'user123' as string | null,
  userInfo: { id: 'user123' },
  allFunctionNames: [] as string[],
};

vi.mock('~stores/stores/user', () => ({
  useUserStore: () => mockUserStore,
}));

describe('MyService - 100% Coverage', () => {
  let service: MyService;

  beforeEach(() => {
    vi.clearAllMocks();
    mockUserStore.userId = 'user123';
    service = new MyService();
  });

  // 1. Constructor 測試
  // 2. Public Methods - 每個方法完整測試（正常 + 分支 + 邊界）
  // 3. 錯誤處理測試（try/catch、async 錯誤）
  // 4. Switch/Case 分支測試（每個 case + default）
  // 5. 三元運算子覆蓋（truthy + falsy）
  // 6. Optional Chaining (?.) 覆蓋（存在 + undefined）
  // 7. Nullish Coalescing (??) 覆蓋（有值 + null + undefined）
  // 8. Callback/Event Handler 測試
  // 9. Array Methods 覆蓋（filter/find/map 的 callback）
  // 10. Private 方法間接測試
});
```

## Mock 常用模式

### Mock Pinia Store
```typescript
vi.mock('pinia', () => ({
  defineStore: vi.fn((name, setup) => () => setup()),
  storeToRefs: vi.fn((store) => store),
}));
```

### Mock API 呼叫
```typescript
vi.mock('~api/composables/useCustomFetch', () => ({
  useCustomFetch: vi.fn(() => ({
    data: ref({ id: '1' }),
    pending: ref(false),
    error: ref(null),
  })),
}));
```

### Mock window 物件
```typescript
Object.defineProperty(global, 'window', {
  value: {
    innerWidth: 1920,
    innerHeight: 1080,
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
  },
  writable: true,
});
```

## 單元測試完成檢查清單

- [ ] 測試覆蓋所有 public 方法
- [ ] 測試覆蓋所有分支條件 (if/else/switch)
- [ ] 測試覆蓋所有錯誤處理路徑
- [ ] 測試邊界情況 (null/undefined/empty)
- [ ] 所有測試通過
- [ ] **Service Class 覆蓋率達 100%**
- [ ] 執行 `pnpm test:coverage` 確認覆蓋率報告
- [ ] **無使用 `.skip` 跳過任何測試**
- [ ] **若未達 100%，已提供未覆蓋原因分析**

## 覆蓋率檢查指令

```bash
pnpm test:coverage
pnpm test:coverage -- --coverage.include="packages/**/app/services/MyService.ts"
```

---

# Part 2：E2E 測試（Playwright）

## 測試檔案位置

```
packages/SyncoBox.Testing.E2E.Playwright/tests/
├── shared/
│   ├── accounts.ts        # 帳號管理
│   └── utils.ts          # 通用工具
├── {功能模組}/
│   └── {編號}-{功能名稱}.spec.ts
```

## E2E 測試執行指令

```bash
# 本地執行
pnpm e2e:local

# 指定環境執行
pnpm e2e test:onpremise

# UI 模式
pnpm e2e test:onpremise-ui

# 錄製測試
pnpm codegen
```

## E2E 測試模板

```typescript
import { test, expect } from '@playwright/test';
import { accounts, loginUser } from '../shared/accounts';

test.describe('功能模組名稱', () => {
  test.skip(process.env.IS_TSMC === 'true', 'TSMC 環境跳過此測試');

  test('測試案例描述', async ({ page }) => {
    await test.step('前置準備 - 登入系統', async () => {
      await page.goto('/');
      await loginUser(page, accounts.companyManager);
      await page.waitForLoadState('networkidle');
    });

    await test.step('導航至目標頁面', async () => {
      await page.getByRole('link', { name: '功能名稱' }).click();
      await page.waitForLoadState('networkidle');
      await expect(page).toHaveURL(/\/target-page/);
    });

    await test.step('執行主要操作', async () => {
      const submitBtn = page.getByRole('button', { name: '提交' });
      await submitBtn.waitFor({ state: 'visible' });
      await submitBtn.click();
      await expect(page.getByText('操作成功')).toBeVisible();
    });

    await test.step('驗證 API 回應', async () => {
      const [response] = await Promise.all([
        page.waitForResponse(resp =>
          resp.url().includes('/api/endpoint') && resp.status() === 200
        ),
        page.getByRole('button', { name: '確認' }).click(),
      ]);
      expect(response.status()).toBe(200);
    });

    await test.step('資料清理', async () => {
      const deleteBtn = page.locator('.mdi-delete');
      if (await deleteBtn.isVisible()) {
        await deleteBtn.click();
        await page.locator('#confirmBtn').click();
        await page.waitForLoadState('networkidle');
      }
    });
  });
});
```

## 選擇器優先順序

1. `page.getByRole()` - 最推薦
2. `page.getByLabel()` - 表單欄位
3. `page.getByText()` - 文字內容
4. `page.getByTestId()` - 測試 ID
5. `page.locator('#id')` - ID 選擇器
6. `page.locator('.class')` - 最後手段

## 等待機制

### 正確寫法
```typescript
await page.waitForLoadState('networkidle');
await element.waitFor({ state: 'visible' });
await page.waitForResponse(resp => resp.url().includes('/api/'));
```

### 禁止寫法
```typescript
// ❌ 禁止使用固定延遲
await page.waitForTimeout(3000);
```

## E2E 測試完成檢查清單

- [ ] 每個測試都有 `test.step`
- [ ] 使用推薦的選擇器策略
- [ ] 無 `waitForTimeout` 固定延遲
- [ ] 包含適當的斷言
- [ ] 測試資料有清理機制
