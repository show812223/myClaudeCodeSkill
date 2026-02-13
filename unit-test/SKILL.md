---
name: unit-test
description: 撰寫 Vitest 單元測試。使用此技能當被要求寫單元測試、測試元件、vitest 時。
---

# Unit Test Skill

此技能用於在 Monorepo 專案中撰寫 Vitest 單元測試。

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

### ⚠️ 重要規則

1. **禁止使用 `.skip` 跳過測試**
   - 不可使用 `it.skip()`、`describe.skip()` 或 `test.skip()` 跳過任何測試
   - 如果測試暫時無法通過，必須修復而非跳過

2. **覆蓋率未達 100% 時必須說明原因**
   - 執行 `pnpm test:coverage` 後，若覆蓋率未達 100%
   - 必須在回覆中清楚列出：
     - 未覆蓋的程式碼行數 (Uncovered Lines)
     - 未覆蓋的原因分析
     - 補足覆蓋率的建議測試案例

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
   
   2. **第 72 行**：條件分支 `if (data === null)` 未被測試
      - 建議：新增測試案例傳入 null 值
   
   ### 需補充的測試案例
   - [ ] 測試 API 錯誤時的 catch 處理
   - [ ] 測試 data 為 null 的情況
   ```

## 執行流程

1. 分析目標元件/函數
2. 建立測試檔案於 `tests/` 目錄
3. 撰寫測試案例
4. 執行 `pnpm test`
5. 確認測試通過

## 指令

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

## 測試模板

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { ref, computed } from 'vue';

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
      // Arrange
      const initialState = {};
      
      // Act
      const result = functionUnderTest(initialState);
      
      // Assert
      expect(result).toBeDefined();
    });
  });

  describe('功能測試', () => {
    it('應該正確處理使用者操作', async () => {
      // Arrange
      const mockData = { id: '1', name: 'Test' };
      
      // Act
      await performAction(mockData);
      
      // Assert
      expect(mockFn).toHaveBeenCalledWith(mockData);
    });
  });

  describe('邊界情況', () => {
    it('應該處理空資料', () => {
      expect(() => functionUnderTest(null)).not.toThrow();
    });

    it('應該處理無效輸入', () => {
      const result = functionUnderTest(undefined);
      expect(result).toEqual([]);
    });
  });
});
```

## 測試覆蓋範圍

- 元件初始化狀態
- Props 響應式變化
- 事件發射 (emit)
- 計算屬性 (computed)
- 邊界情況處理
- 錯誤處理

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

## 完成檢查清單

- [ ] 測試覆蓋所有 public 方法
- [ ] 測試覆蓋所有分支條件 (if/else/switch)
- [ ] 測試覆蓋所有錯誤處理路徑
- [ ] 測試邊界情況 (null/undefined/empty)
- [ ] 所有測試通過
- [ ] **Service Class 覆蓋率達 100%**
- [ ] 執行 `pnpm test:coverage` 確認覆蓋率報告
- [ ] **無使用 `.skip` 跳過任何測試**
- [ ] **若未達 100%，已提供未覆蓋原因分析**

## Service Class 測試模板

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import MyService from '../app/services/MyService';

// Mock stores
const mockUserStore = {
  userId: 'user123' as string | null,
  userInfo: { id: 'user123' },
  allFunctionNames: [] as string[],
};

vi.mock('~stores/stores/user', () => ({
  useUserStore: () => mockUserStore,
}));

vi.mock('~api/stores/auth', () => ({
  useAuthStore: () => ({
    token: 'mock-token',
  }),
}));

