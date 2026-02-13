---
name: e2e-test
description: 撰寫 Playwright 端對端測試。使用此技能當被要求寫 E2E 測試、端對端測試、playwright、整合測試時。
---

# E2E Test Skill

此技能用於撰寫 Playwright 端對端測試，遵守 E2E_TESTING_GUIDELINES.md 規範。

## 測試檔案位置

```
packages/SyncoBox.Testing.E2E.Playwright/tests/
├── shared/
│   ├── accounts.ts        # 帳號管理
│   └── utils.ts          # 通用工具
├── {功能模組}/
│   └── {編號}-{功能名稱}.spec.ts
```

## 執行流程

1. 分析功能流程
2. 建立 `.spec.ts` 檔案
3. 使用 `test.step` 結構化步驟
4. 執行測試驗證

## 指令

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

## 測試模板

```typescript
import { test, expect } from '@playwright/test';
import { accounts, loginUser } from '../shared/accounts';

test.describe('功能模組名稱', () => {
  // 環境條件跳過
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

### ✅ 正確寫法
```typescript
// 等待頁面載入
await page.waitForLoadState('networkidle');

// 等待元素可見
await element.waitFor({ state: 'visible' });

// 等待 API 回應
await page.waitForResponse(resp => resp.url().includes('/api/'));
```

### ❌ 禁止寫法
```typescript
// 禁止使用固定延遲
await page.waitForTimeout(3000);
```

## 禁止事項

- ❌ `page.waitForTimeout()` - 禁止固定延遲
- ❌ 過長的 CSS 選擇器鏈
- ❌ 依賴動態生成的 class 名稱

## test.step 命名規範

- 格式：`[動作描述]`
- 範例：`"進入公司設定>群組"`、`"新增測試群組"`、`"驗證群組已建立"`
- 使用清晰具體的動作描述

## 完成檢查清單

- [ ] 每個測試都有 `test.step`
- [ ] 使用推薦的選擇器策略
- [ ] 無 `waitForTimeout` 固定延遲
- [ ] 包含適當的斷言
- [ ] 測試資料有清理機制