describe('MyService', () => {
  let service: MyService;

  beforeEach(() => {
    // Reset mocks
    vi.clearAllMocks();
    mockUserStore.userId = 'user123';
    
    // Create a new service instance
    service = new MyService();
  });

  describe('publicMethod', () => {
    it('應正確處理正常情況', () => {
      // Arrange
      const input = { id: '1' };
      
      // Act
      const result = service.publicMethod(input);
      
      // Assert
      expect(result).toBeDefined();
    });

    it('應正確處理邊界情況', () => {
      expect(() => service.publicMethod(null)).not.toThrow();
    });
  });
});
```

## 100% 覆蓋率測試模板

以下模板確保完整覆蓋所有程式碼路徑：

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import MyService from '../app/services/MyService';

describe('MyService - 100% Coverage', () => {
  let service: MyService;

  beforeEach(() => {
    vi.clearAllMocks();
    service = new MyService();
  });

  // ============================================
  // 1. Constructor 測試
  // ============================================
  describe('constructor', () => {
    it('應正確初始化實例', () => {
      expect(service).toBeInstanceOf(MyService);
    });

    it('應正確設定初始屬性', () => {
      // 驗證所有初始屬性
      expect(service.someProperty).toBeDefined();
    });
  });

  // ============================================
  // 2. Public Methods - 每個方法完整測試
  // ============================================
  describe('methodName', () => {
    // 正常流程
    it('應正確處理有效輸入', () => {
      const result = service.methodName({ id: '1' });
      expect(result).toEqual(expectedValue);
    });

    // 分支覆蓋 - if 條件為 true
    it('應處理條件為 true 的情況', () => {
      const result = service.methodName({ condition: true });
      expect(result).toEqual(trueBranchResult);
    });

    // 分支覆蓋 - if 條件為 false
    it('應處理條件為 false 的情況', () => {
      const result = service.methodName({ condition: false });
      expect(result).toEqual(falseBranchResult);
    });

    // 邊界情況 - null
    it('應處理 null 輸入', () => {
      const result = service.methodName(null);
      expect(result).toBeNull();
    });

    // 邊界情況 - undefined
    it('應處理 undefined 輸入', () => {
      const result = service.methodName(undefined);
      expect(result).toBeUndefined();
    });

    // 邊界情況 - 空陣列
    it('應處理空陣列', () => {
      const result = service.methodName([]);
      expect(result).toEqual([]);
    });

    // 邊界情況 - 空物件
    it('應處理空物件', () => {
      const result = service.methodName({});
      expect(result).toEqual({});
    });
  });

  // ============================================
  // 3. 錯誤處理測試
  // ============================================
  describe('錯誤處理', () => {
    it('應在無效輸入時拋出錯誤', () => {
      expect(() => service.methodThatThrows('invalid'))
        .toThrow('Expected error message');
    });

    it('應正確處理 async 錯誤', async () => {
      await expect(service.asyncMethodThatThrows())
        .rejects.toThrow('Async error');
    });

    it('應在 catch 區塊中正確處理錯誤', async () => {
      // Mock 依賴拋出錯誤
      vi.mocked(dependency.method).mockRejectedValueOnce(new Error('API Error'));
      
      const result = await service.methodWithTryCatch();
      expect(result).toEqual({ error: 'API Error' });
    });
  });

  // ============================================
  // 4. Switch/Case 分支測試
  // ============================================
  describe('switch 語句覆蓋', () => {
    it('應處理 case "A"', () => {
      expect(service.switchMethod('A')).toEqual(resultA);
    });

    it('應處理 case "B"', () => {
      expect(service.switchMethod('B')).toEqual(resultB);
    });

    it('應處理 default case', () => {
      expect(service.switchMethod('unknown')).toEqual(defaultResult);
    });
  });

  // ============================================
  // 5. 三元運算子覆蓋
  // ============================================
  describe('三元運算子覆蓋', () => {
    it('應處理條件為 truthy', () => {
      expect(service.ternaryMethod(true)).toEqual(truthyResult);
    });

    it('應處理條件為 falsy', () => {
      expect(service.ternaryMethod(false)).toEqual(falsyResult);
    });
  });

  // ============================================
  // 6. Optional Chaining (?.) 覆蓋
  // ============================================
  describe('optional chaining 覆蓋', () => {
    it('應處理物件存在的情況', () => {
      const result = service.optionalMethod({ nested: { value: 'test' } });
      expect(result).toEqual('test');
    });

    it('應處理物件為 undefined 的情況', () => {
      const result = service.optionalMethod(undefined);
      expect(result).toBeUndefined();
    });
  });

  // ============================================
  // 7. Nullish Coalescing (??) 覆蓋
  // ============================================
  describe('nullish coalescing 覆蓋', () => {
    it('應使用原值當值存在時', () => {
      expect(service.nullishMethod('value')).toEqual('value');
    });

    it('應使用預設值當值為 null 時', () => {
      expect(service.nullishMethod(null)).toEqual('default');
    });

    it('應使用預設值當值為 undefined 時', () => {
      expect(service.nullishMethod(undefined)).toEqual('default');
    });
  });

  // ============================================
  // 8. Callback/Event Handler 測試
  // ============================================
  describe('callback 處理', () => {
    it('應正確執行 callback', () => {
      const callback = vi.fn();
      service.methodWithCallback(callback);
      expect(callback).toHaveBeenCalledWith(expectedArgs);
    });

    it('應處理 callback 未提供的情況', () => {
      expect(() => service.methodWithCallback(undefined)).not.toThrow();
    });
  });

  // ============================================
  // 9. Array Methods 覆蓋 (map/filter/find/etc.)
  // ============================================
  describe('陣列方法覆蓋', () => {
    it('應正確 filter 陣列', () => {
      const result = service.filterMethod([1, 2, 3, 4, 5]);
      expect(result).toEqual([2, 4]); // 假設 filter 偶數
    });

    it('應處理 find 找到元素', () => {
      const result = service.findMethod([{ id: 1 }, { id: 2 }], 2);
      expect(result).toEqual({ id: 2 });
    });

    it('應處理 find 未找到元素', () => {
      const result = service.findMethod([{ id: 1 }], 999);
      expect(result).toBeUndefined();
    });
  });

  // ============================================
  // 10. Private 方法間接測試
  // ============================================
  describe('private 方法 (透過 public 方法測試)', () => {
    it('應透過 public 方法觸發 private 邏輯 - 路徑 A', () => {
      // 設定會觸發 private 方法路徑 A 的條件
      const result = service.publicMethod({ triggerPathA: true });
      expect(result).toEqual(pathAResult);
    });

    it('應透過 public 方法觸發 private 邏輯 - 路徑 B', () => {
      // 設定會觸發 private 方法路徑 B 的條件
      const result = service.publicMethod({ triggerPathA: false });
      expect(result).toEqual(pathBResult);
    });
  });
});
```

## 覆蓋率檢查指令

```bash
# 執行測試並檢查覆蓋率
pnpm test:coverage

# 檢查特定 service 的覆蓋率
pnpm test:coverage -- --coverage.include="packages/**/app/services/MyService.ts"

# 在 CI 中強制 100% 覆蓋率
pnpm test:coverage -- --coverage.thresholds.100
```

